Corser
=======

A highly configurable, middleware compatible implementation of [CORS](http://www.w3.org/TR/cors/) for [Node.js](http://nodejs.org/).

[![Build Status](https://secure.travis-ci.org/agrueneberg/Corser.png)](http://travis-ci.org/agrueneberg/Corser)


Examples
--------

### How to use Corser with `http`

    var http, corser, corserRequestListener;

    http = require("http");
    corser = require("corser");

    // Create Corser request listener.
    corserRequestListener = corser.create();

    http.createServer(function (req, res) {
        // Route req and res through the request listener.
        corserRequestListener(req, res, function () {
            if (req.method === "OPTIONS") {
                // End CORS preflight request.
                res.writeHead(204);
                res.end();
            } else {
                // Your code goes here.
                res.writeHead(200);
                res.end("Nice weather today, huh?");
            }
        });
    }).listen(1337);

### How to use Corser as a middleware in Connect

See `example/connect/` for a working example.

    var connect, corser, app;

    connect = require("connect");
    corser = require("corser");

    app = connect();

    app.use(corser.create());

    app.use(function (req, res) {
        // Finish preflight request.
        if (req.method === "OPTIONS") {
            res.writeHead(204);
            res.end();
        } else {
            res.writeHead(200);
            res.end("Nice weather today, huh?");
        }
    });

    app.listen(1337);

### How to use Corser as a middleware in Express

See `example/express/` for a working example.

    var express, corser, app;

    express = require("express");
    corser = require("corser");

    app = express();

    app.use(corser.create());

    app.options("*", function (req, res) {
        // Finish preflight request.
        res.writeHead(204);
        res.end();
    });

    app.get("/", function (req, res) {
        res.writeHead(200);
        res.end("Nice weather today, huh?");
    });

    app.listen(1337);


API
---

### Creating a Corser request listener

Creating a Corser request listener that generates the appropriate response headers to enable CORS is as simple as:

    corser.create()

This is the equivalent of setting a response header of `Access-Control-Allow-Origin: *`. If you want to restrict the origins, or allow more sophisticated request or response headers, you have to pass a configuration object to `corser.create`.

To give you full control over everything that happens, Corser will not end preflight requests for you. A preflight request is a special `OPTIONS` request that the browser sends under certain conditions to negotiate with the server what methods, request headers and response headers are allowed for a CORS request. If you don't understand the full impact of this and don't use the `OPTIONS` method for any custom stuff, just end those requests manually after they were routed through the request listener:

    // Let the request go through the request listener first.
    corserRequestListener(req, res, function () {
        // CORS preflight requests use the OPTIONS method.
        if (req.method === "OPTIONS") {
            // End CORS preflight request.
            res.writeHead(204);
            res.end();
        } else {

            //
            // Your code goes here.
            //

        }
    });


#### Configuration Object

A configuration object with the following properties can be passed to `corser.create`.

##### `origins`

A case-sensitive whitelist of origins. Unless unbound, if the request comes from an origin that is not in this list, it will not be handled by CORS.

To allow for dynamic origin checking, a function `(origin, callback)` can be passed instead of an array. `origin` is the Origin header, `callback` is a function `(matches)`, where `matches` is a boolean flag that indicates whether the given Origin header matches or not.

Default: unbound, i.e. every origin is accepted.

##### `methods`

An uppercase whitelist of methods. If the request uses a method that is not in this list, it will not be handled by CORS.

Default: simple methods (`GET`, `HEAD`, `POST`).

##### `requestHeaders`

A case-insensitive whitelist of request headers. If the request uses a request header that is not in this list, it will not be handled by CORS.

Default: simple request headers (`Accept`, `Accept-Language`, `Content-Language`, `Content-Type`, `Last-Event-ID`).

##### `responseHeaders`

A case-insensitive whitelist of response headers. Any response header that is not in this list will be filtered out by the user-agent (the browser).

Default: simple response headers (`Cache-Control`, `Content-Language`, `Content-Type`, `Expires`, `Last-Modified`, `Pragma`).

##### `supportsCredentials`

A boolean that indicates if cookie credentials can be transferred as part of a CORS request. Currently, only a few HTML5 elements can benefit from this setting.

Default: `false`.

##### `maxAge`

An integer that indicates the maximum amount of time in seconds that a preflight request is kept in the client-side preflight result cache.

Default: not set.


FAQ
---

### My Ajax call returns `Origin X is not allowed by Access-Control-Allow-Origin`.

Check if the `Origin` header of your request matches one of the origins provided in the `origins` property of the configuration object. If you didn't set any `origins` property, jump to the next question.


### My Ajax call still returns `Origin X is not allowed by Access-Control-Allow-Origin`.

Your request might use a non-simple method or one or more non-simple headers. According to the specification, the set of simple methods is `GET`, `HEAD`, and `POST`, and the set of simple request headers is `Accept`, `Accept-Language`, `Content-Language`, `Content-Type`, and `Last-Event-ID`. If your request uses **any** other method or header, you have to explicitly list them in the `methods` or `requestHeaders` property of the configuration object.


#### Example

You want to allow requests that use an `X-Requested-With` header. Pass the following configuration object to `corser.create`:

    corser.create({
        requestHeaders: corser.simpleRequestHeaders.concat(["X-Requested-With"])
    });


### Getting a response header returns `Refused to get unsafe header "X"`.

Your browser blocks every non-simple response headers that was not explicitly allowed in the preflight request. The set of simple response headers is `Cache-Control`, `Content-Language`, `Content-Type`, `Expires`, `Last-Modified`, `Pragma`. If you want to access **any** other response header, you have to explicitly list them in the `responseHeaders` property of the configuration object.

#### Example

You want to allow clients to read the `ETag` header of a response. Pass the following configuration object to `corser.create`:

    corser.create({
        responseHeaders: corser.simpleResponseHeaders.concat(["ETag"])
    });
