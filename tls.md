## tls

### mesos sanbox .ssl folder

Marathon services out of the box have a .ssl folder in the sanbox containing 3 files.

- ca-bundle.crt - the [dc/os CA bundle](https://docs.mesosphere.com/1.11/security/ent/tls-ssl/get-cert/)
- scheduler.crt - the service certificate
- scheduler.key - the service private key

The schedule crt at the moment is created with the ip as host. If you have one marathon service calling a target marathon service using like the vip name, and the target service uses scheduler.crt to identify itself, then validation will fail. A JIRA is open to add alternative subjets like dns name , and vip name to scheduler.crt.

In the next section we show how you can create your own certificate requests for your service.


### creating a certificate request for your service

*setup.sh* a scipt to create your specific service.crt and service.key file.
```sh
#!/bin/bash
  
# setup dcos cli
chmod +x dcos
export LC_ALL=C.UTF-8
export LANG=C.UTF-8

# authN with dcos using service account
cat service_account_secret | jq -r .private_key > service_account_key.pem
chmod 600 service_account_key.pem
./dcos cluster setup https://leader.mesos --ca-certs=.ssl/ca-bundle.crt --username=$SERVICE_ACCOUNT --private-key=service_account_key.pem

# install dcos enterprise cli
./dcos package install dcos-enterprise-cli --yes

# create service crt and key
ID=$(echo $MARATHON_APP_ID | sed 's/\///g')
./dcos security cluster ca newcert --cn $ID.marathon.l4lb.thisdcos.directory --name-c US --name-st CA --name-o "Mesosphere, Inc" --name-l "San Francisco" --key-algo rsa --key-size 2048 --host $ID.marathon.l4lb.thisdcos.directory -j > servicecert.json
cat servicecert.json | jq -r .certificate > service.crt
cat servicecert.json | jq -r .private_key > service.key

```

*marathon.json*, shows the download of the dc/os cli and the setup.sh script, and how you run setup.sh in cmd before any other execution.
```js
{
  "id": "service",
  "container": {
    "type": "MESOS",
    ...
    "volumes": [
      {
        "containerPath": "service_account_secret",
        "secret": "service-account-secret"
      }
    ]
  },
  "fetch": [
    {"uri": "https://downloads.dcos.io/binaries/cli/linux/x86-64/dcos-1.11/dcos", "executable": true},
    {"uri": "https://s3.amazonaws.com/.../setup.sh", "executable": true}
  ],
  "secrets": {
    "service-account-secret": {
      "source": "my-service-acct-secret"
    }
  },
  "env": {
    "SERVICE_ACCOUNT": "my-service-acct"
  },
  "cmd": "./setup.sh && ..."
}

```

### a tls service enpoint implementation

*service.js*, a service endpoint implementation using node js with tls enabled, you can see how key and cert get configured with the files created from the setup.sh script.
```js
var express = require('express');
var http = require('http');
var https = require('https');
var fs = require('fs');

var sslOptions = {
  key: fs.readFileSync('service.key'),
  cert: fs.readFileSync('service.crt')
};

var server = express();

server.get('/', function (req, res) {
    res.send("Hello World!");
});

https.createServer(sslOptions, server).listen(process.argv[2], "0.0.0.0")

```

*marathon.json*, you san see how in the cmd we call first setup before starting the actual service.
```js
{
  "id": "service",
  "container": {
    "type": "MESOS",
    "docker": {
      "image": "...",
      "forcePullImage": true
    },
    "volumes": [
      {
        "containerPath": "service_account_secret",
        "secret": "service-accountt-secret"
      }
    ]
  },
  "fetch": [
    {"uri": "https://downloads.dcos.io/binaries/cli/linux/x86-64/dcos-1.11/dcos", "executable": true},
    {"uri": "https://s3.amazonaws.com/.../setup.sh", "executable": true}
  ],
  "secrets": {
    "service-account-secret": {
      "source": "my-service-acct-secret"
    }
  },
  "networks": [
    {
      "mode": "host"
    }
  ],
  "portDefinitions": [ {
      "name": "web",
      "port": 8080,
      "protocol": "tcp",
      "labels": {
        "VIP_0": "service:8080"
      }
  } ],
  "env": {
    "SERVICE_ACCOUNT": "my-service-acct"
  },
  "cmd": "./setup.sh && nodejs service.js $PORT0"
}

```

For testing you can *dcos task exec ...* into another marathon service and use curl to call the former service.

```console
curl --cacert .ssl/ca-bundle.crt https://service.marathon.l4lb.thisdcos.directory:8080
```

### a tls service endpoint implementation requiring client authentication

*service.js*, this service implementation requires that that calling client autheticate via certificate. For that we set *requestCert* to true, and also configure the dc/os CA bundle.
```js
var express = require('express');
var http = require('http');
var https = require('https');
var fs = require('fs');

var sslOptions = {
  key: fs.readFileSync('service.key'),
  cert: fs.readFileSync('service.crt'),
  ca: fs.readFileSync('.ssl/ca-bundle.crt'),
  requestCert: true,
  rejectUnauthorized: false
};

var server = express();

server.use(function (req, res, next) {
   if (!req.client.authorized) {
       return res.status(401).send('User is not authorized');
   }
   var cert = req.socket.getPeerCertificate();
   if (cert.subject) {
       console.log(cert.subject.CN);
   }
   next();
});

server.get('/', function (req, res) {                                                                                        

    res.send("Hello World!");                                                                                               
    
});

https.createServer(sslOptions, server).listen(process.argv[2], "0.0.0.0") 

```

*marathon.json*, you san see how in the cmd we call first setup before starting the actual service.
```js
{
  "id": "service",
  "container": {
    "type": "MESOS",
    "docker": {
      "image": "...",
      "forcePullImage": true
    },
    "volumes": [
      {
        "containerPath": "service_account_secret",
        "secret": "service-account-secret"
      }
    ]
  },
  "fetch": [
    {"uri": "https://downloads.dcos.io/binaries/cli/linux/x86-64/dcos-1.11/dcos", "executable": true},
    {"uri": "https://s3.amazonaws.com/.../setup.sh", "executable": true}
  ],
  "secrets": {
    "service-account-secret": {
      "source": "my-service-acct-secret"
    }
  },
  "networks": [
    {
      "mode": "host"
    }
  ],
  "portDefinitions": [ {
      "name": "web",
      "port": 8080,
      "protocol": "tcp",
      "labels": {
        "VIP_0": "service:8080"
      }
  } ],
  "env": {
    "SERVICE_ACCOUNT": "my-service-acct"
  },
  "cmd": "chmod +x setup.sh && ./setup.sh && nodejs server.js $PORT0"
}

```

For testing you can *dcos task exec ...* into another marathon service and use curl to call the former service.

```console
curl --cert service.crt --key service.key --cacert .ssl/ca-bundle.crt https://service.marathon.l4lb.thisdcos.directory:8080
```


### moustache templating, tls on/off, client authenticaton on/off

*marathon.json*, ...
```js
{
  "id": "service",
  "container": {
    "type": "MESOS",
    "docker": {
      "image": "...",
      "forcePullImage": true
    },
    "volumes": [
      {{#service.tls_enabled}}
      {
        "containerPath": "service_account_secret",
        "secret": "service-account-secret"
      }
      {{/service.tls_enabled}}
    ]
  },
  "fetch": [
    {"uri": "https://downloads.dcos.io/binaries/cli/linux/x86-64/dcos-1.11/dcos", "executable": true},
    {"uri": "https://s3.amazonaws.com/.../setup.sh", "executable": true}
  ],
  {{#service.tls_enabled}}
  "secrets": {
    "service-accountt-secret": {
      "source": "{{service.service_account_secret}}"
    }
  },
  {{/service.tls_enabled}}
  "networks": [
    {
      "mode": "host"
    }
  ],
  "portDefinitions": [ {
      "name": "web",
      "port": 8080,
      "protocol": "tcp",
      "labels": {
        "VIP_0": "service:8080"
      }
  } ],
  "env": {
    "SERVICE_ACCOUNT": "{{service.service_account}}",
    "TLS_ENABLED": {{service.tls_enabled}},
    "CLIENT_AUTH_ENABLED": {{service.client_auth_enabled}}
  },
  "cmd": "./setup.sh && nodejs server.js $PORT0"
}

```

