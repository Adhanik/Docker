
# Calculator App

Suppose you have to write a Dockerfile for a simple calculator. We would start writing by importing a base OS image, for eg - ubuntu, create a working dir, Install python, pip, using RUN command and then once the binary files are downloaded, we will execute the application as part of CMD.

# Problem

The problem with this docker file, even for a basic python calculator application, we only require python runtime to execute the application. If we look at docker image that we have created, it has ubuntu as base image(ubunutu system dependencies, apt packages, etc ) which creates a lot of overload on simple Docker image.

Our python application only requires python runtime to run, neither ubuntu, nor the pip packages, they are only required to build the application. What we conclude from here?

**Building the application &&  Running the application are 2 different stages.**

To build a Java application, we might need 100s of libraries(pom.xml, jar files), but to execute the application, we only need JRE JAVA RUNTIME + JAVA BINARY (jar file)

So we dont need the base image that has lot of overload. So docker introduced the concept of multistage builds.

## MultiStage Builds

We will split our Docker file into multiple parts. Lets start with splitting it into 2 parts.

- We have used ubuntu as its very easy to install all the packages that are necessary to build our application. ubunutu image by default has curl, wget that are required for building the application.
- We will give alias to Stage 1 as build

**STAGE 1**

FROM ubuntu as build
RUN ------(all dependenices install)

We will not execute CMD or ENTRYPOINT HERE. We will take binary from this stage of Dockerfile into the next stage. ->

- So we will have **2 FROM Statement** in our Docker file. 
- We will choose a very **minimal image**, which has only python runtime. this will not have curl, and it was required only during build time. 
- OR you could also choose a **Distroless image**

**STAGE 2**

FROM 

COPY --from build
CMD[]

- Copy the artifact binary from Stage 1 TO Stage 2 and then run the CMD or ENTRYPOINT IN final stage


# 3 Tier Web application

Suppose we have 3 tier application where, front end is React(100mb), backend is java(100mb), and DB is mysql(100mb). we will import a base image(400mb for ubuntu), we have to install all these things in our Dockerfile. then build java application and build frontend, and as part of entrypoint, we will run a /app.ear. This DockerImage would be very big(1G.B)

FROM ubuntu - Why cant we choose directly java runtime? bcoz it wil not have the dependencies like curl, wget, and we will have to install a lot of things. 
**So we choose a  base image that all the dependencies**

Now using Multi stage build, 

- In first stage, we only build the frontend
- In second stage, we only build the backend
- In final stage, we will copy only the binary(50mb) from frontend and backend and pass it in ENTRYPOINT/CMD, using base image only of openjdk.

We can create 'n' no of stages, and there will be only one final stage which will be minimalistic image.

After this we will discuss Distroless images.
