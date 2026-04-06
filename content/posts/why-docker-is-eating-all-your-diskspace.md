---
title: 'Why Docker is eating your disk space'
draft: false
date: '2021-09-23T09:33:01+00:00'
author: juhana.jauhiainen
tags: ["docker", "logging", "linux"]
description: "Have you ever had your server disk space exhausted by Docker? Is it happening right now? What can we do?!"
---
I learned recently, in less than ideal circumstances, that Docker doesn't do **any** log rotation by default 😱

So if you're running your application in production with Docker and using Docker's default [logging driver](https://docs.docker.com/config/containers/logging/configure/) you might be awakened some night at 3 AM to fix a service outage when you can barely think straight. 😅

The default logging driver is `json-file` which caches the container logs as JSON. In practice, this means Docker is writing all the container `stdout` and `stderr` to a JSON file located in the host systems `/var/lib/docker/containers` folder.

You can find out the locations and sizes of these log files with `docker inspect`. You might need to run this with `sudo`.

```shell
du -h $(docker inspect --format='{{.LogPath}}' $(docker ps -qa))
```

If you need to delete logs of a container before continuing, you can use `echo` to overwrite them as empty. This might require stopping the containers first.

```shell
echo "" > $(docker inspect --format='{{.LogPath}}' container_name)
```

### Changing the default logging driver

You can change the default logging driver by adding a configuration for it in `/etc/docker/daemon.json`. Docker supports multiple logging drivers but the simplest solution to our disk gorging logs problem is to use the `local` driver. The official Docker documentation explicitly [refers to using it](https://docs.docker.com/config/containers/logging/configure/) to prevent disk-exhaustion.

Let's create a basic config for `local` logging driver.

```json
{
  "log-driver": "local",
  "log-opts": {
    "max-size": "100m",
    "max-file": "3"
  }
}
```

We are defining the logging driver with `log-driver` and setting the maximum size of a log file before it is __rolled__. This means, when the file reaches 100 megabytes, a new file is created and the old one is archived. `max-file` is here set to "3", so at any point in time there will be only three log files stored. When the third file reaches 100 megabytes, a new file is created and the oldest log file is deleted.


Next, we need to reload the config for the Docker daemon. The Docker documentation suggests we can use `SIGHUP` [signal](https://www.man7.org/linux/man-pages/man7/signal.7.html) to reload the configuration. Unfortunately, this [doesn't work on all configuration options](https://stackoverflow.com/a/51206053/11849122) with logging driver being one of them.

This means we will have to restart the Docker daemon to reload the config. The command for restarting the daemon depends on the system it's running on. On Linux systems using [systemd] docker daemon can be restarted using the `service` command.

```shell
service docker restart
```

Now if we check the logging driver currently in use with [info](https://docs.docker.com/engine/reference/commandline/info/), it should print `local`

```shell
docker info --format '{{.LoggingDriver}}'
```

Now we have changed the default logging driver but if we have **running containers** this is **not** enough. The configuration change will only affect **new** containers. This means we will have to recreate all our existing containers. Depending on your setup you can do this using `docker-compose` or by running `docker build`

```shell
docker-compose up -d --force-recreate
```

<div className="bg-yellow-200 text-black font-thin p-6 rounded-md flex gap-4 shadow-md mt-4">
<div>👉</div>
<div>
    If you're using docker-compose and get an warning <span className="font-bold">WARNING: no logs are available with the 'local' log driver</span> when you're trying to read logs from a container, it means you're running an older version of docker-compose which doesn't support <span className="font-bold">local</span> logging driver. Try <a href="https://docs.docker.com/compose/install/">upgrading</a> your docker-compose
</div>
</div>


### Further reading

[Docker logging drivers](https://docs.docker.com/config/containers/logging/configure/)  
[Configuring local driver](https://docs.docker.com/config/containers/logging/local/)
