
# Distroless images

A Distroless image is a minimalistic/lightweight docker image that will have have  only runtime env. For eg, a python distroless image consist only of python runtime.


# Advantage of Distroless images

1. Security - Previously we were using ubuntu as base images, and which are vulnerable so mvoing to distroless image which only had python runtime, so no basic packages were there, hence giving highest security, so no OS related vulnebrality were there.

2. For applications, like GO language, they provide even more advantage with distroless images, as we dont event need a runtime because they are statically typed.


Now we have cloned the golang-multi-stage-docker-build in our repo, we will create Dockerfile for the calcualator which is a go application.

### Explaination for multi stage build 

## Build stage

- In multi stage Dockerfile,  each stage can be given a name using AS.

- In Stage 1, i dont have CMD or ENTRYPOINT. Stage 1 is dedicated only to BUILD stage

###########################################
# BASE IMAGE
###########################################

FROM ubuntu AS build

RUN apt-get update && apt-get install -y golang-go

ENV GO111MODULE=off

COPY . .

RUN CGO_ENABLED=0 go build -o /app .

## Final stage

- In final stage, we are using a special image called scratch. This is minimilastic distroless image that we have.

- Scratch will fail for python application as python app requires python runtime still. GO does not need a runtime, hence we can make use of `scratch` image.

- Install python if u use scratch image or  use python or java distroless images.

- --from=build: This flag specifies that the source of the files being copied comes from a previous build stage named build. Here it indiactes that we are copying the binary from build stage, and we are executing it as part of ENTRYPOINT.

- /app(source path): this is the path inside the build stage where the files are located. It indicates that the content of /app dir from build stage will be copied.

- /app (Destination path): This is the path in the current stage(the final stage) where the files will be copied to. It will place the contents from the previous stage's /app dir into the /app dir of current image.

############################################
# HERE STARTS THE MAGIC OF MULTI STAGE BUILD
############################################

FROM scratch

# Copy the compiled binary from the build stage
COPY --from=build /app /app

# Set the entrypoint for the container to run the binary
ENTRYPOINT ["/app"]



### RESULT

WITH MULTI STAGE BUILD

```
adminik/simplecalwithmultistage   latest    4057884431b5   11 seconds ago   2.03MB

```
WITHOUT MULTI STAGE BUILD

```

adminnik/simplecal                latest    f022c341025f   10 minutes ago   651MB

```