## ucr


### why ucr
The [Universal Container Runtime (UCR)](http://mesos.apache.org/documentation/latest/container-image/) launches Mesos containers from binary executables and extends the Mesos container runtime to support provisioning Docker images. The UCR has many [advantages](https://docs.mesosphere.com/1.11/deploying-services/containerizers/) over the Docker Engine for running Docker images. 

### cmd and fetch

*marathon.json*, showing how you can run e.g. jetty without building a docker image. The required binary packages, jre and jetty, are fetched into the sandbox. From the cmd you launch jetty in the foreground.
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

*marathon.json*, showing how you can run an docker image using the ucr. The sample shows the usage of the nginx docker image. From the cmd you launch nginx in the foreground. 
```js
{
  "id": "service",
  ...
  "container": {
    "type": "MESOS",
    "docker": {
      "image": "nginx:latest",
      "forcePullImage": true
    }
  },
  "cmd": "nginx -g 'daemon off;'"
}
```

### cmd and endless loop

*marathon.json*, showing how you can keep the container long running by adding endless loop to the cmd. Its a handy way during development amd exploration of a container. You may have all the artifacts but want to explore configuration and execution in a more interactive way. The next section shows you how to get into the container.
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

The following two command allow you to ssh into a ucr container. Something that is handy during development, but also for later problem determination.

The first command lists you all the running tasks. Pick the id of the task you want to ssh into.

```console
dcos task
```

Using the task-id with the following command gets you to the console of the container running the task.
```console
dcos task exec -ti <task-id> bash
```

