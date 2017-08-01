# docker-workshop
*Description*:

docker workshop for beginners

*GOAL*:

At the end of the workshop, participants should feel confident about :
- creating their own images
- run/stop/remove containers
- allow containers to communicate with each other
- write his/her own docker-compose

## Prerequisites
1. Watch this video to get an idea of what containers are: 

[![Watch this video](https://i.ytimg.com/vi/EnJ7qX9fkcU/default.jpg)](https://www.youtube.com/watch?v=EnJ7qX9fkcU)

2. Make sure you have docker installed on your machine. Whether you are using [docker-machine](https://docs.docker.com/machine/overview/) or something like [Docker for Mac](https://docs.docker.com/docker-for-mac/).

3. Register account with Docker Hub

- - - - 

## Part 1: Hello World
For our Hello World exercise, we will run NGINX in a docker container.

GOALS:
- Verify that our containers are running (hence *index.html* and *50x.html* get served)
- Mount volume from your host machine, to serve your own *index.html* and *50x.html*

Commands to run
```
docker pull nginx:latest            # pull the nginx image (tagged with 'latest') from docker hub
docker images                       # show the list of images available on your machine
docker run -p 80:80 nginx:latest    # run the nginx image your just pulled, port forward 80 to 80. 
```

Now check localost on your browser http://localhost/ and http://localhost/50x.html

In case you are running `docker-machine`, get the ip with `docker-machine ip {machine-name}` (without the curly brackets).

Then open another tab and try out some of the following commands:
```
docker ps                                # (-a) check the running/exited containers
docker stop {container_id}               # stop running container
docker restart {container_id}            # restart running container
docker exec -it {container_id} /bin/sh   # grab a shell to execute commands in a container
docker rm {container_id}                 # remove running container
```

Now that you have familiarized with some commands related to *containers*, we will proceed to serve our own static files with nginx.

First, stop your running container, either by `ctrl+c` in the terminal running you nginx or execute `docker stop {container_id}` in another tab.

Then create your own index.html and 50x.html files

```
mkdir ~/somewhere/nginx_example
cd ~/somewhere/nginx_example
echo "My own index page" > index.html
echo "My own 50x page" > 50x.html
```

Run your docker again, but this time mount the files from your host's filesystem onto your container (`-v` option)
```
docker run -p 80:80 -v $(pwd):/usr/share/nginx/html nginx:latest
# run your nginx:latest image, port forward 80 on host to 80 in container
# $(pwd) prints the current working directory
```
Now check the changes on http://localhost/ and http://localhost/50x.html (or replace localhost with docker-machine's ip)

Commonly used options for **`docker run`**
```
-p          # port forward from container to host
-v          # mount volume from host onto container
--name      # option to assign a name, which would appear in `docker ps` instead of a randomly generated name
--rm        # remove container on exit
-it         # interactive mode and grab TTY. Useful for interactions, not useful for running web-apps or daemons
-d          # daemon mode
```

So to wrap up, we have pulled the nginx *image* and based on that image, we ran a nginx *container*.

The main takeaway is to think Image vs Containers as Classes vs Objects! (like OO-programming)


### Mini Challenge
Try to run two instances of NGINX: one on localhost:8001 and another on localhost:8002. This time, also make sure that they run in the backgroud (daemon).

Verify that they are two separate instances by serving different static pages.

<details>
  <summary>Possible solution:</summary>
  <p>

```
cd ~/somewhere
mkdir nginx1 nginx2
echo "Welcome to Nginx 1" > nginx1/index.html
echo "Welcome to Nginx 2" > nginx2/index.html
echo "50x on Nginx 1" > nginx1/50x.html
echo "50x on Nginx 2" > nginx2/50x.html
docker run -d  -v $(pwd)/nginx1:/usr/share/nginx/html -p 8001:80 nginx:latest
docker run -d  -v $(pwd)/nginx2:/usr/share/nginx/html -p 8002:80 nginx:latest
```

</p></details>
<br/>

Once you have verified that both NGINX containers serve the correct static pages, find the container-ids and subsequently stop and remove those containers
```
docker ps                            # get the container ids of your nginx containers
docker stop {container-ids}          # stop your containers
docker rm {container-ids}            # remove your containers
```

If you want, you can now remove your nginx image with:

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
docker stop
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
To get started, clone an ready-made sinatra application with:
```
git clone git@github.com:actfong/docker-workshop.git
```

### 2. Dockerfile
In this repo, create a `Dockerfile`. Pay attention to the capital `D`

`FROM`

`Dockerfile` always starts with a `FROM` command, indicating the base image for our own image. 
Be specific about the version of the image. When not specified, the `latest` tag will be used.
Since we are building an image for our Ruby application, have a look at https://hub.docker.com/_/ruby/ 

*Good to know: Pay attention to the various versions and their sizes under the **tags** tab. The ones end with **alpine** and **slim** are much smaller in size. I'd strongly advise you to look at their own Dockerfiles to figure out why that is the case.*


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


When both are specified, `CMD` will be used as arguments to `ENTRYPOINT`. Example:
```
ENTRYPOINT ["npm", "run"]
CMD ["development"]
```

In the case above, if you run `docker run {image-name} production`, the argument that you passed in ("production") will override "development" as specified in CMD. 

The main difference between them is that `ENTRYPOINT` will always be run unless `--entrypoint` option is provided to `docker run`. `CMD` on the other hand, can be overridden by providing arguments to the `docker run` command.

If you don't use ENTRYPOINT:
```
CMD ["npm", "run", "production"]               # node way
# or
CMD ["ruby", "my-app.rb", "-o", "0.0.0.0"]     # sinatra way
```

In this case, you could provide anything to the `docker run` command... whether it is `/bin/bash` to `ls /etc`.

In the FromAtoB universe, it seems we have settle the the latter.

Also, for inspiration, check the Dockerfiles used to create the official [NGINX](https://hub.docker.com/_/nginx/) and [Ruby](https://hub.docker.com/_/ruby/) images on Docker Hub.

By now, you should be abe to create your own Dockerfile!


### 3. Build your image
`docker build -t {username/image-name:version} .`

The `.` (current location) implies that there is a `Dockerfile` in your current directory. Else you would have to provide another directory.
`-t` allows you to tag your image

While building your image, pay attention to the layers being created, as well as the time it takes. (This has to do with the Layered File System)

<details>
  <summary>Possible solution:</summary>
  <p>

```
### Dockerfile

# Specify base image
FROM ruby:2.3.3-slim

# update packages
RUN apt-get update
ENV APP_DIR /app
RUN mkdir -p $APP_DIR
WORKDIR $APP_DIR

# Add Gemfile and Gemfile.lock
ADD Gemfile* $APP_DIR/

# Install dependencies for our Sinatra app
RUN bundle install

# Add rest of the code-base to the working directory
ADD . $APP_DIR/

# Expose the port where our Sinatra app is running
EXPOSE 4567

# start our server
CMD ["ruby", "docker-newbies.rb", "-o", "0.0.0.0"]

```

  </p>
</details>

### Challenges

#### Mini Challenge 1
Let's say we want to install the `whois` package into our image. How would you do it? Tip: check [Dockerfile Best practices](https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/#run)

Once you have figured it out, build your image again and see whether the image-layers (id's) are still the same as the last time.
Also, pay attention to the build time.

#### Mini Challenge 2 
Run your brand new image as a container. Can you use `bash` within the container you just spun up? Can you find out which process has PID 1?

<details>
  <summary>Possible solution:</summary>
  <p>
  
```shell
docker run --rm -p 4567:4567 --name {container-name} -d {image}
docker exec -it {container-name} /bin/bash

# Within the container
$ ps aux
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.3  1.0  97172 22144 ?        Ssl  06:52   0:00 ruby docker-newbies.rb -o 0.0.0.0

# Get out of the container with ctrl-d, then
docker stop {container-name}
# since we ran with the --rm option, the container will automatically be removed when stopped
```

PID 1 should be the same command as specified in your CMD or ENTRYPOINT (unless you override it with options during `docker run`). 

This is the whole point of containerization: **Each container should have only one process. And when that process exits, the container will be stopped.**

  </p>
</details>


### What you have learned in this section
1. Write your own Dockerfile, with various commands such as `FROM`, `RUN`, `WORKDIR`, `ADD`, `COPY`, `EXPOSE`, `ENTRYPOINT` and `CMD`.
2. Build your own image with `docker build` and tag it with `-t`
3. Layered File System (LFS)
4. PID 1 in Docker containers

- - - -

## Part 3: Networking
GOALS:
- Being able to run multiple containers: One containing your Ruby app, the other one a Redis
- Verify that a connection has been established. In our case, we will use `redis` gem to verify that (, but feel free to use  `telnet`. And figure out for yourself how to install telnet on a debian distro).


### 1 Create and list your networks
```
docker network ls                         # list existing networks
docker network create {network-name}      # create your network
```

### 2 Ensure that your container runs inside a specific network

Now run your container (`docker run`) containing the Sinatra app with the following option
```
--net {network-name}
```

You can now verify that your container is indeed connected to your network by:
```
docker network inspect {network-name}
```

### Challenges:

#### Mini Challenge 1
By now, you should have the skills to *pull* the *redis* image and run it as a container.
Can you run your container in such a way, that it is attached to your network (`--net`)?

Another important note, to allow connections from other containers in the same network, they also need some kind of a *hostname*. 
When running containers in a network, the name that you provide with `--name` can be used as a hostname.

<details>
  <summary>Possible solution:</summary>
  <p>

```
docker network create {network-name}
docker pull redis:latest
docker run -d --rm --net {network-name} --name {container-name-redis} redis:latest
docker network inspect {network-name}
# Within the `Containers` section, you should see your redis container
```

  </p>
</details>

#### Mini Challenge 2
Please install the `redis` gem from https://github.com/redis/redis-rb by adding it to your Gemfile of the Sinatra application and run `bundle install` by rebuilding your image (`docker build`)

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
docker network inspect {network-name}
```
, where the attached containers are listed.

<details>
  <summary>Possible solution:</summary>
  <p>

```
# In your Gemfile, add the following line
gem 'redis', '~>3.2'

# Build your image again, now tagged with a new version
# When re-building your image, you will see that the step to run `bundle install` will take longer than the other steps, 
# due to the Layered File System
docker build -t {username/image-name:new-version} .

# Run your application in background
docker run -d --rm --name {container-name-sinatra} --net {network-name} -p 4567:4567 {image-name-with-new-version}

# execute `irb` in the container you just spun up
docker exec -it {container-name-sinatra} /usr/local/bin/irb

# In your irb, require redis and perform the get/set commands as described above
```

  </p>
</details>  

#### Mini Challenge 3
Redis clients (such as the installed `redis` gem) has a function called `ping`, when the connection can be established, it return `PONG`.

Let's create a route in your sinatra-app called `/redis-status`. We will use this route to see whether your Redis instance is up.

Task: Display the result from your `ping` onto the page as plain text.

<details>
  <summary>Possible solution:</summary>
  <p>
  
```
# In docker-newbies.rb, add the following lines
require 'redis'
get '/redis-status' do
  redis = Redis.new(:host => #{name-redis-container}, :port => 6379)
  redis.ping
end

# rebuild your image again, tagged with new version
docker build -t {username/image-name:new-version} .

# Run your application with the latest version
docker run -d --rm --name {container-name-sinatra}  --net {network-name} -p 4567:4567 {image-name-with-new-version}

# Now go to /redis-status of your Sinatra application
```

  </p>
</details>

### What you have learned in this section
1. A bunch of commands for docker networks
```
docker network create {network-name}
docker network ls
docker network inspect {network-name}
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
