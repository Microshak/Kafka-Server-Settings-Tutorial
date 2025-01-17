# Getting started

```sh
mkdir Producer
cd Producer
dotnet new console
dotnet add package Confluent.Kafka --version 1.7.0

```
## Create a producer
Open the project and put int the following code 

```dotnet
  using System;
using Confluent.Kafka;
using System.Net;
using System.Threading.Tasks;

namespace CCDAK
{
    class Program
    {
        static void  Main(string[] args) 
        {
            var pConfig = new ProducerConfig
            {
                BootstrapServers = "localhost",
                ClientId = Dns.GetHostName(),
            };

            ProduceAsync(pConfig).GetAwaiter().GetResult();
        }


         static async Task ProduceAsync(ProducerConfig pConfig)
         {
            using (var producer = new ProducerBuilder<string, string>(pConfig).Build())
            {
                var dr = await producer.ProduceAsync("myTopic", new Message<string, string> { Key = "1", Value = "Some Data" } )  ;
                Console.WriteLine($"Delivered '{dr.Value}' to '{dr.TopicPartitionOffset}'");
                
            }
         }
      }
}
```

## Create a consumer
cd into your Consumer directory


```sh
mkdir Consumer
cd Consumer
dotnet new console
dotnet add package Confluent.Kafka --version 1.7.0

```

Then in your program.cs file enter in: 

```dotnet

using Confluent.Kafka;
using System;
using System.Threading;


namespace KafakConsumer
{
    class Program
    {
        static void Main(string[] args)
        {
            var cConfig = new ConsumerConfig
            {
                BootstrapServers = "localhost",
                GroupId = "amikesGroupz",
            };

            using (var consumer = new ConsumerBuilder<string, string>(cConfig).Build())
               {
                consumer.Subscribe("myTopic");
                var consumeResult = consumer.Consume();
                Console.WriteLine($"consumed: {consumeResult.Message.Value}   partition: {consumeResult.Partition}");
            }
            
        }
    }
}

```

# Producer Properties

## Back Off Retry
Backoff allows you to resend a message if there is a failure.

> If you are trying to send messages in order and using async this can cause racetrack errors. You may want to ensure that your message is sent before the next one is sent.

```
   RetryBackoffMs = 1000,
```

## Idempotence
When a producer does not get an acknowledgement of the message it could be because a) the message never made it to the broker or b) the success message never made it back to the producer.  Idempotence puts a number on the message that is unique allowing the producer to replay the message without having to worry about duplicates.

```
    EnableIdempotence = true,
```



## ACKS
Acknowledgements now many servers need to respond to the producer before it responds.  These are:
* None
* 1
* All

## linger.ms 
During a certain time when records are being batched, the producer will wait a set period of time before sending records in order​

```Having a slight delay increases throughput since the Producer will wait a little bit longer to be able to send more records```

```
LingerMs = 2300,
```

## batch.size 
The size of the records being sent to a specific partition

```
BatchSize = 1000,
```


## compression.type
The compression type for data generated by the producer ​
```
CompressionType = CompressionType.Gzip,
```


## max.in.flight.requests.per.connection  
The maximum number of unacknowledged requests the client will send on a single connection before blocking. Having this value set to a higher value when retries is also high causes message ordering not to be preserved​
```
                MaxInFlight = 11
```


# Consumer Properties

## session.timeout.ms 

Timeout used to detect client failures ​
```
SessionTimeoutMs = 9000
```


## consumer.max.poll.interval.ms 
Delay between invocations when using Consumer Groups ​

```
MaxPollIntervalMs = 9000

```

## consumer.max.poll.records 
Max records to return in a single poll​

## consumer.fetch.max.wait.ms 
How long to wait before polling records ​
```
FetchWaitMaxMs = 1000

```


## group.id 

The consumer group id​

```
GroupId = "mikesGroup"

```

## auto.offset.reset 
Where the consumer reads messages from ​

```
AutoOffsetReset = AutoOffsetReset.Latest
```

## heartbeat.interval.ms  
How long to send a heartbeat to the Group Coordinator (1/3 lower than session.timeout.ms)
```
HeartbeatIntervalMs = 9000

```

## enable.auto.commit 

If offsets will be committed automatically or manually 
```
EnableAutoCommit = true

```
