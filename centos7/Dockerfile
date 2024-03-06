FROM centos:7

ARG user=jenkins
ARG group=jenkins
ARG uid=1000
ARG gid=1000
ARG JENKINS_AGENT_HOME=/home/${user}

ENV JENKINS_AGENT_HOME=${JENKINS_AGENT_HOME}

RUN groupadd -g ${gid} ${group} \
    && useradd -d "${JENKINS_AGENT_HOME}" -u "${uid}" -g "${gid}" -G wheel -m -s /bin/bash "${user}" \
    # Prepare subdirectories
    && mkdir -p "${JENKINS_AGENT_HOME}/.ssh/" \
    # Make sure that user 'jenkins' own these directories and their content
    && chown -R "${uid}":"${gid}" "${JENKINS_AGENT_HOME}"

RUN yum makecache \
  && yum install -y openssh-server \
  		    java-11-openjdk \
                    git \
                    python3 \
                    python3-pip \
                    sudo \
                    # umd-verification deps \
                    gcc \
                    python3-devel \
                    openldap-devel \
  && yum clean all

# symbolic link for pip
RUN ln -sf /usr/bin/pip3 /usr/bin/pip
# RUN ln -sf /usr/bin/python3.6 /usr/bin/python

# config sudo
RUN sed -i '/%wheel/s/^/#/' /etc/sudoers \
    && sed -i 's/^#*\s*\(%wheel\s\+ALL=(ALL)\s\+NOPASSWD:\s\+ALL\)/\1/' /etc/sudoers

# Install gosu.  https://github.com/tianon/gosu
# ENV GOSU_VERSION=1.16
# RUN gpg --keyserver pgp.mit.edu --recv-keys B42F6819007F00F88E364FD4036A9C25BF357DD4 \
#     && curl -o /usr/local/bin/gosu -SL "https://github.com/tianon/gosu/releases/download/${GOSU_VERSION}/gosu-amd64" \
#     && curl -o /usr/local/bin/gosu.asc -SL "https://github.com/tianon/gosu/releases/download/${GOSU_VERSION}/gosu-amd64.asc" \
#     && gpg --verify /usr/local/bin/gosu.asc \
#     && rm /usr/local/bin/gosu.asc \
#     && rm -r /root/.gnupg/ \
#     && chmod +x /usr/local/bin/gosu \
#     # Verify that the binary works
#     && gosu nobody true

# setup SSH server
RUN sed -i /etc/ssh/sshd_config \
        -e 's/#PermitRootLogin.*/PermitRootLogin no/' \
        -e 's/#RSAAuthentication.*/RSAAuthentication yes/'  \
        -e 's/#PasswordAuthentication.*/PasswordAuthentication no/' \
        -e 's/#SyslogFacility.*/SyslogFacility AUTH/' \
        -e 's/#LogLevel.*/LogLevel INFO/' && \
    mkdir /var/run/sshd

COPY setup-sshd /usr/local/bin/setup-sshd

# Expose docker port 22
EXPOSE 22

ENTRYPOINT ["setup-sshd"]

