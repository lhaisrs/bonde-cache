Bonde Cache

1. http web server with maximum performance
2. read json from mobilizations
3. parse html to save in bolt at startup script
4. configure bolt as in-memory db
5. serve domains based on custom_domain in mobilizations
6. add worker to listen queue and update bolt cache from mobilization
7. add support to auto tls custom_domain
8. production test with multiple domains and certificates
9. check if request is not empty then not override cache saved previous content
10. enabled grateful shutdown when server is restarted because new domains was added
11. document how to configure persistent storage using dokku to save certificates and db to local file
12. create script to sync local files (db and certificates) to s3
12. create terraform to load ec2 instance 
13. create shell script to setup instance and configure docker to run bonde-cache
14. save to analytics db the views and stats from each request recieved
15. provide some stats by host
 
As ready as possible to shutdown the lights and close the door.

```
docker build -t nossas/bonde-cache .
docker run -it --rm -p 3000:3000 -v "$PWD":/go/src/app -w /go/src/app -e PORT=80 -e CACHE_PORT=80 -e CACHE_PORTSSL=443 -e CACHE_DEV=false -e CACHE_INTERVAL=60 -e CACHE_RESET=false --name bonde-cache-app nossas/bonde-cache

# dev mode with proxy to 3000 to enable auto builds
docker build -f Dockerfile.dev -t nossas/bonde-cache .
docker run -it --rm -p 3000:3000 -v "$PWD":/go/src/app -w /go/src/app -e CACHE_PORT=3001 -e PORT=3001 -e CACHE_PORTSSL=443 -e CACHE_DEV=true -e CACHE_INTERVAL=20 -e CACHE_RESET=false --name bonde-cache-app nossas/bonde-cache```


openssl req -new -newkey rsa:2048 -sha1 -days 3650 -nodes -x509 -subj "/C=US/ST=Georgia/L=Atlanta/O=BNR/CN=www.en.nossas.org" -keyout server.key -out server.crt


Setup app with dokku

```
dokku apps:create 00-cache

dokku config:set CACHE_ENV=production
dokku config:set CACHE_INTERVAL=40
dokku config:set CACHE_PORT=80
dokku config:set CACHE_PORTSSL=443
dokku config:set CACHE_RESET=false
dokku config:set DOKKU_DOCKERFILE_PORTS="443/tcp 80/tcp"
dokku config:set DOKKU_PROXY_PORT_MAP="http:443:443 http:80:80"
dokku config:set PORT=80
dokku config:set AWS_SECRET_ACCESS_KEY=
dokku config:set AWS_ACCESS_KEY_ID=

sudo docker pull nossas/bonde-cache:${DRONE_BRANCH}
sudo docker tag nossas/bonde-cache:${DRONE_BRANCH} dokku/00-cache:latest
dokku tags:deploy 00-cache latest

dokku storage:mount 00-cache /var/lib/dokku/data/storage/cache-certificates:/go/src/app/data/certificates
dokku storage:mount 00-cache /var/lib/dokku/data/storage/cache-db:/go/src/app/data/db

``` 