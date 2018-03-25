## networking


### host networking

marathon.json
```js
{
  "id": "service",
  "container": {
    "type": "MESOS",
    ...
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
  "cmd": "<executable> $PORT0"
}

```

### virtual networking

marathon.json
```js
{
  "id": "service",
  "container": {
    "type": "MESOS",
    ...
    "portMappings": [
        {
            "containerPort": 8080,
            "servicePort": 8080,
            "protocol": "tcp",
            "labels": {
              "VIP_0": "service:8080"
            }
        }
    ],
  },
  "networks": [
    {
      "name": "dcos",
      "mode": "container"
    }
  ],
  "cmd": "<executable> $PORT0"
}

```

### moustache templating, virtual networking on/off
