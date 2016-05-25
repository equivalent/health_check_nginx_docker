# Health Check Nginx Docker image

Just simple docker container running NginX responding to:

```
curl -v localhost/                  # 404 Not Found
curl -v localhost/health-check      # 200 OK
curl -v localhost/health-check.html # 200 OK
curl -v localhost/healthcheck       # 200 OK
curl -v localhost/healthcheck.html  # 200 OK
```

## Usage example

This is docker image is useful if you are configuring server that is running a worker server
running background tasks independent of the Web-Server.

That means that the BG server does not necessary need to respond to http requests)
yet you still want to expose http health-check endpoint that can be
pinged.


```
docker pull equivalent/health_check_nginx
docker run -p 80:80 -d equivalent/health_check_nginx
```

### AWS Elastic Beanstalk example

If you have AWS ElasticBeanstalk enviroment (EB), you can
configure "healthcheck endpoint"
(e.g.: `http://my-enviromnet-app.elasticbeanstalk.com/health-check.html`).

When that healthcheck is not responding with Status 200 AWS EB will
remove that instance from load balancer or it will raise "red" flang in
server monitoring.

Now here is the useful part. When you are  running Docker environment on AWS ElasticBeanstalk
if any of your `esential` containers is failing (e.g. memory leak) EB will
retriger EB job. That means it effectively restarts
all docker containers in the definition. So this `Nginx` health-check
container will restart as well.

*Dockerrun.aws.json*

```json
{
  "AWSEBDockerrunVersion": 2,

  "containerDefinitions": [
    {
      "name": "background-worker",
      "image": "my-company/my-bg-worker-app",
      "essential": true
    },
    {
      "name": "health_check_nginx",
      "image": "equivalent/health_check_nginx"
      "essential": true,
      "portMappings": [
        {
          "hostPort": 80,
          "containerPort": 80
        }
      ]
    }
  ]
}
```


### Other examples

Just make sure that when your "primary" docker image (e.g. BG
worker) goes down it will bring down this docker container as well (e.g.
specify in `docker-compose.yml` specify `depends_on: my_bg_worker` for
this container.

## Building docker on your local machine

```
git clone git@github.com:equivalent/health_check_nginx_docker.git
docker build -t=nginx-health .
docker run -p 80:80 -it nginx-health
```
