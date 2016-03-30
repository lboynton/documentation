Client Common
=============

The client-common package provides some useful tools for working with HTTPlug.
Include them in your project with composer:

.. code-block:: bash

    composer require php-http/client-common "^1.0"

HttpMethodsClient
-----------------

This client wraps the HttpClient and provides convenience methods for common HTTP requests like ``GET`` and ``POST``.
To be able to do that, it also wraps a message factory::

    use Http\Discovery\HttpClientDiscovery;
    use Http\Discovery\MessageFactoryDiscovery

    $client = new HttpMethodsClient(
        HttpClientDiscovery::find(),
        MessageFactoryDiscovery::find()
    );

    $foo = $client->get('http://example.com/foo');
    $bar = $client->get('http://example.com/bar', ['accept-encoding' => 'application/json']);
    $post = $client->post('http://example.com/update', [], 'My post body');

BatchClient
-----------

This client wraps a HttpClient and extends it with the possibility to send an array of requests and to retrieve
their responses as a ``BatchResult``::

    use Http\Discovery\HttpClientDiscovery;
    use Http\Discovery\MessageFactoryDiscovery;

    $messageFactory = MessageFactoryDiscovery::find();

    $requests = [
        $messageFactory->createRequest('GET', 'http://example.com/foo'),
        $messageFactory->createRequest('POST', 'http://example.com/update', [], 'My post body'),
    ];

    $client = new BatchClient(
        HttpClientDiscovery::find()
    );

    $batchResult = $client->sendRequests($requests);

The ``BatchResult`` itself is an object that contains responses for all requests sent.
It provides methods that give appropriate information based on a given request::

    $requests = [
        $messageFactory->createRequest('GET', 'http://example.com/foo'),
        $messageFactory->createRequest('POST', 'http://example.com/update', [], 'My post body'),
    ];

    $batchResult = $client->sendRequests($requests);

    if ($batchResult->hasResponses()) {
        $fooSuccessful = $batchResult->isSuccesful($requests[0]);
        $updateResponse = $batchResult->getResponseFor($request[1]);
    }

If one or more of the requests throw exceptions, they are added to the
``BatchResult`` and the ``BatchClient`` will ultimately throw a
``BatchException`` containing the ``BatchResult`` and therefore its exceptions::

    $requests = [
        $messageFactory->createRequest('GET', 'http://example.com/update'),
    ];

    try {
        $batchResult = $client->sendRequests($requests);
    } catch (BatchException $e) {
        var_dump($e->getResult()->getExceptions());
    }


HTTP Client Router
------------------

This client accepts pairs of clients (both sync and async) and request matchers.
Every request is "routed" through this single client, checked against the request matchers
and sent using the appropriate client. If there is no matching client, an exception is thrown.

This allows to inject a single client into multiple services,
but under the hood there can be clients configured for each of them::

    use Http\Client\Common\HttpClientRouter;
    use Http\Discovery\HttpClientDiscovery;
    use Http\Message\RequestMatcher\RequestMatcher;
    use Http\Discovery\MessageFactoryDiscovery;

    $client = new HttpClientRouter();

    $requestMatcher = new RequestMatcher(null, 'api.example.com');

    $client->addClient(HttpClientDiscovery::find(), $requestMatcher);

    $messageFactory = MessageFactoryDiscovery::find();

    $request = $messageFactory->createRequest('GET', 'http://api.example.com/update');

    // Returns a response
    $client->send($request);

    $request = $messageFactory->createRequest('GET', 'http://api2.example.com/update');

    // Throws an Http\Client\Exception\RequestException
    $client->send($request);
