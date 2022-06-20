## Installation Redash sur Ubuntu 18.04 & 20.04

### Setup Environment [^1]

This installation of Redash has the following dependencies

- Installed and running Ubuntu 20.04/18.04 LTS server
- Docker Engine
- Docker compose

The installation of Redash Data Visualization Dashboard on Ubuntu can be done from a script which automates the process for you, or manual steps.

### Step 1 : Update Ubuntu system

As a rule of thumb, your system should be updated before installing any packages.

    sudo apt update
    sudo apt upgrade -y
    sudo reboot

Once the system is rebooted, proceed to step 2

### Step 2 : Install Docker and Docker Compose

Run the following commands to install Docker on Ubuntu 20.04/18.04:

    sudo apt update
    sudo apt -yy install apt-transport-https ca-certificates curl software-properties-common wget pwgen
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
    sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
    sudo apt update && sudo apt -y install docker-ce docker-ce-cli containerd.io

Allow the current user to run Docker commands

    sudo usermod -aG docker $USER
    newgrp docker

Start the docker service and check the status of Docker engine.[^2]

    sudo systemctl start docker
    sudo systemctl status docker

We can check the version of docker engine installed.

    sudo docker --version

![Version docker chargée](https://fitdevops.in/wp-content/uploads/Screenshot-from-2020-06-02-19-37-06.png)

Lets install the docker compose on the ubuntu server.

    sudo curl -L https://github.com/docker/compose/releases/download/1.22.0/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose

Making the docker-compose binary file executable.

    sudo chmod +x /usr/local/bin/docker-compose

You can check the version of docker-compose using the below command.

    sudo docker-compose --version


### Step 3 : Prepare environment and install Redash[^1]

You can perform the installation automatically via a bash setup script or manually step-by-step.
Let’s consider both applicable methods:

Automated installation with Script

You can download and run Redash installation script without following all the steps shown in the next manual installation section.

    curl -O https://raw.githubusercontent.com/getredash/setup/master/setup.sh

Make the script executable and run it

    chmod +x setup.sh
    sudo ./setup.sh

The script will :

- Install both Docker and Docker Compose.
- Download  Docker Compose configuration files and bootstrap Redash environment
- Start all Redash docker containers

Confirm containers were created and in running status:

    docker ps

Access Redash Dashboard

Once Redash is installed, the service will be available on your server IP or DNS name assigned. Point your browser to the server address to access it.

in this case, the default localhost : http://127.0.0.1

### Step 4 : MAJ v8 vers v10[^3]

These steps are performed on the server that runs Docker.

1. Make sure to backup your data. You only need to backup Redash’s PostgreSQL database (the database Redash stores metadata in, not the ones you might be querying) as the data in Redis is transient.
> **Note :** If you just deployed a Redash V8 AMI and have not used it, you can skip this step 2
2. Open a terminal

        cd /opt/redash
4. Update `opt/redash/docker-compose.yml` to reference the docker image you want to upgrade to: `redash/redash:10.0.0.b50363`. For instance,

        sudo gedit docker-compose.yml
5. Under `services.scheduler.environment` omit `QUEUES` and `WORKERS_COUNT` and omit environment altogether if it is empty.
6. Under `services`, add a new service for general RQ jobs:

        worker:
            <<: \*redash-service
            command: worker
            environment:
                QUEUES: "periodic emails default"
                WORKERS_COUNT: 1
    ![Version docker chargée](https://user-images.githubusercontent.com/17067911/142930697-ed7d2881-ca3f-4449-a096-b906f1d983c8.png)

6. Stop Redash services:

        docker-compose stop server scheduler scheduled_worker adhoc_worker
    (you might need to list additional services if you defined them in your docker-compose.yml previously)
7. Force a recreation of your containers with

        docker-compose up --force-recreate --build
Le service va tourner en boucle en indiquant des erreurs au bout d'un moment.
Du coup il faut le stopper en faisant

        Ctrl+C
8. Run the necessary migrations with

        docker-compose run --rm server manage db upgrade
9. Restart the containers

    	docker-compose up -d

Et voilà, ça marche !!

Allez sur la console d'admnistration à l'adresse suivante : http://127.0.0.1



[^1]: https://computingforgeeks.com/install-redash-data-visualization-dashboard-on-ubuntu/

[^2]: https://fitdevops.in/how-to-setup-redash-dashboard-on-ubuntu/
[^3]: https://github.com/getredash/redash/releases/tag/v10.0.0
