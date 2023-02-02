# Author
Quentin Miguel lopez

# TP1 Discover Docker

## 1-1 Document your database container essentials: commands and Dockerfile.

### DOCKERFILE

```cli
FROM {{image}}

ENV {{VALUE_CONTAINER}}={{value}}

ADD {{script.sql}} /docker-entrypoint-initdb.d
```

### Commandes
```hs
- launch adminer :
docker run --network app-network -p 8090:8080 --name=adminer -d adminer


- launch postgres:
run --name Database --network app-network -p 5432:5432 -e POSTGRES_PASSWORD=pwd -v /database-volume:/var/lib/postgresql/data krouzet/database

- link le container a un network :
docker run --network {{nom network}}

- link un volume de donnée vers un volume du container
docker run -v {{volume-docker}}:{{volume-image}}

- Port expose vers la machine haute :
docker run -p {{exposed pot}}/{{port du container}}

- Build DOCKERFILE :
docker build -t krouzet/database .

- Create Network :
docker network create  app-network

- Create volume :
docker volume create database-volume 

```

## 1-2 Why do we need a multistage build? And explain each step of this dockerfile.

Nous avons besoin d'un multistagebuild car il faut d'abord compile le projet maven avec toute les dépendencie, puis ensuite il faut run le .jar obtenue pour lancer l'application


```Dockerfile
# Build
FROM maven:3.8.6-amazoncorretto-17 AS myapp-build #récupération de l''image java pour compile le projet avec maven
ENV MYAPP_HOME /opt/myapp #set la variable d''envirenement MYAPP_HOME 
WORKDIR $MYAPP_HOME #set l''environement de travail
COPY pom.xml .
COPY src ./src
RUN mvn package -DskipTests

# Run
FROM amazoncorretto:17 #image qui va servir à lancer le point jar
ENV MYAPP_HOME /opt/myapp #set la variable d''envirenement MYAPP_HOME 
WORKDIR $MYAPP_HOME #set l''environement de travail
COPY --from=myapp-build $MYAPP_HOME/target/*.jar $MYAPP_HOME/myapp.jar
#récupération du fichier .jar compilé
ENTRYPOINT java -jar myapp.jar #execute le fichier jar compilé
```

## 1-3 Document docker-compose most important commands.

Most useful comande :
```hs
docker-compose up #lancer les containers
docker-compose build #build tout les container
```
## 1-4 Document your docker-compose file.

```yml
version: '3.7'

services:
    backend:
        build:
          context: ./BackendAPI
          dockerfile: Dockerfile # lien vers le dockerfile du backend
        networks:
          - app-network #network a qu''elle il est lié
        depends_on:
          - database #de qu''elle container il dépends

    database:
        build:
          context: ./Database
          dockerfile: Dockerfile # lien vers le dockerfile de la database
        networks:
          - app-network #network a qu''elle il est lié
        volumes:
          - database-volume:/var/lib/postgresql/data #volume utilisé
          

    httpd:
        build:
          context: ./HTTPServer
          dockerfile: Dockerfile # lien vers le dockerfile du http
        ports:
          - '80:80' #port exposer
        networks:
          - app-network #network a qu''elle il est lié
        depends_on:
          - backend #de qu''elle container il dépends
    
networks:
    app-network:
      name: app-network #network utiliser
      external: true #définition qu''il est déja définie 

volumes:
    database-volume:
      name: database-volume #volume utilisé
      external: true #définition qu''il est déja définie 

```

## 1-5 Document your publication commands and published images in dockerhub.

```
docker tag {{nom de l'image}} {{nom du repository}}:{{numéro de version}}

docker push {{nom de l'image}}
```

# TP2 Discover Github Action

## 2-1 What are testcontainers?

TestContainers sont des librairies java qui permette d'utilisé des container docker pour réaliser des tests

## 2-2 Document your Github Actions configurations.

```yml
name: CI devops 2023
on:
  #to begin you want to launch this job in main and develop
  push:
    branches: 
      - main # Activation des jobs lors du push sur la branche main
  pull_request:

jobs:
  test-backend: 
    runs-on: ubuntu-22.04 #Utilisation de l'image Ubuntu
    steps:
     #checkout your github code using actions/checkout@v2.5.0
      - uses: actions/checkout@v2.5.0 

     #do the same with another action (actions/setup-java@v3) that enable to setup jdk 17
      - name: Set up JDK 17
        uses: actions/setup-java@v3
        with: #use de l'action au dessus et définition de la version de java
          java-version: '17'
          distribution: 'adopt'

     #finally build your app with the latest command
      - name: Build and test with Maven
        run: cd ./BackendAPI/simple-api-student-main/ && mvn clean verify #déplacement dans le bon dossier et on éxécute la commande mvn clean verify
```
### 2-3 Document your quality gate configuration.

