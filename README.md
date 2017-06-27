# docker-workshop
docker workshop for beginners

## Part 1: Hello World
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
Now check the changes on http://localhost/ and http://localhost/50x.html

Commonly used options for `docker run`
```
-p          # port forward from container to host
-v          # mount volume from host onto container
--name      # option to assign a name, which would appear in `docker ps`
--rm        # remove container on exit
```

Think Image vs Containers as Classes vs Instances!

#### Mini Challenge
Try to run two instances of NGINX: one on localhost:8000 and another on localhost:8001. Verify that they are two separate instances by serving different static pages.

- - - -
 
## Part 2: Create your own image
For this part of the workshop, we will create our own image, containing our own application.

### 1. Get an app
To get started, clone an ready-made sinatra application with `git clone `

### 2. Dockerfile
In this repo, create a `Dockerfile`. Pay attention to the capital `D`

`FROM`

`Dockerfile` always starts with a `FROM` command, indicating the base image for our own image. 
Be specific about the version of the image. When not specified, the `latest` tag will be used.
Since we are building an image for our Ruby application, have a look at https://hub.docker.com/_/ruby/ 

`RUN` 

`RUN` instructions are used to run commands against the images that we are building. Typically, one of the first things you would do is to perform a `apt-get` or `yum` update.

This would also result in new layers in the image (LFS => Layered File System). This can be demonstrated that by installing additional software packages later.

`WORKDIR` 

Once the working directory has been specified, commands such as `RUN`, `ADD`, `COPY`, `ENTRYPOINT` and `CMD` will be run from that directory. 
For a web-application, this is typically the root directory your app.

`ADD` or `COPY` 

These commands puts files from your filesystem into the image's filesystem. These commands would also create a layer in the LFS. `ADD` can take URL's as arguments.

`EXPOSE` 

Allows the container to expose a port to your host. At runtime, the `-p` option is still necessary to perform port-forwarding.

`ENTRYPOINT` and `CMD` 

NOTE: If you want to keep it simple, stick to `CMD`

Both can be used to set the default command (such as `npm run development`) and there are no hard rules regarding which one is best.

Difference: the commands specifed in `ENTRYPOINT` will always be executed by `docker run` command, while the `CMD` commands can be overridden by passing arguments to the `docker run` command. 
When both are specified, `CMD` will be used as arguments to `ENTRYPOINT`

Example:
```
ENTRYPOINT ["npm", "run"]
CMD ["production"]
```
In this case above, if you provide 'development' as argument to `docker run`, "production" will be overridden by "development". 
However, whenever you run a container built with the ENTRYPOINT and CMD from above, the command `npm run` will always be run, regardless of arguments you provide.

If you don't use ENTRYPOINT:
```
CMD ["npm", "run", "production"]
```
In this case, you could provide anything to the `docker run` command... whether it is `/bin/bash` to `ls /etc`.

In the FromAtoB universe, it seems we have settle the the latter.

By now, you should be abe to create your own Dockerfile!


### 3. Build your image
`docker build -t {username/image-name:version} .`

The `.` (current location) implies that there is a `Dockerfile` in your current directory. Else you would have to provide another directory.
`-t` allows you to tag your image

While building your image, pay attention to the layers being created, as well as the time it takes. (This has to do with the Layered File System)

#### Mini Challenge
Let's say we want to install the `whois` package into our image. How would you do it?

Once you have figure it out, build your image again and see whether the layers (id's) are still the same as the last time.
Also, pay attention to the build time.
