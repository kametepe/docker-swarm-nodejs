# docker-swarm-nodejs
a sample of Docker Swarm with nodejs


Create a new files index.js and paste the code below : 

```node
var http = require('http');
var os = require('os');

http.createServer(function (req, res) {
    res.writeHead(200, {'Content-Type': 'text/html'});
    res.end(`<h1>I'm ${os.hostname()}</h1>`);
}).listen(8080);
```
