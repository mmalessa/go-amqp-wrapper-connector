# go-amqp-wrapper-connector

PHP connector for https://github.com/mmalessa/go-amqp-wrapper  

Development version.  
Use it at your own risk.

# Install
//TODO

# Example of use
```php
$connector = new GoAmqpWrapperConnector();

$body = $connector->getBody();
print_r($body);

$connector->ack();
```
