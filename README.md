# jenkins-docker

## Overview
This project sets up a Jenkins CI/CD environment using Docker. It includes a Docker-in-Docker (dind) service to enable Jenkins to run Docker commands within its own container.

## Project Structure
```
jenkins-docker
├── docker-compose.yml
├── jenkins
│   └── Dockerfile
├── .env
└── README.md
```

## Setup Instructions

1. **Clone the Repository**
   ```bash
   git clone <repository-url>
   cd jenkins-docker
   ```

2. **Configure Environment Variables**
   - Update the `.env` file with your desired configurations, including Jenkins admin credentials.

3. **Build and Start the Containers**
   ```bash
   docker-compose up -d
   ```

4. **Access Jenkins**
   - Open your web browser and navigate to `http://localhost:8080` to access the Jenkins dashboard.

## Usage Guidelines
- Use the Jenkins interface to create and manage your CI/CD pipelines.
- Ensure that the Docker-in-Docker service is running to allow Jenkins to execute Docker commands.

## Additional Information
- For more details on configuring Jenkins and using Docker, refer to the official Jenkins documentation.
- Make sure to monitor the logs for any issues during the setup or execution of your pipelines.

## License
This project is licensed under the MIT License.