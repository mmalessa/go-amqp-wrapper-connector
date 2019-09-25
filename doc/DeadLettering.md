# Dead-lettering
https://www.rabbitmq.com/dlx.html#effects

Dead-lettering a message modifies its headers:

    - the exchange name is replaced with that of the latest dead-letter exchange,
    - the routing key may be replaced with that specified in a queue performing dead lettering,
    - if the above happens, the CC header will also be removed, and
    - the BCC header will be removed as per Sender-selected distribution

The dead-lettering process adds an array to the header of each dead-lettered message named x-death.
This array contains an entry for each dead lettering event, identified by a pair of {queue, reason}.
Each such entry is a table that consists of several fields:

    - queue: the name of the queue the message was in before it was dead-lettered
    - reason: reason for dead lettering, see below
    - time: the date and time the message was dead lettered as a 64-bit AMQP 0-9-1 timestamp
    - exchange - the exchange the message was published to (note that this will be a dead letter exchange if the message is dead lettered multiple times)
    - routing-keys: the routing keys (including CC keys but excluding BCC ones) the message was published with
    - count: how many times this message was dead-lettered in this queue for this reason
    - original-expiration (if the message was dead-letterered due to per-message TTL): the original expiration property of the message. The expiration property is removed from the message on dead-lettering in order to prevent it from expiring again in any queues it is routed to.

New entries are prepended to the beginning of the x-death array. 
In case x-death already contains an entry with the same queue and dead lettering reason, 
its count field will be incremented and it will be moved to the beginning of the array.

The reason is a name describing why the message was dead-lettered and is one of the following:

    - rejected: the message was rejected with requeue parameter set to false
    - expired: the message TTL has expired
    - maxlen: the maximum allowed queue length was exceeded

Three top-level headers are added for the very first dead-lettering event. They are

    - x-first-death-reason
    - x-first-death-queue
    - x-first-death-exchange

They have the same values as the reason, queue, and exchange fields of the original dead lettering event. 
Once added, these headers are never modified.

Note that the array is sorted most-recent-first, so the most recent dead-lettering will be recorded 
in the first entry.


  
  
Example:  
```php
Array
(
    [Acknowledger] => Array
        (
        )

    [Headers] => Array
        (
            [x-death] => Array
                (
                    [0] => Array
                        (
                            [count] => 25
                            [exchange] => test-exchange
                            [queue] => test-queue
                            [reason] => rejected
                            [routing-keys] => Array
                                (
                                    [0] =>
                                )

                            [time] => 2019-09-25T19:08:01+02:00
                        )

                )

        )

    [ContentType] =>
    [ContentEncoding] =>
    [DeliveryMode] => 1
    [Priority] => 0
    [CorrelationId] =>
    [ReplyTo] =>
    [Expiration] =>
    [MessageId] =>
    [Timestamp] => 0001-01-01T00:00:00Z
    [Type] =>
    [UserId] =>
    [AppId] =>
    [ConsumerTag] => test-consumer
    [MessageCount] => 0
    [DeliveryTag] => 26
    [Redelivered] =>
    [Exchange] => test-exchange
    [RoutingKey] =>
    [Body] => ZmFmYWZh
)
```
