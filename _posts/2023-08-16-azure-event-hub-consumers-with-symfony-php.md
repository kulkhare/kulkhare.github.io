---
author: kulkhare
layout: post
title: Azure Event Hub consumers with symfony/php
date: 2023-08-16 21:10 +0530
categories: [Symfony, PHP, Azure, Event Hub, kafka, RdKafka]
tags: [symfont, php, azure, event hub, kafka, rdkafka, consumers]
---

Recently I had a requirement to consume Azure event hubs events using PHP/Symfony.

To my surprise there were a number of SDK's but none for PHP, also as of now there is no REST support for consuming events; doing so requires using the AMQP protocol.

Well, look no further! In this post, I'll guide you through the process of consuming Azure Event Hub messages using the rdkafka extension in a Symfony/PHP application.

Azure Event Hub is a fully managed, real-time data streaming platform that can handle massive amounts of event data from various sources and applications.

Unfortunately, there is no Event Hubs support in the [Azure SDK for PHP](https://github.com/Azure/azure-sdk-for-php/blob/master/README.md), so we are going to use PHP rdkafka extension for Apache Kafka, which is compatible with Azure Event Hub.

Let's dive in!

### Step 1: Set Up Your Azure Event Hub

Before you start consuming messages, you'll need an Azure Event Hub instance. If you don't have one already, head over to the Azure portal and create a new Event Hub namespace with the necessary settings.
I am not going to cover that part in this post assuming you can do that successfully; you will have a connection string looking like this:

> ```Endpoint=sb://<YOUR_NAMESPACE>.windows.net/;SharedAccessKeyName=<YOURA_ACCESS_KEY_NAME>;SharedAccessKey=<YOUR_ACCESS_KEY>;EntityPath=<YOUR_EVENT_PATH>```

### Step 2: Install the librdkafka extension and PECL-package

> ```shell
# Install PHP pecl and pear
sudo apt install php-pear
# Install librdkafka
sudo apt-get install -y librdkafka-dev
# Install PECL-package
sudo pecl install rdkafka
# Enable PHP-extension in PHP config. Add to php.ini
sudo nano /etc/php/7.4/cli/php.ini
# add extension=rdkafka.so
# Restart apache server
sudo service apache2 restart
```

### Step 3: Install packages for symfony
> ```shell
composer require symfony/messenger 
composer require enqueue/rdkafka 
composer require enqueue/enqueue-bundle 
composer require sroze/messenger-enqueue-transport 
composer require enqueue/async-event-dispatcher
```

### Step 4: Register bundles - add to `config/bundles.php`{: .filepath}
>```php
Enqueue\Bundle\EnqueueBundle::class => ['all' => true],
Enqueue\MessengerAdapter\Bundle\EnqueueAdapterBundle::class => ['all' => true],
```

> If you have Symfony 6^ it will be done automatically.
{: .prompt-info }

### Step 5: Add settings in `config/packages/enqueue.yaml`{: .filepath}
>```yaml
enqueue:
    default:
        transport:
            dsn: 'rdkafka://'
            global:
                ### See Kafka documentation regarding `group.id` property if you want to know more
                group.id: 'your-app'
                metadata.broker.list: '%env(resolve:KAFKA_BROKER_LIST)%'
                security.protocol: 'SASL_SSL'
                sasl.mechanisms: 'PLAIN'
                sasl.username: '$ConnectionString'
                sasl.password: '%env(resolve:AZURE_EVENT_MESSENGER_TRANSPORT_DSN)%'
                ### Download the certificate frome here (https://curl.se/docs/caextract.html) and put it somewhere in the app
                ssl.ca.location: 'config/certificates/cacert-2023-05-30.pem'
                log_level: '4' # 1~7
                enable.partition.eof: 'true'
            topic:
                auto.offset.reset: 'earliest'
            commit_async: true
        client: null
```

> Configuration parameters used above can be found [here](https://github.com/confluentinc/librdkafka/blob/master/CONFIGURATION.md)
{: .prompt-info }

### Step 6: Add settings in `config/packages/messenger.yaml`{: .filepath}
```yaml
framework:
    messenger:
        transports:
            azure-event-hub:
                dsn: '%env(AZURE_MESSENGER_TRANSPORT_DSN)%'
            
            failed: 'in-memory://'

        routing:
            'App\Message\DemoEvent': azure-event-hub
```

### Step 7: Add settings to .env
```env
### messenger ###
AZURE_EVENT_MESSENGER_TRANSPORT_DSN=Endpoint=sb://<YOUR_NAMESPACE>.windows.net/;SharedAccessKeyName=<YOURA_ACCESS_KEY_NAME>;SharedAccessKey=<YOUR_ACCESS_KEY>;EntityPath=<YOUR_EVENT_PATH>
### messenger ###

### enqueue rdkafka ###
AZURE_MESSENGER_TRANSPORT_DSN=enqueue://default?topic[name]=messages&queue[name]=messages
KAFKA_BROKER_LIST=example-namesapece.servicebus.windows.net:9093
### enqueue rdkafka ###
```

### Step 8: Event and EventeHandler
```php
// src/Message/DemoEvent.php
namespace App\Message;

class DemoEvent
{
    public function __construct(
        private string $content,
    ) {
    }

    public function getContent(): string
    {
        return $this->content;
    }
}

// src/MessageHandler/DemoEventHandler.php
namespace App\MessageHandler;

use App\Message\DemoEvent;
use Symfony\Component\Messenger\Attribute\AsMessageHandler;

#[AsMessageHandler]
class DemoEventHandler
{
    public function __invoke(DemoEvent $event)
    {
        // ... do some work - like sending an SMS message!
    }
}
```

> For more check Message and MessageHandler from [symfony documentation](https://symfony.com/doc/current/messenger.html)
{: .prompt-info }

### Step 9: Run consumer
```shell
php bin/console messenger:consume azure-event-hub
```

And that's it! You now have a Symfony application set up to consume messages from Azure Event Hub using the rdkafka extension.

Happy coding, and enjoy harnessing the power of real-time data streaming with Azure Event Hub!