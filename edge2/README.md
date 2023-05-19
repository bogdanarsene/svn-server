https://hub.docker.com/r/svnedge/app

SVN Edge Docker Image
This is a ready to run docker image for SVN Edge. The base image is centos:7. You can run this image as is or use it as a base image and update/install additional CentOS packages, such as those needed for your hook scripts.

The image exposes the Apache Subversion server on port 18080 and the SVN Edge console on port 3343. The assumption is that these ports will be proxied by some other layer where you perform SSL termination. You must supply a persistent volume to permanently store the data, which would include your Subversion repositories as well as the database for SVN Edge.

Running the App
A simple docker-compose for this image would look like this:

version: "3.7"

services:

  edge:
    image: svnedge/app:latest
    ports:
      - "18080:18080"
      - "3343:3343"
    environment:
      SET_PERMS: "false"
    volumes:
      - data:/home/svnedge/csvn/data

volumes:
  data:
You would then run the application with docker-compose up and login at http://localhost:3343

It is worth noting there are some features of the SVN Edge web console that do not make sense in Docker. We are trusting that you will just avoid those features. For example, do not change the path where repositories are stored unless you are sure you have provided a data volume for that path. Likewise, do not try to change the port number for the Apache server unless you are going to make your own Dockerfile that exposes that port. You CAN configure Apache to use SSL if you want, on port 18080, but as noted earlier, our assumption is that your ingress traffic will hit some kind of load balancer or proxy that terminates the SSL connection and proxies the traffic to port 18080 or 3343 depending on the context.

If you want to just run the image with a single command without using docker-compose you would want something like this:

docker run -d --name=svnedge --mount type=bind,src=`pwd`/data,dst=/home/svnedge/csvn/data -p 3343:3343 -p 18080:18080 svnedge/app:latest
This would mount a folder named "data" in the current working directory as the volume. Adjust accordingly.

Upgrading to a new version
Upgrading to a new version of SVN Edge should be as simple as stopping the container, pulling the new image and starting it back up with that image. Assuming you point it at the same data volume, SVN Edge will handle the upgrade process for the data.

If you run into OS permissions problems where the SVN Edge container cannot access your data, try starting the container with the envvar SET_PERMS=true and the container will attempt to change the owner of all of the data on the volume before it starts the SVN Edge console. You should only need to do this one time and you should unset that envvar or set it to false for subsequent startups so that it does not waste processing time changing the owner again.

Migrating to Docker
It should also be possible to migrate from a bare metal install of SVN Edge to the Docker version by mounting the data folder of the old install when running the container. There are only a few considerations to factor in:

You must delete the data/conf/httpd.conf file before starting the container. This file will be recreated when the image starts up.
There are a few areas in the database where the path of your install is stored and those have to be fixed for the new path within the container. For example, the SVN Edge admin UI has the path to the repositories and backups. Those need to be changed to /home/svnedge/csvn/data/repositories and /home/svnedge/csvn/data/dumps. You can do this via the SVN Edge admin UI after starting the app in the console.
You will need to change the Apache server to listen on port 18080, you can do this via the SVN Edge admin UI.
You will need to adjust the server name to match the hostname your users will use to reach the server.
If you were using hook scripts, there is a good chance that you will need to something to adjust them. The scripts will be running inside the docker container which means you may need to build your own container that installs needed dependencies or you may need to adjust paths within the script.


https://manualzz.com/doc/30684291/collabnet-subversion-edge-user-guide

http://localhost:3343/csvn/
user: admin
password: admin
