## ssh-service-ubuntu-docker-image

### Build an `ubuntu_sshd` image
The following `Dockerfile` sets up an SSHd service in a container that you can use to connect to and inspect other container’s volumes, or to get quick access to a test container.

```ssh
FROM ubuntu:16.04

RUN apt-get update && apt-get install -y openssh-server
RUN mkdir /var/run/sshd
RUN echo 'root:screencast' | chpasswd
RUN sed -i 's/PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config

# SSH login fix. Otherwise user is kicked off after login
RUN sed 's@session\s*required\s*pam_loginuid.so@session optional pam_loginuid.so@g' -i /etc/pam.d/sshd

ENV NOTVISIBLE "in users profile"
RUN echo "export VISIBLE=now" >> /etc/profile

EXPOSE 22
CMD ["/usr/sbin/sshd", "-D"]
```
Build the image using:

```ssh
$ docker build -t ubuntu_sshd .
```

### Run a test_sshd container

Then run it. You can then use `docker port` to find out what host port the container’s port 22 is mapped to:

```ssh
$ docker run -d -P --name test_sshd ubuntu_sshd
$ docker port test_sshd 22

0.0.0.0:49154
```

And now you can ssh as root on the container’s IP address (you can find it with docker inspect) or on port 49154 of the Docker daemon’s host IP address (ip address or ifconfig can tell you that) or localhost if on the Docker daemon host:

```ssh
$ ssh root@192.168.1.2 -p 49154
# The password is ``screencast``.
root@f38c87f2a42d:/#
```
### Environment variables
Using the `sshd` daemon to spawn shells makes it complicated to pass environment variables to the user’s shell via the normal Docker mechanisms, as `sshd` scrubs the environment before it starts the shell.

If you’re setting values in the `Dockerfile` using `ENV`, you need to push them to a shell initialization file like the `/etc/profile` example in the `Dockerfile` above.

If you need to pass `docker run -e ENV=value` values, you need to write a short script to do the same before you start `sshd -D` and then replace the `CMD` with that script.

### Clean up
Finally, clean up after your test by stopping and removing the container, and then removing the image.

```ssh
$ docker container stop test_sshd
$ docker container rm test_sshd
$ docker image rm ubuntu_sshd
```

