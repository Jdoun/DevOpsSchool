# TP1

## 1-1

### DOCKERFILE

```hs
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

## 1-2

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

## 1-3

Most useful comande :
```
docker-compose up
```

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

## 1-5

```
docker tag {{nom de l'image}} {{nom du repository}}:{{numéro de version}}

docker push {{nom de l'image}}
```


## 2-1

TestContainers sont des librairies java qui permtte d'utilisé des container docker pour réaliser des tests

## 2-2

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
### Document your quality gate configuration.

![codecoverage](/images/codecoverage.png "Quality gate from sonar")

On peut voir que notre gate configuration montre que notre code est couvert à 92.1%, il à 2 vulnérabilité de sécurité et il à 3 Security Review 