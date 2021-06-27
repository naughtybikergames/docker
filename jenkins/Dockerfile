FROM debian:stretch

ARG jenkins_version=2.289.1
ARG jenkins_sha=70f9cc6ff1ac59aeeb831b980709a9ddb0ee70d216ee50625a8508b9840f75f2

ARG tini_version=0.19.0
ARG tini_sha=c5b0666b4cb676901f90dfcb37106783c5fe2077b04590973b885950611b30ee

ARG jenkins_url=https://repo.jenkins-ci.org/releases/org/jenkins-ci/main/jenkins-war/${jenkins_version}

ARG user=jenkins
ARG group=jenkins
ARG uid=1000
ARG gid=1000

ENV DEBIAN_FRONTEND=noninteractive

ENV JENKINS_VERSION=${jenkins_version}
ENV C2D_RELEASE=${jenkins_version}
ENV JENKINS_HOME=/var/jenkins_home
ENV JENKINS_PORT=8080
ENV JENKINS_SLAVE_AGENT_PORT=50000

ENV JENKINS_UC=https://updates.jenkins.io

ENV COPY_REFERENCE_FILE_LOG=/var/jenkins_home/copy_reference_file.log

ENV OPENJDK_VERSION=8
ENV GAE_IMAGE_NAME=openjdk8
ENV GAE_IMAGE_LABEL=8-2021-03-30-17-30

COPY setup-env.d/ /setup-env.d/
COPY shutdown/ /shutdown/
COPY usr/share/jenkins/ref/init.groovy.d/tcp-slave-agent-port.groovy /usr/share/jenkins/ref/init.groovy.d/tcp-slave-agent-port.groovy
COPY usr/local/bin/jenkins-support /usr/local/bin/jenkins-support
COPY usr/local/bin/jenkins.sh /usr/local/bin/jenkins.sh
COPY usr/local/bin/plugins.sh /usr/local/bin/plugins.sh
COPY usr/local/bin/install-plugins.sh /usr/local/bin/install-plugins.sh
COPY usr/share/jenkins/ref/init.groovy.d/monitoring-plugin.groovy /usr/share/jenkins/ref/init.groovy.d/monitoring-plugin.groovy

ADD https://storage.googleapis.com/cloud-debugger/appengine-java/current/cdbg_java_agent.tar.gz /opt/cdbg/
ADD https://storage.googleapis.com/cloud-profiler/java/latest/profiler_java_agent.tar.gz /opt/cprof/

VOLUME [/var/jenkins_home]

RUN apt-get -q update && apt-get -y -q --no-install-recommends install ca-certificates-java openjdk-8-jdk-headless procps netbase wget unzip && apt-get clean && rm /var/lib/apt/lists/*_*
RUN tar Cxfvz /opt/cdbg /opt/cdbg/cdbg_java_agent.tar.gz --no-same-owner && rm /opt/cdbg/cdbg_java_agent.tar.gz && tar Cxfvz /opt/cprof /opt/cprof/profiler_java_agent.tar.gz --no-same-owner && rm /opt/cprof/profiler_java_agent.tar.gz && chmod +x /shutdown/*.bash /setup-env.d/*.bash && mkdir /var/log/app_engine && chmod go+rwx /var/log/app_engine
RUN apt-get update && apt-get install -y git curl && rm -rf /var/lib/apt/lists/*
RUN groupadd -g ${gid} ${group} && useradd -d "$JENKINS_HOME" -u ${uid} -g ${gid} -m -s /bin/bash ${user}
RUN mkdir -p /usr/share/jenkins/ref/init.groovy.d
RUN curl -fsSL https://github.com/krallin/tini/releases/download/v${tini_version}/tini-static-amd64 -o /bin/tini && chmod +x /bin/tini   && echo "${tini_sha}  /bin/tini" | sha256sum -c -
RUN mkdir -p /usr/share/doc/tini && curl -fsSL https://raw.githubusercontent.com/krallin/tini/master/LICENSE -o /usr/share/doc/tini/copyright
RUN curl -fsSL ${jenkins_url}/jenkins-war-${JENKINS_VERSION}.war -o /usr/share/jenkins/jenkins.war && echo "${jenkins_sha}  /usr/share/jenkins/jenkins.war" | sha256sum -c -
RUN mkdir /usr/share/doc/jenkins && curl -fsSL ${jenkins_url}/jenkins-war-${JENKINS_VERSION}.license.xml -o /usr/share/doc/jenkins/license.xml
RUN chown -R ${user} "$JENKINS_HOME" /usr/share/jenkins/ref
RUN chown ${user}:${group} /usr/local/bin/*.sh && chown ${user}:${group} /usr/local/bin/jenkins-support && chown -R ${user}:${group} /usr/share/jenkins/

EXPOSE ${JENKINS_PORT}
EXPOSE ${JENKINS_SLAVE_AGENT_PORT}

USER jenkins

ENTRYPOINT ["/bin/tini", "--", "/usr/local/bin/jenkins.sh"]
