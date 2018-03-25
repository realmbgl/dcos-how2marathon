## healthchecks


### without tls

*marathon.json*, ...
```js
{
  "id": "service",
  "container": {
    "type": "MESOS",
    ...
  },
  ...
  "cmd": "...",
  "healthChecks": [
    {
      "protocol": "COMMAND",
      "command": { "value": "curl -I http://localhost:$PORT0 | grep \"HTTP/1.1 200 OK\"" },
      "gracePeriodSeconds": 180, "intervalSeconds": 10, "timeoutSeconds": 20, "maxConsecutiveFailures": 
    }
  ]
}
```


### with tls

*marathon.json*, ...
```js
{
  "id": "service",
  "container": {
    "type": "MESOS",
    ...
  },
  ...
  "cmd": "...",
  "healthChecks": [
    {
      "protocol": "COMMAND",
      "command": { "value": "curl --cacert $MESOS_SANDBOX/.ssl/ca-bundle.crt -I https://localhost:$PORT0 | grep \"HTTP/1.1 200 OK\"" },
      "gracePeriodSeconds": 180, "intervalSeconds": 10, "timeoutSeconds": 20, "maxConsecutiveFailures": 3
    }
  ]
}
```


### with tls and client authentication

*marathon.json*, ...
```js
{
  "id": "service",
  "container": {
    "type": "MESOS",
    ...
  },
  ...
  "cmd": "...",
  "healthChecks": [
    {
      "protocol": "COMMAND",
      "command": { "value": "curl --cert $MESOS_SANDBOX/service.crt --key $MESOS_SANDBOX/service.key --cacert $MESOS_SANDBOX/.ssl/ca-bundle.crt -I https://localhost:$PORT0 | grep \"HTTP/1.1 200 OK\"" },
      "gracePeriodSeconds": 180, "intervalSeconds": 10, "timeoutSeconds": 20, "maxConsecutiveFailures": 3
    }
  ]
}
```


### moustache templating, tls on/off, client authenticaton on/off

*marathon.json.moustache*, ...
```js
{
  "id": "service",
  "container": {
    "type": "MESOS",
    ...
  },
  ...
  "cmd": "...",
  "healthChecks": [
    {
      "protocol": "COMMAND",
      {{^service.tls_enabled}}
      "command": { "value": "curl -I http://localhost:$PORT0 | grep \"HTTP/1.1 200 OK\"" },
      {{/service.tls_enabled}}
      {{#service.tls_enabled}}
      "command": { "value": "curl --cert $MESOS_SANDBOX/service.crt --key $MESOS_SANDBOX/service.key --cacert $MESOS_SANDBOX/.ssl/ca-bundle.crt -I https://localhost:$PORT0 | grep \"HTTP/1.1 200 OK\"" },
      {{/service.tls_enabled}}
      "gracePeriodSeconds": 180, "intervalSeconds": 10, "timeoutSeconds": 20, "maxConsecutiveFailures": 3
    }
  ]
}
```


