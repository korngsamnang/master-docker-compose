# Node.js App with Docker Compose

## Overview

This project demonstrates how to run a Node.js application with MongoDB using Docker Compose. It includes Docker setup for a Node.js app, MongoDB, and Mongo Express (a web interface for MongoDB). The app is containerized using Docker, allowing you to easily manage and deploy the app in different environments.

## Pre-Requisites to Learn Docker Compose

Before diving into Docker Compose, it is important to have a basic understanding of the following concepts:

-   **Docker**: Basic knowledge of how Docker works, including images, containers, and Dockerfile.
-   **Containers and Images**: Understanding the difference between a Docker container and a Docker image.
-   **Basic Command Line**: Comfort with using the command line, especially for Docker commands.
-   **YAML syntax**: Familiarity with the YAML format used for Docker Compose configuration.

## What is Docker Compose

Docker Compose is a tool for defining and running multi-container Docker applications. With Compose, you define a multi-container setup using a single `docker-compose.yml` file. You can then start all the containers with a single command, making it easier to manage and deploy applications that require multiple services.

## Without Docker Compose

Without Docker Compose, you would need to run each service (e.g., your Node.js app, MongoDB, and Mongo Express) in separate Docker containers and manually configure their connections. This would require multiple `docker run` commands and complex network configurations.

## Why Docker Compose

Docker Compose simplifies the management of multi-container applications. Some key benefits include:

-   **Simplified Configuration**: Define all your services, networks, and volumes in a single file.
-   **Single Command for Multiple Services**: Start, stop, or manage all services with one command.
-   **Consistent Environment**: Easily recreate the same environment across multiple machines, ensuring consistency.
-   **Dependency Management**: Control the startup order of services and manage environment variables efficiently.

## From Docker Commands to Compose File

You can convert individual Docker commands into a Compose file. For example:

-   Docker Command: `docker run -p 3000:3000 --name my-app my-app:1.0`
-   Docker Compose Equivalent:

    ```yaml
    services:
        my-app:
            image: my-app:1.0
            ports:
                - "3000:3000"
    ```

Similarly, you can convert other Docker commands into Compose syntax, such as port mappings, environment variables, and service dependencies.

## Create Compose File and Start Application

To create the Docker Compose file (docker-compose.yml), follow the structure of the mongo-services.yaml file provided. Here's how to start the application using Docker Compose:

1. Ensure you have Docker and Docker Compose installed.
2. Navigate to the directory containing your project files.
3. Create a `docker-compose.yml` file with the following content:

    ```yaml
    version: "3.1"
    services:
        my-app:
            build: . #build the image from the Dockerfile in the current directory
            ports: - 3000:3000 #host:container
            environment:
                MONGO_DB_USERNAME: ${MONGO_DB_USERNAME}
                MONGO_DB_PWD: ${MONGO_DB_PWD}

        mongodb:
            image: mongo
            ports:
                - 27017:27017
            environment:
                MONGO_INITDB_ROOT_USERNAME: ${MONGO_DB_USERNAME}
                MONGO_INITDB_ROOT_PASSWORD: ${MONGO_DB_PWD}

        mongo-express:
            image: mongo-express
            ports:
                - 8081:8081
            environment:
                ME_CONFIG_MONGODB_ADMINUSERNAME: ${MONGO_DB_USERNAME}
                ME_CONFIG_MONGODB_ADMINPASSWORD: ${MONGO_DB_PWD}
                ME_CONFIG_MONGODB_SERVER: mongodb
            depends_on:
                - mongodb
    ```

4. Run the following command to start the services:

    ```bash
    docker-compose up
    ```

## Control Startup Order

Docker Compose allows you to control the startup order of services using the depends_on key. In the example above, mongo-express will not start until the mongodb service is up and running.

## Docker Compose Commands (Up and Down vs Start and Stop)

-   `docker-compose up`: Starts all services defined in the docker-compose.yml file. If the images are not present, it will build them.
-   `docker-compose down`: Stops and removes all containers, networks, and volumes associated with the services.
-   `docker-compose start`: Starts the services that were previously stopped.
-   `docker-compose stop`: Stops the services without removing them.

## Connect Own Web Application

To connect your own web application to the Docker Compose setup:

1. Make sure your web application is configured to communicate with the MongoDB service using the MongoDB credentials provided.
2. In your `Dockerfile` for the web application, make sure you expose the correct port and connect to the database using the environment variables defined in `docker-compose.yml`.

## Variables in Docker Compose

Docker Compose allows you to use environment variables in the docker-compose.yml file. You can define environment variables in a `.env` file or directly in the Compose file. For example:

```yaml
environment:
    MONGO_DB_USERNAME: ${MONGO_DB_USERNAME}
    MONGO_DB_PWD: ${MONGO_DB_PWD}
```

This allows you to easily configure different environments (development, staging, production).

## Docker Compose Secrets

Docker Compose supports secrets management, allowing you to securely store and manage sensitive data such as database credentials. Secrets are defined in a separate secrets file and can be used by services. For example:

```yaml
secrets:
    mongo_password:
        file: ./secrets/mongo_password.txt
```

To use the secret in a service:

```yaml
services:
    mongodb:
        image: mongo
        secrets:
            - mongo_password
```

## Use Image from Private Repository

If you need to pull an image from a private Docker repository, use the image directive in the Compose file:

```yaml
services:
    my-app:
        image: your-private-repo/my-app:1.0
        environment:
            - DOCKER_REGISTRY_USERNAME=${DOCKER_USERNAME}
            - DOCKER_REGISTRY_PASSWORD=${DOCKER_PASSWORD}
```

To log in to the private repository, use the `docker login` command before running `docker-compose up`.

## Limitations: Docker Compose vs Kubernetes

While Docker Compose is great for managing multi-container applications on a single host, Kubernetes is designed for managing containerized applications in a distributed environment. Some key differences:

-   Scalability: Kubernetes is designed for scaling applications across multiple nodes, while Docker Compose is more suited for development or small-scale environments.
-   Complexity: Kubernetes has a steeper learning curve but offers more powerful features like auto-scaling, load balancing, and persistent storage management.
-   Networking: Docker Compose uses Docker's default network, while Kubernetes has advanced networking capabilities, such as service discovery and load balancing.

For simple applications or development environments, Docker Compose is an excellent tool, while Kubernetes is ideal for large-scale, production environments with complex scaling and management needs.
