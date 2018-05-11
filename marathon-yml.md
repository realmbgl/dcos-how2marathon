## using marathon yml

Use yml instead of json to author your marathon apps/services. It is much less verbose and is especially handy when you have to author multi-line commands.


### install yml2json

Download the yml2json command.
```
curl -O https://s3.amazonaws.com/mbgl-universe/yml2json && chmod +x yml2json
```

Move it to a location that is on your PATH.
```
mv yml2json /usr/local/bin
```

### run it

Author you marathon.yml file, then run it.
```
yml2json marathon.yml | dcos marathon app add
```

### samples

A kafka producer service.
```
id: k-producer
cpus: 0.1
mem: 1024
container:
  type: MESOS
  docker:
    image: python:latest
cmd: |
  env | sort
  pip install kafka-python

  cat > function.py << EOF

  from kafka import KafkaProducer
  producer = KafkaProducer(bootstrap_servers='broker.kafka.l4lb.thisdcos.directory:9092')
  for i in range(10000):
      producer.send('hello', b'hello %d' % i)

  EOF

  python function.py
```

A kafka consumer service.
```
id: k-consumer
cpus: 0.1
mem: 1024
container:
  type: MESOS
  docker: 
    image: python:latest
cmd: |
  env | sort
  pip install kafka-python

  cat > function.py << EOF

  from kafka import KafkaConsumer
  consumer = KafkaConsumer('hello', bootstrap_servers='broker.kafka.l4lb.thisdcos.directory:9092')
  for msg in consumer:
      print(msg)

  EOF

  python function.py
```
