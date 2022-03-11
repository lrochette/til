# Building ARM images using buildx

*Note:* As you need access to the docker daemon, this can only work on a Hybrid runner

```
version: "1.0"
stages:
  - "clone"
  - "build"

steps:
  clone:
    title: "Cloning repository"
    type: "git-clone"
    repo: "${{CF_REPO_OWNER}}/${{CF_REPO_NAME}}"
    revision: "${{CF_REVISION}}"
    stage: "clone"

  buildx:
    title: "Building/Pushing ARMv7 Docker image"
    type: "freestyle"
    working_directory: ${{clone}}
    image: dustinvanbuskirk/buildx-dind:20.10
    commands:
      - docker run --privileged --rm tonistiigi/binfmt --install all
      - docker context create cf-environment
      - docker buildx create --name multiarch-builder --driver docker-container --use cf-environment
      - docker login --username <username> --password <password> <docker host>
      - docker buildx build --platform linux/arm/v7 -t <docker-image>:<tag> --push .
    stage: "build"
```
