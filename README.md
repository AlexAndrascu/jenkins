# Project Documentation

## Overview

This project sets up a **Jenkins CI/CD environment** using **Docker** and **Docker Compose**. It includes a Jenkins server configured with Configuration as Code (JCasC) and a Docker-in-Docker (dind) service to enable Jenkins agents to execute Docker commands within their containers. The environment also demonstrates deploying a sample Kubernetes application using **k3s** within a Jenkins pipeline.

## Project Structure

```
.
├── .env
├── docker-compose.yml
├── README.md
└── jenkins/
    ├── Dockerfile
    ├── casc/
    │   └── jenkins.yaml
    └── jobs/
```

### Breakdown

- **.env**: Contains environment variables used by Docker Compose and Jenkins.
- **docker-compose.yml**: Defines the Docker services: Jenkins and Docker-in-Docker.
- **README.md**: Documentation and usage instructions.
- **jenkins/**
  - **Dockerfile**: Custom Jenkins image with additional tools and plugins.
  - **casc/**
    - **jenkins.yaml**: Jenkins Configuration as Code file.
  - **jobs/**: Directory for Jenkins job scripts (if any).

## Setup Instructions

### Prerequisites

- **Docker** and **Docker Compose** installed on your system.
- For Windows users, Docker Desktop is required.

### Steps

1. **Clone the Repository**

   ```bash
   git clone <repository-url>
   cd <repository-directory>
   ```

2. **Configure Environment Variables**

   - Edit the .env

 file to set your desired Jenkins admin credentials and Docker host settings:

     ```dotenv
     JENKINS_ADMIN_USER=admin
     JENKINS_ADMIN_PASSWORD=admin123
     JENKINS_URL=http://localhost:8080
     DOCKER_HOST=tcp://host.docker.internal:2375
     DOCKER_TLS_CERTDIR=
     DOCKER_TLS_VERIFY=1
     ```

3. **Adjust Docker Settings (Windows Users)**

   - Ensure Docker is set to expose the daemon on TCP without TLS:
     - Open **Docker Desktop** settings.
     - Navigate to **Settings** > **General**.
     - Check **"Expose daemon on tcp://localhost:2375 without TLS"**.
     - ⚠️ Make sure you understand the security implication of this ⚠️ \
     ( i.e. This is not a production ready deployment. Use for local / demo purposes only )

4. **Build and Start the Docker Containers**

   ```bash
   docker-compose up -d
   ```

   - This command builds the custom Jenkins image and starts both Jenkins and dind services.

5. **Access Jenkins Web Interface**

   - Open your browser and navigate to [http://localhost:8080](http://localhost:8080).
   - Log in using the credentials specified in the .env file.

6. **Verify Jenkins Configuration**

   - Jenkins should display a system message indicating it's configured using Configuration as Code.
   - The predefined pipeline job **`k8s-deployment`** should be visible.

## Jenkins Configuration Details

### Jenkins Configuration as Code (`jenkins/casc/jenkins.yaml`)

- **System Message**: Indicates that Jenkins is configured using JCasC.
- **Executors**: Configured to use two executors.
- **Security Realm**:
  - Uses a local security realm with the admin user defined in the .env file.
- **Authorization Strategy**:
  - Implements a project matrix authorization strategy.
  - Grants `Overall/Administer` permission to the admin user.
  - Grants `Overall/Read` permission to authenticated users.
- **Cloud Configuration**:
  - **Docker Cloud** named `docker-cloud`.
  - Connects to Docker using the `dind` service at `tcp://host.docker.internal:2375`.
  - Defines a Docker agent template with:
    - **Image**: `jenkins/inbound-agent:latest`.
    - **Environment Variables**: Sets `DOCKER_HOST` for Docker commands.
    - **Privileged Mode**: Enabled to allow nested container operations.
    - **User**: Runs as `root` to perform administrative tasks.
    - **Instance Cap**: Limits to 10 instances.

### Pipeline Job: `k8s-deployment`

Defined within jenkins.yaml, this pipeline performs the following stages:

1. **Setup Environment**
   - Updates package lists.
   - Installs `wget` and `curl`.

2. **Install k3s**
   - Downloads the `k3s` binary directly.
   - Makes it executable and starts the server without `systemd` or `openrc`.
   - Waits for the server to be ready by checking node availability.
   - Outputs the k3s server logs for diagnostics.

3. **Deploy Sample App**
   - Uses `k3s kubectl` to deploy `hello-node` application from the `k8s.gcr.io/echoserver:1.4` image.

4. **Expose Sample App**
   - Exposes the deployment via a `NodePort` service on port `8080`.
   - Waits for the service to become available.
   - Retrieves and displays the service details.

## Custom Jenkins Docker Image (jenkins\Dockerfile)

- **Base Image**: `jenkins/jenkins:latest`.
- **User**: Switches to `root` to install additional packages.
- **Installed Packages**:
  - `docker.io`: Docker Engine to allow Docker commands within Jenkins.
  - `curl` and `wget`: Tools required for pipeline operations.
- **Directories and Permissions**:
  - Creates necessary directories for Kubernetes configurations and Jenkins jobs.
  - Adjusts ownership to the jenkins user.
- **Switch Back to Jenkins User**:
  - Ensures Jenkins runs under the correct user for security.
- **Installed Plugins**:
  - `configuration-as-code`
  - `job-dsl`
  - `workflow-aggregator`
  - `docker-workflow`
  - `docker-plugin`
  - `docker-slaves`
  - `kubernetes`
  - `kubernetes-credentials`
  - `matrix-auth`

## Docker Compose Configuration (`docker-compose.yml`)

Defines two main services:

### 1. Jenkins Service

- **Build Context**: Uses the custom Dockerfile from jenkins
- **Ports**:
  - `8080`: Jenkins web interface.
  - `50000`: Jenkins agent communication.
- **Volumes**:
  - `jenkins_home`: Persistent storage for Jenkins data.
  - Binds the `casc` directory for JCasC configuration.
- **Environment Variables**:
  - Pulls in variables from the .env file.
  - Sets `JENKINS_HOME`.
- **Privileged Mode**: Enabled to allow Docker operations.
- **Depends On**: Starts after the `dind` service is up.

### 2. Docker-in-Docker (dind) Service

- **Image**: `docker:dind`.
- **Privileged Mode**: Required for Docker daemon.
- **Command**: Starts the Docker daemon and exposes it on TCP port `2375`.
- **Environment Variables**:
  - Disables TLS verification for simplicity.
- **Ports**:
  - `2375`: Exposes Docker daemon for the Jenkins container to access.

## Environment Variables (`.env`)

- **JENKINS_ADMIN_USER**: Admin username for Jenkins.
- **JENKINS_ADMIN_PASSWORD**: Admin password for Jenkins.
- **JENKINS_URL**: URL where Jenkins is accessible.
- **DOCKER_HOST**: TCP address to communicate with the Docker daemon.
- **DOCKER_TLS_CERTDIR**: Left empty to disable TLS verification.
- **DOCKER_TLS_VERIFY**: Set to `1` to require TLS verification (adjust as needed).

## Usage Guidelines

- **Running the Pipeline**:
  - Navigate to the `k8s-deployment` job in Jenkins.
  - Click **"Build Now"** to start the pipeline.
  - Monitor the console output for progress and any errors.
- **Accessing the Sample Application**:
  - Since the application is deployed within a Docker container, additional network configurations may be needed to access it from the host machine.
- **Troubleshooting**:
  - Check Jenkins logs and pipeline console outputs for error messages.
  - Verify that the `dind` service is running and accessible.
  - Ensure environment variables are correctly set, especially `DOCKER_HOST`.

## Additional Notes

- **Windows Users**:
  - The `host.docker.internal` hostname is used to allow containers to communicate with the host Docker daemon. This is supported in Docker Desktop for Windows and Mac.
- **Security Considerations**:
  - Running containers in privileged mode and as `root` can pose security risks. This setup is intended for development and testing purposes.
- **Extensibility**:
  - Additional Jenkins jobs can be added by editing jenkins.yaml or placing job configurations in the `jobs/` directory.
  - Customize the Jenkins Docker image by modifying the Dockerfile to include other tools or plugins as needed.
- **Cleaning Up**:
  - To stop and remove the Docker containers, run:
    ```bash
    docker-compose down
    ```

## Potential Issues and Solutions

- **Connection Refused Errors**:
  - If Jenkins agents cannot connect to the Docker daemon, ensure that the `DOCKER_HOST` environment variable is correctly set and that the Docker daemon is exposed on the correct port.
- **k3s Installation Failures**:
  - The `k3s` binary is directly downloaded and started without `systemd` or `openrc`. Ensure that the pipeline runs as `root` to have the necessary permissions.
- **Networking Problems**:
  - Containers may have networking restrictions. Adjust Docker network settings if services cannot communicate as expected.

## Conclusion

This project provides a foundational setup for a Jenkins CI/CD pipeline capable of running Docker commands and deploying applications to a Kubernetes cluster using k3s. It demonstrates how to configure Jenkins agents dynamically using Docker and how to manage Jenkins configuration using Configuration as Code.

By following this documentation, you should be able to reproduce the environment, understand its components, and extend it to suit your CI/CD requirements.
