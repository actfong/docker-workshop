# docker-workshop
docker workshop for beginners

## Hello World
For our Hello World exercise, we will run NGINX in a docker container.

GOALS:
- Verify that it's running (index.html and 50x.html get served)
- Mount volume from your host machine, to serve your own index.html and 50x.html

Commands to run
```
docker pull nginx                 # to pull the nginx image from docker hub
docker images                     # show the list of images available on your machine
docker run -p 80:80 nginx:alpine  # run your nginx image, port forward 80 to 80
```

Now check localost on your browser http://localhost/ and http://localhost/50x.html
```
docker ps                 # (-a) check the running/exited containers
docker rm {container_id}  # remove running container
```
Create your own index.html and 50x.html files

```
mkdir ~/somewhere/nginx_example
cd ~/somewhere/nginx_example
vim index.html
vim 50x.html
```

Run your docker again, but this time mount the files from your host's filesystem onto your container
```
docker run -p 80:80 -v $(pwd):/usr/share/nginx/html nginx:alpine
# run your nginx:alpine image, port forward 80 to 80
```

Commonly used options for `docker run`
```
-p          # port forward from container to host
-v          # mount volume from host onto container
--name      # option to assign a name, which would appear in `docker ps`
--rm        # remove container on exit
```

