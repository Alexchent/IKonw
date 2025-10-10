# plusar 

> https://pulsar.apache.org/docs/3.0.x/client-libraries-go/
## docker 部署
```bash
docker run -it -p 6650:6650 -p 8080:8080 --mount source=pulsardata,target=/pulsar/data --mount source=pulsarconf,target=/pulsar/conf apachepulsar/pulsar:3.0.0 bin/pulsar standalone
```


建立客户端链接
```golang
pulsarClient, err = pulsar.NewClient(pulsar.ClientOptions{
	URL: "pulsar://localhost:6550",
	//URL: "pulsar://localhost:6550,localhost:6651,localhost:6652",
})
defer pulsarClient.Close()
```

### py-consumer
```python
import pulsar

client = pulsar.Client('pulsar://localhost:6650')
consumer = client.subscribe('log', subscription_name='my-sub')

while True:
    msg = consumer.receive()
    print("Received message: '%s'" % msg.data())
    consumer.acknowledge(msg)

client.close()
```

### py-producer
```python
import pulsar

client = pulsar.Client('pulsar://localhost:6650')
producer = client.create_producer('log')

for i in range(10):
    producer.send(('hello-pulsar-%d' % i).encode('utf-8'))

client.close()
```


