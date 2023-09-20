---
title: Setting up a Docker Host
date: 2023-09-17 12:00:00 -0700
categories:
  - homelab
tags:
  - docker
  - servers
---
first make sure you have your linux distro of choice setup either as a VM or Bare Metal. For this tutorial i will be using ubuntu 22.04 LTS now lets get started


# Installing Docker and Docker-Compose
---
Docker is a tool for containerization that simplifies the packaging and distribution of applications eliminating the old meme "It works on my machine.". In this guide, we'll explain how to install Docker and Docker-Compose on Ubuntu 22.04 using the package manager. Additionally, we'll provide examples of starting a Docker container using both the ***docker*** and ***docker-compose*** commands.

**Step 1: Update APT Repository** Before installing Docker, it's a good practice to ensure that your APT package repository is up to date. Open a terminal and run the following commands:

```bash
sudo apt update
```

**Step 2: Install Docker.io and Docker-Compose** To install Docker, you can use the APT package manager. Run the following command to install Docker.io and Docker-compose:

```bash
sudo apt install docker.io docker-compose
```

**Step 3: Verify Docker Installation** To confirm that Docker is installed correctly, run the following command:

```bash
docker --version
```

You should see the version of Docker displayed on your terminal.

**Step 4: Create a Docker Container Using `docker` Command** Now that Docker is installed, let's create a simple Docker container using the `docker` command. In this example, we will run a basic Nginx web server container:

```bash
docker run -d -p 80:80 --name my-nginx nginx
```

- `-d` stands for "detached" mode, which runs the container in the background.
- `-p 80:80` maps port 80 of your host to port 80 of the container.
- `--name my-nginx` assigns the name "my-nginx" to the container.
- `nginx` specifies the Docker image to use.

You can access the Nginx web server by opening a web browser and navigating to your server's IP address.

**Step 5: Create a Docker Container Using `docker-compose`** Docker-Compose simplifies the management of multi-container applications. Here's an example of a docker-compose.yml file that defines a simple Nginx and MySQL stack:

```yaml
version: '3'
services:
  web:
    image: nginx
    ports:
      - "80:80"
  db:
    image: mysql
    environment:
      MYSQL_ROOT_PASSWORD: examplepassword
```


Save this file and then run the following command to start the defined services:

```bash
docker-compose up -d
```

This will start both the Nginx and MySQL containers as specified in the docker-compose.yml file.

Now you know how to install and run a container! We've covered the installation of Docker and Docker-Compose on Ubuntu 22.04 using APT. Additionally, we created Docker containers using both the docker and docker-compose commands. Go forth, create and deploy whatever comes to mind! Thank you for reading! Hope to see you again