![codecoverage](/images/codecoverage.png "Quality gate from sonar")

On peut voir que notre gate configuration montre que notre code est couvert à 92.1% ce qui veux dire que notre code passerais en prod et il serait valide, il à 2 vulnérabilité de sécurité qui nous indique donc qu'il faudrait que nous modifions ces points et il à 3 Security Review. ET il y 2 code alerte sur du code qui ne va pas être maintenue. 

# TP3 Discover Ansible

### 3-1 Document your inventory and base commands

```YML
all:
 vars:
   ansible_user: centos #os de la machine
   ansible_ssh_private_key_file: ./id_rsa #chemin de la clé rsa
 children:
   prod:
     hosts: quentin.miguel-lopez.takima.cloud # différents host impacté par le fichier yml
```


```
ansible all -i inventories/setup.yml -m setup -a "filter=ansible_distribution*"

quentin.miguel-lopez.takima.cloud | SUCCESS => {
    "ansible_facts": {
        "ansible_distribution": "CentOS",
        "ansible_distribution_file_parsed": true,
        "ansible_distribution_file_path": "/etc/centos-release",
        "ansible_distribution_file_variety": "CentOS",
        "ansible_distribution_major_version": "8",
        "ansible_distribution_release": "Stream",
        "ansible_distribution_version": "8",
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": false
}

ansible all -i inventories/setup.yml -m yum -a "name=httpd state=absent" --become

quentin.miguel-lopez.takima.cloud | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": true,
    "msg": "",
    "rc": 0,
    "results": [
        "Removed: mod_http2-1.15.7-5.module_el8.6.0+1111+ce6f4ceb.x86_64",
        "Removed: httpd-2.4.37-47.module_el8.6.0+1111+ce6f4ceb.1.x86_64"
    ]
}

```

### 3-2 Document your playbook

```YML
- hosts: all
  gather_facts: false
  become: yes
  roles:
    - docker

# Install Docker
  tasks:
  - name: Clean packages
    command:
      cmd: dnf clean -y packages

  - name: Install device-mapper-persistent-data
    dnf:
      name: device-mapper-persistent-data
      state: latest

  - name: Install lvm2
    dnf:
      name: lvm2
      state: latest

  - name: add repo docker
    command:
      cmd: sudo dnf config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo

  - name: Install Docker
    dnf:
      name: docker-ce
      state: present

  - name: install python3
    dnf:
      name: python3

  - name: Pip install
    pip:
      name: docker

  - name: Make sure Docker is running
    service: name=docker state=started
    tags: docker

```

Push sur le serveur réaliser mais le serveur ne fonctionnait pas lors du push à cause du loadbalancer qi prendrait trop de ram

### 3-3 Document your docker_container tasks configuration.

```yml
  - name: Run HTTPD #nom de la tache qui va séxecuter
    docker_container:
      name: httpd #nom du container sur docker
      image: krouzet/httpserver #image à allez cherche rsur dockerhub
      network_mode: app-network #network à lié au container
      ports: #ports que l'on souhaite exposer ou non
        - '80:80'
        - '8080:8080'
```
## Front
![acceuil](/images/acceuil.png "acceuil")

![pagedepartement](/images/pagedepartement.png "page departement")

# TP Extra

Load balancing gérer avec les deux serveurs

![containers](/images/doublecontainer.png "containers docker")

pour activer il suffisait de changer le virtualhost est d'activer certain module
```
<VirtualHost *:8080>
<Proxy "balancer://mycluster"> # définir les deux chemin de serveur
    BalancerMember "http://backend-green:8080"
    BalancerMember "http://backend-blue:8080"
</Proxy>
ProxyPass        "/" "balancer://mycluster/"
ProxyPassReverse "/" "balancer://mycluster/"
</VirtualHost>

<VirtualHost *:80>
ProxyPreserveHost On
ProxyPass / http://front:80/
ProxyPassReverse / http://front:80/
</VirtualHost>
```