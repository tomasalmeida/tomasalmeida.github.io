---
title: "Mocking a http post server with node.js"
date: 2021-01-06T21:55:47+01:00
draft: false
tags: ["node.js", "tricks", "server"]
categories: ["test"]
ShowToc: true
TocOpen: false
weight: 2
---

Sometimes we need to contact remote servers and send him some data. It is hard to test the whole feature if the remote server does not offer you a sandbox environment.

An easy way to do it is creating a mock http server and for me, the easiest way of doing it is with node.js

## Install node.js

Node.js offers a great [page for downloading](https://nodejs.org/en/download/) it, I prefer to use it the [package manager options](https://nodejs.org/en/download/package-manager/). For macOS, I use brew:

```shell
brew install node
```

## Create your dummy server

Create a `server.js` file and write:

```javascript
var http = require('http');

// get port from arguments or use 8000 by default
var port = process.argv[2] || 8000

var server = http.createServer(function (request, response) {
    
    const { headers, method, url } = request;
    
    console.log('--- HEADERS ---');
    console.log(headers);
    console.log('--------------');
    console.log('Method: ' + method);
    console.log('URL: ' + url);
    console.log('--------------');
    
    // print the data of the request
    request.on('data', (chunk) => {
        console.log('---- BODY ----');
        console.log(chunk.toString());
        console.log('--------------');
    });

    // response status and content-type
    response.statusCode = 200;
    response.setHeader('Content-Type', 'text/plain; charset=utf-8');
    response.setHeader('X-Custom-Header', 'Yes, we have');
    
    // content to answer
    response.write('Thanks ');
    response.end('for the message.\n');
    
});

server.listen(port);

console.log('Server running on port ' + port);
```

Some explanations of this small script:
1. The server is listening to any request (POST, GET, etc..)
2. This server prints the headers, the used method and the url and in other point the body.
3. The answer is a HTTP 200 (OK), 2 headers and some content. 

You can improve and put logic in the answer.

If you found any error or want to help me to improve this article, please [comment the pull request](https://github.com/tomasalmeida/tomasalmeida.github.io/pull/2)

## Reference:
* https://nodejs.org/es/docs/guides/anatomy-of-an-http-transaction