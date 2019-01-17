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


Now we need to dockerize the app, so we’ll create a file named Dockerfile with the following code:


```docker
FROM node
RUN mkdir -p /usr/src/app
COPY index.js /usr/src/app
EXPOSE 8080
CMD [ "node", "/usr/src/app/index" ]
```

To build a docker image of our newly testimony Node.js app from this docker file instructions we’ll write docker build command line, where our Dockerfile file is located:

```bash
$ docker build -t testimony .
```


Now we have a docker image of our simple (and testimony) Node.js app, and we can create containers from that image.

Let's say we want 20 running containers of that image and all behind a load balancing server.
For our HTTP server we’ll use HAProxy that will listen to port 85
Let’s create a docker-compose.yml file:

```docker
version: '3'

services:
  awesome:
   image: testimony
   ports:
     - 8080
   environment:
     - SERVICE_PORTS=8080
   deploy:
     replicas: 20
     update_config:
       parallelism: 5
       delay: 10s
     restart_policy:
       condition: on-failure
       max_attempts: 3
       window: 120s
   networks:
     - web

  proxy:
    image: dockercloud/haproxy
    depends_on:
      - testimony
    environment:
      - BALANCE=leastconn
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    ports:
      - 85:80
    networks:
      - web
    deploy:
      placement:
        constraints: [node.role == manager]

networks:
  web:
    driver: overlay
 ```
 
 
 
 Now let’s create a swarm (with one computer for now, but you can easily add more to the swarm). To do this we'll write docker swarm init and we created a swarm!! 
 
 ```bash
 docker swarm init
 ```
 
 It’s also added our current computer to the swarm, and since our computer is the first it’s also the manager of the swarm.
 
 Let's build the stack with the following comand line :
 
 ```bash
 docker stack deploy --compose-file=docker-compose.yml production
 ```
 We are doing two things : 
  1 - Build the services 
  2 - deploy them to our local stack called production
  
 Once done you browse http://localhost:85 and see the nodejs message, 
 
 Hitting F5 will display a different hostname since HAproxy will load balance the request.
 
 
