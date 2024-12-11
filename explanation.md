## 1. Choice of Base Image
 The base image used to build the containers is `node:16-alpine3.16`. It is derived from the Alpine Linux distribution, making it lightweight and compact. 
 Used 
 1. Client:`node:16-alpine3.16`
 2. Backend: `node:16-alpine3.16`
 3.Mongo : `mongo:6.0 `
       

## 2. Dockerfile directives used in the creation and running of each container.
 I used two Dockerfiles. One for the Client and the other one for the Backend.

**Client Dockerfile**

```
# Use an official Node runtime as a parent image
FROM node:14-slim AS build

# Set the working directory in the container
WORKDIR /usr/src/app

# Copy the package.json and package-lock.json files to the container
COPY package*.json ./

# Install application dependencies
RUN npm install

# Copy the rest of the application code to the container
COPY . .

FROM alpine:3.16.7

WORKDIR /app

RUN apk update && apk add npm

COPY --from=build /usr/src/app /app

# Expose the port the app runs on
EXPOSE 3000

# Define the command to run your app
CMD ["npm", "start"]
```
**Backend Dockerfile**

```
# Use an official Node runtime as a parent image
FROM node:14 AS build

# Set the working directory in the container
WORKDIR /usr/src/app

# Copy the package.json and package-lock.json files to the container
COPY package*.json ./

# Install application dependencies
RUN npm install

# Copy the rest of the application code to the container
COPY . .

#Add multi-stage build
FROM alpine:3.16.7

WORKDIR /app

RUN apk update && apk add --update nodejs

COPY --from=build /usr/src/app /app

# Expose the port the app runs on
EXPOSE 5000

# Define the command to run your app
CMD ["node", "server.js"]

```

## 3. Docker Compose Networking
The (docker-compose.yml) defines the networking configuration for the project. It includes the allocation of application ports. The relevant sections are as follows:


```
services:
  backend:
    # ...
    ports:
      - "5000:5000"
    networks:
      - app-net

  client:
    # ...
    ports:
      - "3000:3000"
    networks:
      - app-net
  
  mongodb:
    # ...
    ports:
      - "27017:27017"
    networks:
      - app-net

networks:
  app-net:
    driver: bridge
```
In this configuration, the backend container is mapped to port 5000 of the host, the client container is mapped to port 3000 of the host, and mongodb container is mapped to port 27017 of the host. All containers are connected to the yolo-network bridge network.


## 4.  Docker Compose Volume Definition and Usage
The Docker Compose file includes volume definitions for MongoDB data storage. The relevant section is as follows:

yaml

```
volumes:
  my-own-volume:/app   # Define Docker volume for MongoDB data
    driver: local

```
This volume, mongodb_data, is designated for storing MongoDB data. It ensures that the data remains intact and is not lost even if the container is stopped or deleted.

## 5. Git Workflow to achieve the task

To achieve the task the following git workflow was used:

1. Fork the repository from the original repository.
2. Clone the repo: `ghttps://github.com/faisoabdirisak/yolo.git`
3. Create a .gitignore file to exclude unnecessary files and directories from version control.
4. Added Dockerfile for the client to the repo:
`git add client/Dockerfile`
5. Add Dockerfile for the backend to the repo:
`git add backend/dockerfile`
6. Committed the changes:
`git commit -m "Added Dockerfiles"`
7. Added docker-compose file to the repo:
`git add docker-compose.yml`
8. Committed the changes:
`git commit -m "Added docker-compose file"`
9. Pushed the files to github:
`git push `
10. Built the client and backend images:
`docker compose build`
11. Pushed the built imags to docker registry:
`docker compose push`
12. Deployed the containers using docker compose:
`docker compose up`

13. Created explanation.md file and modified it as the commit messages in the repo will explain.

