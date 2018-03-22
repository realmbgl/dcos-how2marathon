## service account


### when do you need a service account

... like you want to interact with dc/os to create a certificate and key for your service ...

### create service accout and service account secret

```
dcos security org service-accounts keypair priv.pem pub.pem

dcos security org service-accounts create -p pub.pem -d "testing" my-service-acct

dcos security secrets create-sa-secret priv.pem my-service-acct my-service-acct-secret

dcos security org users grant my-service-acct dcos:superuser full
```

### configure the service with service account and service account secret

```
{
  "id": "service",
  ...
  "secrets": {
    "service-acct-secret": {
      "source": "my-service-acct-secret"
    }
  },
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
  "env": {
    "SERVICE_ACCOUNT": "my-service-acct"
  },
  "cmd": "...",
}
```

using it to login to dc/os from your service
```
cat service_account_secret | jq -r .private_key > service_account_key.pem 
chmod 600 service_account_key.pem
dcos cluster setup https://leader.mesos --ca-certs=.ssl/ca-bundle.crt --username=$SERVICE_ACCOUNT --private-key=service_acct_key.pem
```

more on this in the tls section

### moustache templating

...


