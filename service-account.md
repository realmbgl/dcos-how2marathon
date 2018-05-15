## service account


### when do you need a service account

A service account is required if your service needs to interact with dc/os, e.g to create a certificate sigining request.

### create service account and service account secret

```console
dcos security org service-accounts keypair priv.pem pub.pem

dcos security org service-accounts create -p pub.pem -d "testing" my-service-acct

dcos security secrets create-sa-secret priv.pem my-service-acct my-service-acct-secret

dcos security org users grant my-service-acct dcos:superuser full
```

### configure the service with service account and service account secret

*marathon.json*, shows configured service account and service account secret
```js
{
  "id": "service",
  ...
  "secrets": {
    "service-account-secret": {
      "source": "my-service-acct-secret"
    }
  },
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
  "env": {
    "SERVICE_ACCOUNT": "my-service-acct"
  },
  "cmd": "..."
}
```

Your service can use the dcos cli to login into dc/os.
```console
cat service_account_secret | jq -r .private_key > service_acct_key.pem 

chmod 600 service_acct_key.pem

dcos cluster setup https://leader.mesos --ca-certs=.ssl/ca-bundle.crt --username=$SERVICE_ACCOUNT --private-key=service_acct_key.pem
```

In the TLS section you will learn how after login you can create certificate sigining requests.

### moustache templating

*marathon.json.mustache*, showing how you can make service account and service account secret configurable.
```js
{
  "id": "service",
  ...
  "secrets": {
    "service-acct-secret": {
      "source": "{{service.service_acct_secret}}"
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
    "SERVICE_ACCOUNT": "{{service.service_acct}}"
  },
  "cmd": "..."
}
```


