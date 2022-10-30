# GitHub Action to build and publish Docker Images to GitHub Container registry

## Usage examples:

### Build and publish Docker Image with tje `head` tag for the `develop` branch

#### Full workflow example:
```yaml
name: Build and publish

on: 
  push:
    branches:
    - "develop" # Running this workflow only for develop branch

jobs:
  build-and-publish-head:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v2.5.0 # Checking out the repo

    - name: Build and Publish head Docker image
      uses: VaultVulp/gp-docker-action@1.4.0
      with:
        github-token: ${{ secrets.GITHUB_TOKEN }} # Provide GITHUB_TOKEN to login into the GitHub Packages
        image-name: my-cool-service # Provide Docker image name
        image-tag: head # Provide Docker image tag
```

### Build and publish Docker Image with a `latest` tag for the `master` branch with different dockerfile

#### Full workflow example:
```yaml
name: Build and publish

on: 
  push:
    branches:
    - "master" # Running this workflow only for master branch

jobs:
  build-and-publish-latest:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v2.5.0 # Checking out the repo

    - name: Build and Publish latest Docker image
      uses: VaultVulp/gp-docker-action@1.4.0
      with:
        github-token: ${{ secrets.GITHUB_TOKEN }} # Provide GITHUB_TOKEN to login into the GitHub Packages
        image-name: my-cool-service # Provide only Docker image name, tag will be automatically set to latest
        dockerfile: Alternative.Dockerfile # Provide custom Dockerfile name
```

### Build and publish Docker Image with a tag equal to a git tag

#### Full workflow example:
```yaml
name: Build and publish

on: 
  push:
    tags:
    - "*" # Running this workflow for any tag

jobs:
  build-and-publish-tag:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v2.5.0 # Checking out the repo
    
    - name: Build and Publish Tag Docker image
      uses: VaultVulp/gp-docker-action@1.4.0
      with:
        github-token: ${{ secrets.GITHUB_TOKEN }} # Provide GITHUB_TOKEN to login into the GitHub Packages
        image-name: my-cool-service # Provide only Docker image name
        extract-git-tag: true # Provide flag to extract Docker image tag from git reference
```

### Build and publish Docker Image with a different build context

#### Full workflow example:
```yaml
name: Build and publish

on: push

jobs:
  build-and-publish-context:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v2.5.0 # Checking out the repo
    
    - name: Build and Publish Docker image from a different context
      uses: VaultVulp/gp-docker-action@1.4.0
      with:
        github-token: ${{ secrets.GITHUB_TOKEN }} # Provide GITHUB_TOKEN to login into the GitHub Packages
        image-name: my-cool-service # Provide Docker image name
        build-context: ./dev # Provide path to the folder with a Dockerfile
```

### Pulling an image before building it

#### Full workflow example:
```yaml
name: Build and publish

on: push

jobs:
  pull-and-build-and-publish:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v2.5.0 # Checking out the repo

    - name: Build and Publish head Docker image
      uses: VaultVulp/gp-docker-action@1.4.0
      with:
        github-token: ${{ secrets.GITHUB_TOKEN }} # Provide GITHUB_TOKEN to login into the GitHub Packages
        image-name: my-cool-service # Provide Docker image name
        pull-image: true # Provide the flag to pull image
```

### Passing additional image tags

**NB**: `additional-image-tags` will **not** replace `image-tag` argument - additional tags will be appended to the list. If no `image-tag` was specified, then image will be tagged with the `latest` tag.

#### Examples:

##### `image-tag` was specified: 
```yaml
image-name: my-cool-service
image-tags: first
additional-image-tags: second third
```
Action will produce one image with three tags:
- `my-cool-service:first`
- `my-cool-service:second`
- `my-cool-service:third`

##### No `image-tag` was specified: 

In this case action will use the default `latest` tag.

```yaml
image-name: my-cool-service
additional-image-tags: second third
```
Action will produce one image with three tags:
- `my-cool-service:latest`
- `my-cool-service:second`
- `my-cool-service:third`

#### Full workflow example:
```yaml
name: Build and publish 

on: push

jobs:
  build-with-custom-args:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v2.5.0 # Checking out the repo
 
    - name: Build with --build-arg(s)
      uses: VaultVulp/gp-docker-action@1.4.0
      with:
        github-token: ${{ secrets.GITHUB_TOKEN }} # Provide GITHUB_TOKEN to login into the GitHub Packages
        image-name: my-cool-service # Provide Docker image name
        image-tags: first # if ommitted will be replaced with "latest"
        additional-image-tags: second third # two additional tags for an image
```

### Passing additional arguments to the docker build command

**NB**, additional arguments should be passed with the `=` sign istead of a ` `(space) between argument name and values.

Correct example: 
```yaml
custom-args: --build-arg=some="value" 
                      # ^ this "=" is mandatory
```
Incorrect example:
```yaml
custom-args: --build-arg some="value" 
                      # ^ this space might break the action
```

#### Full workflow example:
```yaml
name: Build and publish

on: push

jobs:
  build-with-custom-args:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v2.5.0 # Checking out the repo
 
    - name: Build with --build-arg(s)
      uses: VaultVulp/gp-docker-action@1.4.0
      with:
        github-token: ${{ secrets.GITHUB_TOKEN }} # Provide GITHUB_TOKEN to login into the GitHub Packages
        image-name: my-cool-service # Provide Docker image name
        custom-args: --build-arg=some="value" --build-arg=some_other="value" # Pass some additional arguments to the docker build command
```

## Security considerations

You will encounter the following log message in your GitHub Actions Pipelines:

```
WARNING! Using --password via the CLI is insecure. Use --password-stdin.
WARNING! Your password will be stored unencrypted in /github/home/.docker/config.json.
Login Succeeded
```

I would like to ensure you, that I do not store your secrets, passwords, token, or any other information.

This warning informs you about the fact, that this Action passes your GitHub token via the command line argument:
```bash
docker login -u publisher -p ${DOCKER_TOKEN} ghcr.io
```

In a non-safe environment, this could raise a security issue, but this is not the case. We are passing a temporary authorization token, which will expire once the pipeline is completed. It would also require additional code to extract this token from the environment or `docker` internals, that this Action does not have.

[This](https://docs.github.com/en/packages/managing-github-packages-using-github-actions-workflows/publishing-and-installing-a-package-with-github-actions#upgrading-a-workflow-that-accesses-a-registry-using-a-personal-access-token
) is the detailed explanation about the `${{ secrets.GITHUB_TOKEN }}` and it's relations with the GCR.
