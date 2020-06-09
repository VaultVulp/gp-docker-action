# GitHub Action to build and publish Docker Images to GitHub Package Registry

## Usage examples:

### Build and publish Docker Image with a `head` tag for the `develop` branch

```yaml
  build-and-publish-head:
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/develop' # Running this job only for develop branch

    steps:
    - uses: actions/checkout@v2 # Checking out the repo

    - name: Build and Publish head Docker image
      uses: VaultVulp/gp-docker-action@1.1.1
      with:
        github-token: ${{ secrets.GITHUB_TOKEN }} # Provide GITHUB_TOKEN to login into the GitHub Packages
        image-name: my-cool-service # Provide Docker image name
        image-tag: head # Provide Docker image tag
```

### Build and publish Docker Image with a `latest` tag for the `master` branch with different dockerfile

```yaml
  build-and-publish-latest:
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/master' # Running this job only for master branch

    steps:
    - uses: actions/checkout@v2 # Checking out the repo

    - name: Build and Publish latest Docker image
      uses: VaultVulp/gp-docker-action@1.1.1
      with:
        github-token: ${{ secrets.GITHUB_TOKEN }} # Provide GITHUB_TOKEN to login into the GitHub Packages
        image-name: my-cool-service # Provide only Docker image name, tag will be automatically set to latest
        dockerfile: Dockerfile_server
```

### Build and publish Docker Image with a tag equal to a git tag

```yaml
  build-and-publish-tag:
    runs-on: ubuntu-latest
    if: startsWith(github.ref, 'refs/tags/') # Running this job only for tags

    steps:
    - uses: actions/checkout@v2

    - name: Build and Publish Tag Docker image
      uses: VaultVulp/gp-docker-action@1.1.1
      with:
        github-token: ${{ secrets.GITHUB_TOKEN }} # Provide GITHUB_TOKEN to login into the GitHub Packages
        image-name: my-cool-service # Provide only Docker image name
        extract-git-tag: true # Provide flag to extract Docker image tag from git reference
```

### Build and publish Docker Image with a differnet build context

```yaml
  build-and-publish-dev:
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/develop' # Running this job only for develop branch

    steps:
    - uses: actions/checkout@v2 # Checking out the repo

    - name: Build and Publish head Docker image
      uses: VaultVulp/gp-docker-action@1.1.1
      with:
        github-token: ${{ secrets.GITHUB_TOKEN }} # Provide GITHUB_TOKEN to login into the GitHub Packages
        image-name: my-cool-service # Provide Docker image name
        build-context: ./dev # Provide path to the folder with the Dockerfile
```

### Pulling the image before building it

```yaml
  pull-and-build-dev:
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/develop' # Running this job only for develop branch

    steps:
    - uses: actions/checkout@v2 # Checking out the repo

    - name: Build and Publish head Docker image
      uses: VaultVulp/gp-docker-action@1.1.1
      with:
        github-token: ${{ secrets.GITHUB_TOKEN }} # Provide GITHUB_TOKEN to login into the GitHub Packages
        image-name: my-cool-service # Provide Docker image name
        pull-image: true # Raise the flag to try to pull image
```
