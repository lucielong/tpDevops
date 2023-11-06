# tpDevops

### 1-1 Document your database container essentials: commands and Dockerfile.

First step: create of Docker file:

```docker
FROM postgres:14.1-alpine *The base image upon which we build*

ENV POSTGRES_DB=db \
   POSTGRES_USER=usr \
   POSTGRES_PASSWORD=pwd

*We define our database environment with the username, password, and the name of the database*
```

We build the image: 

```bash
docker build -t llong1/databasetp .
```

A network is created to allow our database to communicate with Adminer:

```bash
docker network create app-network
```

We run adminer:

```bash
docker run \
    -p "8090:8080" \
    --net=app-network \
    --name=adminer \
    -d \
    adminer
```

Then we run our image:

```bash
docker run -p 8888:5000 --name databasetp  --network app-network llong1/databasetp
```

To add our SQL tables and data, we copy our SQL files to our container:
```docker
FROM postgres:14.1-alpine

COPY /initdb /docker-entrypoint-initdb.d

ENV POSTGRES_DB=db \
   POSTGRES_USER=usr \
   POSTGRES_PASSWORD=pwd
```

We rebuild our image and then rerun it. 

Finally, to preserve our data, we add a volume:

```bash
docker run -p 8888:5000 --name databasetp  --network app-network -v /initdb:/var/lib/postgresql/data llong1/databasetp
```

### 1-2 Why do we need a multistage build? And explain each step of this dockerfile.

We need a multistage build in order to optimize Docker images by separating the build environment from
the final production image. It reduces the size of the final image and eliminates unnecessary build
dependencies.

```bash

# Build

# Use the maven:3.8.6-amazoncorretto-17 image as the build stage and name it myapp-build.
FROM maven:3.8.6-amazoncorretto-17 AS myapp-build

# Set an environment variable MYAPP_HOME to /opt/myapp.
ENV MYAPP_HOME /opt/myapp

# Set the working directory in the container to MYAPP_HOME.
WORKDIR $MYAPP_HOME

# Copy the Maven Project Object Model (POM) file (pom.xml) from the host to the container.
COPY pom.xml .

# Copy the source code from the host to the container in the src directory.
COPY src ./src

# Run the Maven build to package the application, skipping the tests.
RUN mvn package -DskipTests

# Run

# Switch to a new stage using the amazoncorretto:17 base image.
FROM amazoncorretto:17

# Set the same MYAPP_HOME environment variable as in the build stage.
ENV MYAPP_HOME /opt/myapp

# Set the working directory in the container to MYAPP_HOME.
WORKDIR $MYAPP_HOME

# Copy the JAR file(s) from the myapp-build stage to the MYAPP_HOME directory in this stage.
COPY --from=myapp-build $MYAPP_HOME/target/*.jar $MYAPP_HOME/myapp.jar

# Define the entry point for the container, which will run the Java application with the JAR file.
ENTRYPOINT java -jar myapp.jar


```

### 2-1 What are testcontainers?

Testcontainers are Java libraries that allow running multiple Docker containers during testing.

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
