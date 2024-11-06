FROM jenkins/jenkins:lts
USER root

# Install required packages
RUN apt-get update && \
    apt-get install -y \
    docker.io \
    curl \
    wget && \
    apt-get clean

# Install k3s binary
RUN curl -sfL https://get.k3s.io > /usr/local/bin/k3s && \
    chmod +x /usr/local/bin/k3s

# Create required directories
RUN mkdir -p /etc/rancher/k3s && \
    mkdir -p /var/lib/rancher/k3s && \
    mkdir -p /var/log/k3s && \
    mkdir -p /home/jenkins/.kube

# Configure permissions
RUN chown -R jenkins:jenkins /etc/rancher/k3s && \
    chown -R jenkins:jenkins /var/lib/rancher/k3s && \
    chown -R jenkins:jenkins /var/log/k3s && \
    chown -R jenkins:jenkins /home/jenkins/.kube

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
    kubernetes-credentials