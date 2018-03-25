## ucr

### cmd and fetch

```js
{
  "id": "service",
  ...
  "container": {
    "type": "MESOS"
  },
  "fetch": [
    {"uri": "https://.../server-jre-8u162-linux-x64.tar.gz"},
    {"uri": "http://.../jetty-distribution-9.4.9.v20180320.tar.gz"}
  ],
  "cmd": "cd jetty-distribution-9.4.9.v20180320/demo-base && $MESOS_SANDBOX/jdk1.8.0_162/bin/java -jar ../start.jar"
}
```

### cmd and image

```js
{
  "id": "service",
  ...
  "container": {
    "type": "MESOS",
    "docker": {
      "image": "nginx:latest",
      "forcePullImage": true
    },
  },
  "cmd": "nginx -g 'daemon off;'"
}
```

### cmd and endless loop

```js
{
  "id": "service",
  ...
  "container": {
    "type": "MESOS",
    ...
  },
  "cmd": "... && while :; do sleep 1; done"
}
```

### using container console, interactive container exploration


```console
dcos task
```

```console
dcos task exec -ti <task-id> bash
```

