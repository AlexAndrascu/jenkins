FROM jenkins/jenkins:lts

USER root

# Install Docker
RUN apt-get update && \
    apt-get install -y docker.io && \
    apt-get clean

# Switch back to jenkins user
USER jenkins

# Install Jenkins plugins
RUN jenkins-plugin-cli --plugins \
    docker-workflow \
    workflow-aggregator \
    git

# Copy custom configuration files if needed
# COPY config.xml /var/jenkins_home/

# Expose Jenkins port
EXPOSE 8080

# Start Jenkins
ENTRYPOINT ["/usr/local/bin/jenkins.sh"]