## tls


### creating a certificate request

```
#!/bin/bash
  
# setup dcos cli
chmod +x dcos
export LC_ALL=C.UTF-8
export LANG=C.UTF-

# authN with dcos using service account
cat service_account_secret | jq -r .private_key > service_account_key.pem
chmod 600 service_account_key.pem
./dcos cluster setup https://leader.mesos --ca-certs=.ssl/ca-bundle.crt --username=$SERVICE_ACCOUNT --private-key=service_account_key.pem

# install dcos enterprise cli
./dcos package install dcos-enterprise-cli --yes

# create service crt and key
ID=$(echo $MARATHON_APP_ID | sed 's/\///g')
./dcos security cluster ca newcert --cn $ID.marathon.l4lb.thisdcos.directory --name-c US --name-st CA --name-o "Mesosphere, Inc." --name-l "San Francisco" --key-algo rsa --key-size 2048 --host $ID.marathon.l4lb.thisdcos.directory -j > servicecert.json
cat servicecert.json | jq -r .certificate > service.crt
cat servicecert.json | jq -r .private_key > service.key

```

marathon.json
```
{
  "id": "service",
  "container": {
    "type": "MESOS",
    ...
    "volumes": [
      {
        "containerPath": "service_acct_secret",
        "secret": "service-acct-secret"
      }
    ]
  },
  "fetch": [
    {"uri": "https://downloads.dcos.io/binaries/cli/linux/x86-64/dcos-1.11/dcos"},
    {"uri": "https://s3.amazonaws.com/mbgl-universe/setup.sh"}
  ],
  "secrets": {
    "service-acct-secret": {
      "source": "my-service-acct-secret"
    }
  },
  "env": {
    "SERVICE_ACCOUNT": "my-service-acct"
  },
  "cmd": "chmod +x setup.sh && ./setup.sh && ..."
}

```

### a tls service enpoint implementation

server.js
```
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

marathon.json
```
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
        "containerPath": "service_acct_secret",
        "secret": "service-acct-secret"
      }
    ]
  },
  "fetch": [
    {"uri": "https://downloads.dcos.io/binaries/cli/linux/x86-64/dcos-1.11/dcos"},
    {"uri": "https://s3.amazonaws.com/mbgl-universe/setup.sh"}
  ],
  "secrets": {
    "service-acct-secret": {
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
        "VIP_0": "node-server:8080"
      }
  } ],
  "env": {
    "SERVICE_ACCOUNT": "my-service-acct"
  },
  "cmd": "chmod +x setup.sh && ./setup.sh && nodejs server.js $PORT"
}

```


### a tls service endpoint implementation requiring client authentication

```
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

marathon.json
```
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
        "containerPath": "service_acct_secret",
        "secret": "service-acct-secret"
      }
    ]
  },
  "fetch": [
    {"uri": "https://downloads.dcos.io/binaries/cli/linux/x86-64/dcos-1.11/dcos"},
    {"uri": "https://s3.amazonaws.com/mbgl-universe/setup.sh"}
  ],
  "secrets": {
    "service-acct-secret": {
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
        "VIP_0": "node-server:8080"
      }
  } ],
  "env": {
    "SERVICE_ACCOUNT": "my-service-acct"
  },
  "cmd": "chmod +x setup.sh && ./setup.sh && nodejs server.js $PORT"
}

```

### moustache templating, tls on/off

