In today story, I’d like to demonstrate a simple way to dockerize Python program written with CLI tools (like argparse library, Click, etc.). Although there are several ways to run Python script using Docker, this solution is expected to achieve two goals:

1. I want to run the script similarly as I would in the terminal on the host, using only script name and command-line arguments.
2. Docker container should only be run once and allows the script to be used as many times as necessary.

In the following sections, I will show how to prepare Dockerfile, build and run Docker image, take care of generated data, and finally, use Docker Compose to make things even simpler.

So, let’s get started…

**Creating a Dockerfile**

First, we need to prepare a Dockerfile file in the root directory of our Python application. Let’s look at its instructions one by one.

We start by declaring a Python base image, its version and image variant. In our example, we will use Python 3.8 and _slim_ variant to make the size of our final image smaller.

FROM python:3.8-slim

Then we add a new user, named arbitrarily ‘app_user’. Due to security reasons, it’s a good practice to run containers as a non-root user. Option  
`--create-home` will create user’s home directory and `--shell bin/bash` make sure bash is the default shell.

RUN useradd --create-home --shell /bin/bash app_user

The next step is to change the working directory. We will use `/home/app_user` as the root directory for the project in the container.

WORKDIR /home/app_user

Then we copy the requirements.txt file.

COPY requirements.txt ./

And install all required packages. Disable the cash using `--no-cache-dir` option to lower image size.

RUN pip install --no-cache-dir -r requirements.txt

After the installation of all necessary components, we change the user to the previously created “app_user”.

USER app_user

And copy application source code from Dockerfile directory in the host to the `/home/app_user` directory in the container.

COPY . .

Finally, we set bash as default command, which will be invoked when docker container runs.

CMD ["bash"]

As you will see in the next paragraph, we will run this Docker image in interactive mode with pseudo-tty attached. The latter is actually required by bash, without it the shell process would exit immediately and the container would exit as well. This setup will allow us to run our script directly from the container and also will keep our container running as long as the bash is active.

Here is the complete Dockerfile:

FROM python:3.8-slimRUN useradd --create-home --shell /bin/bash app_userWORKDIR /home/app_userCOPY requirements.txt ./RUN pip install --no-cache-dir -r requirements.txtUSER app_userCOPY . .CMD ["bash"]

**Building and running the Docker image**

To build the image run the following command from the Dockerfile directory. You might add `-t` option to name your image and `--rm` to remove intermediate containers after the build is done.

```
docker build -t my_image --rm .
```

Now we can run our image, adding `-i` and `-t` options to make it interactive as mentioned before. In our example, we will name the container ‘my_app’. Adding `--rm` option will remove the container automatically after the container exits.

docker run -it --name my_app --rm my_image

At this point, we can start using our Python script with the same commands we would use on the host, e.g.:

app_user@6a8e836b3f0c:~$ python my_script.py argument1 argument2

This container will be active as long as we don’t exit our pseudo-tty, either by `exit` command or `ctrl+d` shortcut. To go back to the terminal on the host but keep docker container running, use the sequence of `ctrl+p ctrl+q`. You can return to the container by `docker attach my_app` command.

There is also an alternative way of running and using the container, which can be useful in some cases. By adding `-d` option to docker run command, you will start in the detached mode, like this:

```shell
docker run -dit --name my_app --rm my_image
```

Then use Docker exec command, to attach additional bash to your container and run Python script from there.

```
docker exec -it my_app bash
```

Basically, you can have more than one bash session simultaneously. The difference between them is that you can exit this additional shell any time and the container will still be running since the main process in it (the bash run by Dockerfile CMD instruction) is still active. If you wish to remove this container use `docker stop my_app` command or just attach to it and use `exit` from there.

**Saving container data**

In the case our script is creating data and save them is some files, we need to preserve them after container ceases to exist. There is more than one way to achieve that, but in our example, we will take advantage of [bind mounting](https://docs.docker.com/storage/bind-mounts/).

Let’s assume that our application saves all files inside `/data` directory. We add `--mount type=bind,source=”$(pwd)”/data,target=/home/app/data`to the docker run command, which specifies the type of mount (bind), the directory in our host where the files are going to be saved (`source=”$(pwd)”/data`) and the directory in the container (`target=/home/app/data`). Here is the complete docker run command:

```shell
docker run -it --name my_app --rm --mount type=bind,source="$(pwd)"/,target=/home/app my_image
```

From now on, every file created or edited in `/home/app/data` directory in the container will also be saved in the selected directory on the host.

If your program is using SQLite database, you can follow the same steps. Just create a bind mount to the directory where .db file is located (preferably other than your project main directory). Keep in mind that you might have to install SQLite in the container unless you are using some ORM that handles the connection to your database. To do that add an additional instruction to your Dockerfile (the first line is new):

...RUN apt-get -y update && apt-get install -y sqlite3RUN pip install --no-cache-dir -r requirements.txt...

As our docker run command starts to get longer and longer, it becomes inconvenient to type it every time manually. And even if we don’t use any type of mounting, it is still a good idea to configure everything once and be able to run the application with a simple command. In other words, it’s time for Docker Compose.

**Docker Compose to make things easier**

We will start by creating a new file called docker-compose.yml in the root directory of our application, the same where Dockerfile is located. In this file, we will put the following lines:

```Dockerfile
version: "3.8"  
services:  
  app:  
    build: .  
    image: my_image  
    volumes:  
      - ./data:/home/app_user/data  
    stdin_open: true  
    tty: true
```

Firstly, we define the version of our docker-compose file. Then we declare all services required by our application. Since we only need to run one container, there is only one service that we will arbitrarily call “app”. Now we specify all information that Docker Compose will use to run this container for us, which are as follows: directory of the Dockerfile, name of docker image that will be built, volume to be created, keeping STDIN open (docker run `-i`option) and attaching pseudo-tty (docker run `-t`option). Based on this configuration, Docker Compose will create a container that will be almost equivalent to the one that would be created with the last docker run command (in Saving container data). The only difference is the lack of `--rm` option.

At this point, the only thing we need to do is to run the following command:

```shell
docker-compose run --rm app
```

It will build the docker image, run it and also attach us to the container associated with the service name ‘app’. The `--rm` option works the same as for the docker run command. Now we can use our script identically as described before.

If you like the idea of attaching additional bash, use these two commands instead:

```shell
docker-compose up -ddocker exec -it <container-name-or-id> bash
```

When using `docker-compose up` the name of the container is generated automatically. Check container name or id by using `docker ps`. It is possible though to set the [custom container name](https://docs.docker.com/compose/compose-file/#container_name).

When done, use `docker-compose down` command to stop and remove the container.

**Summary**

Although this solution was originally intended for Python CLIs, a similar approach can be applied in other situations as well. Basically, each time when you want to run a single script many times or run multiple scripts from a single docker container. This way, once you have Dockerfile and docker-compose.yml prepared, you can set up your program quickly and use it directly.