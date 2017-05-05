Api.ai PHP sdk
==============

This is fork of the unofficial php sdk for [Api.ai][1] - [iboldurev/api-ai-php][3]

In this fork, we have added the ability to create, update and delete intents. Specifically, it now has a better support for POST queries as well as PUT and DELETE verbs.

```
Api.ai: Build brand-unique, natural language interactions for bots, applications and devices.
```

## Install:

Via composer:

```
$ composer require davidniki02/api-ai-php
```

## Usage:

Using the low level `Client`:

```php
require_once __DIR__.'/vendor/autoload.php';

use ApiAi\Client;

try {
    $client = new Client('access_token');

    $query = $client->get('query', [
        'query' => 'Hello',
    ]);

    $response = json_decode((string) $query->getBody(), true);
} catch (\Exception $error) {
    echo $error->getMessage();
}
```

Creating a sample intent with a context:

```php
private function createIntent($context, $issue, $response){
    try {
        $client = new \ApiAi\Client('<developer token>');

        $query = $client->post('intents', [
            "name" => "issue name",
            "auto" => true,
            "templates" => [
                $issue
            ],
            "contexts" => [
                $context
            ],
            "userSays" => [
                [
                    "data" => [
                        ["text" => $issue]
                    ],
                    "isTemplate" => false,
                    "count" => 0
                ],
            ],
            "responses" => [
                ["speech" => $response]
            ]
        ]);

        $r = json_decode((string) $query->getBody(), true);
        
        if ($r['status']['code'] != "200")
            return false;
        
        return $r;
    } catch (\Exception $error) {
        return false;
    }
}
```

## Usage:   

Using the low level `Query`:

```php
require_once __DIR__.'/vendor/autoload.php';

use ApiAi\Client;
use ApiAi\Model\Query;
use ApiAi\Method\QueryApi;

try {
    $client = new Client('access_token');
    $queryApi = new QueryApi($client);

    $meaning = $queryApi->extractMeaning('Hello', [
        'sessionId' => '1234567890',
        'lang' => 'en',
    ]);
    $response = new Query($meaning);
} catch (\Exception $error) {
    echo $error->getMessage();
}
```

## Dialog

The `Dialog` class provides an easy way to use the `query` api and execute automatically the chaining steps :

First, you need to create an `ActionMapping` class to customize the actions behavior.

```php
namespace Custom;

class MyActionMapping extends ActionMapping
{
    /**
     * @inheritdoc
     */
    public function action($sessionId, $action, $parameters, $contexts)
    {
        return call_user_func_array(array($this, $action), array($sessionId, $parameters, $contexts));
    }

    /**
     * @inheritdoc
     */
    public function speech($sessionId, $speech, $contexts)
    {
        echo $speech;
    }
    
    /**
     * @inheritdoc
     */
    public function error($sessionId, $error)
    {
        echo $error;
    }
}

```

And using it in the `Dialog` class. 

```php
require_once __DIR__.'/vendor/autoload.php';

use ApiAi\Client;
use ApiAi\Method\QueryApi;
use ApiAi\Dialog;
use Custom\MyActionMapping;

try {
    $client = new Client('access_token');
    $queryApi = new QueryApi($client);
    $actionMapping = new MyActionMapping();
    $dialog = new Dialog($queryApi, $actionMapping);
    
    // Start dialog ..
    $dialog->create('1234567890', 'Привет', 'ru');
    
} catch (\Exception $error) {
    echo $error->getMessage();
}

```

Some examples are describe in the [davidniki02/api-ai-php-example][2] repository.

[1]: https://api.ai
[2]: https://github.com/davidniki02/api-ai-php-example
[3]: https://github.com/iboldurev/api-ai-php-example
