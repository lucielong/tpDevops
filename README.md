# tpDevops

### 1-1 Document your database container essentials: commands and Dockerfile.

Première étape : créer notre Docker file :

```docker
FROM postgres:14.1-alpine *Image de base sur laquelle on se base*

ENV POSTGRES_DB=db \
   POSTGRES_USER=usr \
   POSTGRES_PASSWORD=pwd

*On défini notre environnement de database avec le user, le password et le nom de la database.*
```

On build l’image : 

```bash
docker build -t llong1/databasetp .
```

On crée un network pour que notre database puisse communiquer avec adminer :

```bash
docker network create app-network
```

On run adminer :

```bash
docker run \
    -p "8090:8080" \
    --net=app-network \
    --name=adminer \
    -d \
    adminer
```

Puis on run notre image :

```bash
docker run -p 8888:5000 --name databasetp  --network app-network llong1/databasetp
```

Pour ajouter nos tables et nos données sql, on copie nos fichier sql sur notre container : 

```docker
FROM postgres:14.1-alpine

COPY /initdb /docker-entrypoint-initdb.d

ENV POSTGRES_DB=db \
   POSTGRES_USER=usr \
   POSTGRES_PASSWORD=pwd
```

On re build notre image puis on la re run 

Enfin, pour conserver nos données, on ajoute un volume :

```bash
docker run -p 8888:5000 --name databasetp  --network app-network -v /initdb:/var/lib/postgresql/data llong1/databasetp
```

### 1-2 Why do we need a multistage build? And explain each step of this dockerfile.

```bash
# Build
FROM maven:3.8.6-amazoncorretto-17 AS myapp-build
ENV MYAPP_HOME /opt/myapp
WORKDIR $MYAPP_HOME
COPY pom.xml .
COPY src ./src
RUN mvn package -DskipTests

# Run
FROM amazoncorretto:17
ENV MYAPP_HOME /opt/myapp
WORKDIR $MYAPP_HOME
COPY --from=myapp-build $MYAPP_HOME/target/*.jar $MYAPP_HOME/myapp.jar

ENTRYPOINT java -jar myapp.jar

```

### 2-1 What are testcontainers?

They simply are java libraries that allow you to run a bunch of docker containers while testing.

### 2-2 Document your Github Actions configurations.

```yaml
name: CI devops 2023
on:
  #to begin you want to launch this job in main and develop
  push:
    branches: 
      - main
  pull_request:

jobs:
  test-backend: 
    runs-on: ubuntu-22.04
    steps:
     #checkout your github code using actions/checkout@v2.5.0
      - uses: actions/checkout@v2.5.0

     #do the same with another action (actions/setup-java@v3) that enable to setup jdk 17
      - name: Set up JDK 17
        uses: actions/setup-java@v3
        with:
          java-version: '17'
          distribution: 'adopt'

     #finally build your app with the latest command
      - name: Build and test with Maven
        working-directory: simple-api
        run: mvn clean verify
```

### Document your quality gate configuration.

### 3-1 Document your inventory and base commands

### 3-2 Document your playbook

### Document your docker_container tasks configuration.
