
###Introduction
For local development GitlabCI without many commits you need to install Docker and gitlab-ci-multi-runner locally.

Docker basically guarrantees that testing on your machine will produce identical results with a remote CI server.

Docker can ensure that the testing environment configured to our exact specifications.

Get Started:
On macOs:
Install with brew:
You can install gitlab-runner with Docker

```
$ brew install gitlab-runner
```

If you have already installed Docker - run command:

```
$ brew install gitlab-runner --without-docker
```


###Install with curl:
Go to Docker page and install the latest version

###Install gitlab-ci-multi-runner:

```
$ sudo curl --output /usr/local/bin/gitlab-ci-multi-runner https://gitlab-ci-multi-runner-downloads.s3.amazonaws.com/latest/binaries/gitlab-ci-multi-runner-darwin-amd64 
$ sudo chmod +x /usr/local/bin/gitlab-ci-multi-runner
```
Create symlink and check if gitlab-runner was successfully installed:

```
$ sudo ln -s /usr/local/bin/gitlab-ci-multi-runner /usr/local/bin/gitlab-runner 
$ which gitlab-runner /usr/local/bin/gitlab-runner
```


On Ubuntu: Install gitlab-ci-multi-runner:

```
$ curl -L https://packages.gitlab.com/install/repositories/runner/gitlab-ci-multi-runner/script.deb.sh | sudo bash 
$ sudo apt-get install gitlab-ci-multi-runner
```
Check if gitlab-runner was successfully installed:

```
$ gitlab-runner
```


###How to use
NOTE: project should be under git repository

```
$ cd path/to/project with .gitlab-ci.yml
$ ls .gitlab-ci.yml
.gitlab-ci.yml
```

If you use images from registry.zeo.lcl - need login to registry before exec CI job

```
$ docker login https://registry.zeo.lcl
```


Finally, you can exec job from .gitlab-ci.yml.

For example, 

```
$ gitlab-runner exec docker test-runner
```
where docker - executor and  test-runner - job from .gitlab-ci.yml


You can also pass custom environment variables injected to build environment:

Using cli param --env:

```
$ gitlab-runner exec docker test-runner --env='ZDT_TEST_ENV=test'
```

or

```
$ gitlab-runner exec docker test-runner --env CI_REGISTRY_USER="$CI_REGISTRY_USER" --env CI_BUILD_TOKEN="$CI_BUILD_TOKEN" --env CI_REGISTRY="$CI_REGISTRY"
```


or environment variable $RUNNER_ENV:

```
$ export RUNNER_ENV='ZDT_TEST_ENV=test' 
$ gitlab-runner exec docker test-runner
```

For verbose output, you can use next command:

```
$ gitlab-runner --debug exec docker test-runner
```


To test locally with docker inside container, for example, docker:dind, need inject /var/run/docker.sock into runner as volume

```
$ gitlab-runner exec docker api-test-runner --docker-volumes /var/run/docker.sock:/var/run/docker.sock --env CI_REGISTRY_USER="$CI_REGISTRY_USER" --env CI_BUILD_TOKEN="$CI_BUILD_TOKEN" --env CI_REGISTRY="$CI_REGISTRY"
```



###Useful gitlab-runner cli params:

```
--timeout "0" Job execution timeout (in seconds)
--docker-wait-for-services-timeout "0" How long to wait for service startup [$DOCKER_WAIT_FOR_SERVICES_TIMEOUT]
--docker-volumes Bind mount a volumes [$DOCKER_VOLUMES]
--executor Select executor, eg. shell, docker, etc. [$RUNNER_EXECUTOR]
--builds-dir Directory where builds are stored [$RUNNER_BUILDS_DIR]
--cache-dir Directory where build cache is stored [$RUNNER_CACHE_DIR]
--env Custom environment variables injected to build environment [$RUNNER_ENV]
--pre-clone-script Runner-specific command script executed before code is pulled [$RUNNER_PRE_CLONE_SCRIPT]
--pre-build-script Runner-specific command script executed after code is pulled, just before build executes [$RUNNER_PRE_BUILD_SCRIPT]
--post-build-script Runner-specific command script executed after code is pulled and just after build executes [$RUNNER_POST_BUILD_SCRIPT]
```



