FROM jenkins/jenkins:lts-jdk17
USER root
ARG KUBECTL_VERSION=v1.31.0
RUN apt-get update \
 && apt-get install -y --no-install-recommends python3 python3-venv python3-pip git jq curl ca-certificates \
 && rm -rf /var/lib/apt/lists/*
COPY --from=docker:cli /usr/local/bin/docker /usr/local/bin/docker
COPY --from=aquasec/trivy:latest /usr/local/bin/trivy /usr/local/bin/trivy
COPY --from=hadolint/hadolint:latest /bin/hadolint /usr/local/bin/hadolint
RUN curl -fsSL -o /usr/local/bin/kubectl "https://dl.k8s.io/release/${KUBECTL_VERSION}/bin/linux/amd64/kubectl" \
 && chmod +x /usr/local/bin/kubectl
RUN python3 -m venv /opt/ansible \
 && /opt/ansible/bin/pip install --no-cache-dir ansible-core kubernetes PyYAML jsonpatch \
 && ln -s /opt/ansible/bin/ansible /opt/ansible/bin/ansible-playbook /opt/ansible/bin/ansible-galaxy /opt/ansible/bin/ansible-vault /usr/local/bin/ \
 && /opt/ansible/bin/ansible-galaxy collection install kubernetes.core community.docker -p /opt/ansible-collections
ENV ANSIBLE_COLLECTIONS_PATH=/opt/ansible-collections
USER jenkins
COPY --chown=jenkins:jenkins plugins.txt /usr/share/jenkins/ref/plugins.txt
RUN jenkins-plugin-cli --plugin-file /usr/share/jenkins/ref/plugins.txt
