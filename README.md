# docker-workshop
docker workshop for beginners

*GOAL*:

At the end of the workshop, participants should feel confident about :
- creating their own images
- run/stop/remove containers
- allow containers to communicate with each other
- write his/her own docker-compose

Pre-requisite
- Make sure you have docker installed on your machine. Whether you are using the old docker-machine or the new Docker for Mac.
- register account with Docker Hub

- - - - 

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
docker ps                      # (-a) check the running/exited containers
docker stop {container_id}     # stop running container
docker restart {container_id}  # restart running container
docker exec -it {container_id} /sh  # grab a shell
docker rm {container_id}       # remove running container
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

### Mini Challenge
Try to run two instances of NGINX: one on localhost:8000 and another on localhost:8001. Verify that they are two separate instances by serving different static pages.

Once everything works (and verified), you can remove your nginx image with

```
docker rmi {image-name}
```

### What you have learned in this section
1. A bunch of image commands:
```
docker pull {image-name}
docker images (try the filter option!)
docker rmi {image-name}
```

2. A bunch of container commands:
```
docker run (with various options)
docker ps
docker rm
```

3. Difference between Image and Container

- - - -
 
## Part 2: Create your own image
For this part of the workshop, we will create our own image, containing our own application.

GOALS:
- Understand Dockerfile
- Build your own image
- Basic understanding of the Layered File System
- Run your own image as a container. Run bash within your container.

### 1. Get an app
To get started, clone an ready-made sinatra application with `git clone git@github.com:actfong/docker-workshop.git`

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

Also, for inspiration, check the Dockerfiles used to create the official NGINX and Ruby images on Docker Hub.

By now, you should be abe to create your own Dockerfile!


### 3. Build your image
`docker build -t {username/image-name:version} .`

The `.` (current location) implies that there is a `Dockerfile` in your current directory. Else you would have to provide another directory.
`-t` allows you to tag your image

While building your image, pay attention to the layers being created, as well as the time it takes. (This has to do with the Layered File System)

### Challenges

#### Mini Challenge 1
Let's say we want to install the `whois` package into our image. How would you do it?

Once you have figure it out, build your image again and see whether the layers (id's) are still the same as the last time.
Also, pay attention to the build time.

#### Mini Challenge 2 
Run your brand new image as a container. Can you run `bash` within your container? Can you find out which process has PID 1?

### What you have learned in this section
1. Write your own Dockerfile, with various commands such as `FROM`, `RUN`, `WORKDIR`, `ADD`, `COPY`, `EXPOSE`, `ENTRYPOINT` and `CMD`.
2. Build your own image with `docker build` and tag it with `-t`
3. Layered File System (LFS)
4. PID 1 in Docker containers

- - - -

## Part 3: Networking
GOALS:
- Being able to run multiple containers: One containing your Ruby app, the other one a Redis
- Verify that a connection has been established. In our case, we will use `redis` gem to verify that (, but feel free to use  `telnet`).


### 1 Create and list your networks
```
docker network ls                         # list existing networks
docker network create {network-name}      # create your network
```

### 2 Ensure that your container runs inside a specific network

Now run your container (`docker run`) containing the Sinatra app with the following option
```
--net={network-name}
```

You can now verify that your container is indeed connected to your network by:
```
docker network inspect {network-name}
```

### Challenges:

#### Mini Challenge 1
By now, you should have the skills to pull the *redis* image and run it as a container.
Can you run your container in such a way, that it is attached to your network (`--net`)?

Another important note, to allow connections from other containers in the same network, they also need some kind of a *hostname*. 
When running containers in a network, the name that you provide with `--name` can be used as a hostname.

#### Mini Challenge 2
Please install the `redis` gem from https://github.com/redis/redis-rb by adding it to your Gemfile and run `bundle install`
(TIP: There are various ways to do that!)

Once you have installed the `redis` gem, can you use `irb` within your sinatra-app container to set and get values on the Redis container that you just created? 

One tip: 
```
require 'redis'
redis = Redis.new(:host => {name-of-your-redis-container}, :port => 6379)
redis.get "foo"
redis.set "foo", "bar"
redis.get "foo"        # should return bar
```

If you can set and get values from your IRB console to redis, that means you have succesfully "networked" the two containers.

Also take a look in 
```
docker network {network-name} inspect
```
, where the attached containers are listed.

#### Mini Challenge 3
Redis clients (such as the installed `redis` gem) has a function called `ping`, when the connection can be established, it return `PONG`.

Let's create a route in your sinatra-app called `/redis-status`. We will use this route to see whether your Redis instance is up.

Task: Display the result from your `ping` onto the page as plain text.

### What you have learned in this section
1. A bunch of commands for docker networks
```
docker network create {network-name}
docker network ls
docker network inspect
docker network rm {network-name}
```

2. How to run a container in a specific network
```
docker run --net {network-name} {image-name}     # potentially along with some other options
```

3. How to lookup a container within a network. The container's name provided in:
```
docker run --name {container-name}
```
can be used as a hostname within a network.
- - - -

## Part 4: Docker Compose
GOALS:
- Basic understanding of docker-compose
- Being able to run multiple containers without running separate `docker run` commands

By the time you have a platform containing multiple services (containers), it no longer makes sense to run `docker run` for each container. 
Instead, you can configure multiple containers in such a way, that you can run the whole stack using one command: `docker-compose up`

### docker-compose.yml
A basic structure can be found here => https://docs.docker.com/compose/overview/

`services` 
It contains a list of containers that you want to run. The first level below `services` are names that you want to give to your containers. These names will also be used for lookup within your network

`build` and `image`
Indicates the location of your Dockerfile. 
Only needed if you need docker-compose to build your image. 
Otherwise, just spefified `image` with necessary tags is enough

`ports` 
Takes care of port forwarding

`networks`
They can appear at top-level and within your service.

At top level, it allows you to create a network from scratch. 
Once created, you can let your containers (services) attach to it by again defining `networks` under the name of your `service`, along with `ports`, `volumes` etc.

`volumes`
Actually... I don't know enough about this :)

Within a `service`, it allows you to mount volumes from host into your image.
A top level, AFAIK, you can share volume between containers.

### Challenge
By now, you should know enough to write your own `docker-compose.yml`.

Your challenge now is to write your `docker-compose.yml`, which includes your Sinatra app and redis.
Ensure that when you run `docker-compose up`, it will start your Sinatra app.
Then verify that the pages it serves at http://localhost/ and http://localhost/redis-status are behaving as you expected.

Tear down your whole setup with `docker-compose down`. Check the status of your containers and networks. What would you need to do to clean everything up?

### What you have learned in this section
Basic understang of `docker-compose`

## The End
This is it! You got all the basics to get you going in Docker.
In your spare time, try to build images for your applications and boot them up with `docker-compose`