###Cross service communication using GitlabCI
Gitlab CI runner connect all services (mysql, redis etc) with job container using --link option (Docs https://docs.docker.com/network/links/). So, main container can connect to all services, but each service has separate network and cannot connect to other services.

Main (job) container resolve containers correctly by using name and alias in /etc/hosts file
For example:

```
172.17.0.3	mongo e5d080425c5b runner--project-0-concurrent-0-mongo-1
172.17.0.3	go-zdt-mongodb e5d080425c5b runner--project-0-concurrent-0-mongo-1
172.17.0.4	rabbitmq d920fa83b7f5 runner--project-0-concurrent-0-rabbitmq-2
172.17.0.4	go-zdt-rabbitmq d920fa83b7f5 runner--project-0-concurrent-0-rabbitmq-2
172.17.0.2	mysql 45a8d0139846 runner--project-0-concurrent-0-mysql-0
172.17.0.2	go-zdt-mysql 45a8d0139846 runner--project-0-concurrent-0-mysql-0
172.17.0.5	runner--project-0-concurrent-0
```
###Possible solution is create special network

# create network
```
docker network create gitlab-ci-network
```
and connect all services to created network

# connect a container to a network
```
docker network connect --alias go-zdt-mongodb gitlab-ci-network runner--project-0-concurrent-0-mongo-1
```

Useful command to get container ID from /etc/hosts file

```
cat /etc/hosts |grep go-zdt-mongodb|cut -d$' ' -f2
```

After job execution need disconnect all containers from network

# disconnect a container from a network
```
docker network disconnect -f gitlab-ci-network runner--project-0-concurrent-0-mongo-1
```


and remove network

# remove network
```
docker network rm gitlab-ci-network
```

More info about docker network, you can find here, here and here

Local gitlab-runner exec limitations
https://docs.gitlab.com/runner/commands/README.html#limitations-of-gitlab-runner-exec

Useful links:
https://docs.gitlab.com/runner/commands/README.html#gitlab-runner-exec

https://bryce.fisher-fleig.org/blog/faster-ci-debugging-with-gitlabci/index.html

```
GitlabCI variables https://docs.gitlab.com/ee/ci/variables/roject-0-concurrent-0-mongo-1
172.17.0.4	rabbitmq d920fa83b7f5 runner--project-0-concurrent-0-rabbitmq-2
172.17.0.4	go-zdt-rabbitmq d920fa83b7f5 runner--project-0-concurrent-0-rabbitmq-2
172.17.0.2	mysql 45a8d0139846 runner--project-0-concurrent-0-mysql-0
172.17.0.2	go-zdt-mysql 45a8d0139846 runner--project-0-concurrent-0-mysql-0
172.17.0.5	runner--project-0-concurrent-0
```
Possible solution is create special network

# create network
```
docker network create gitlab-ci-network
```
and connect all services to created network

# connect a container to a network
```
docker network connect --alias go-zdt-mongodb gitlab-ci-network runner--project-0-concurrent-0-mongo-1
```

Useful command to get container ID from /etc/hosts file

```
cat /etc/hosts |grep go-zdt-mongodb|cut -d$' ' -f2
```

After job execution need disconnect all containers from network

# disconnect a container from a network
```
docker network disconnect -f gitlab-ci-network runner--project-0-concurrent-0-mongo-1
```


and remove network

# remove network
```
docker network rm gitlab-ci-network
```

More info about docker network, you can find here, here and here

Local gitlab-runner exec limitations
https://docs.gitlab.com/runner/commands/README.html#limitations-of-gitlab-runner-exec

Useful links:
https://docs.gitlab.com/runner/commands/README.html#gitlab-runner-exec

https://bryce.fisher-fleig.org/blog/faster-ci-debugging-with-gitlabci/index.html

GitlabCI variables https://docs.gitlab.com/ee/ci/variables/
