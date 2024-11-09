FROM jenkins/jenkins:latest
USER root

# Install required packages
RUN apt-get update && \
    apt-get install -y \
    docker.io \
    curl \
    wget && \
    apt-get clean

# Install k3s binary
# RUN curl -sfL https://get.k3s.io > /usr/local/bin/k3s && \
#     chmod +x /usr/local/bin/k3s

# Create required directories
RUN mkdir -p /home/jenkins/.kube && \
    mkdir -p /var/jenkins_home/jobs

# Configure permissions
RUN chown -R jenkins:jenkins /home/jenkins/.kube && \
    chown -R jenkins:jenkins /var/jenkins_home/jobs

USER jenkins

# Install required plugins
RUN jenkins-plugin-cli --plugins \
    configuration-as-code \
    job-dsl \
    workflow-aggregator \
    docker-workflow \
    docker-plugin \
    docker-slaves \
    kubernetes \
    kubernetes-credentials \
    matrix-auth