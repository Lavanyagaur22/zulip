FROM ubuntu:18.04

# Basic packages and dependencies of docker-systemctl-replacement
RUN echo locales locales/default_environment_locale select en_US.UTF-8 | debconf-set-selections \
    && echo locales locales/locales_to_be_generated select "en_US.UTF-8 UTF-8" | debconf-set-selections \
    && apt-get update \
    && apt-get install --no-install-recommends -y \
           ca-certificates \
           curl \
           locales \
           lsb-release \
           openssh-server \
           python3 \
           sudo \
           systemd \
    && rm -rf /var/lib/apt/lists/*

RUN \
    # We use https://github.com/gdraheim/docker-systemctl-replacement
    # to make services we install like postgres, redis, etc. normally
    # managed by systemd start within Docker, which breaks normal
    # operation of systemd.
    dpkg-divert --add --rename /bin/systemctl \
    && curl -so /bin/systemctl 'https://raw.githubusercontent.com/gdraheim/docker-systemctl-replacement/b0588e003562f9a8eb76c98512c6d61146a81980/files/docker/systemctl3.py' \
    && chmod +x /bin/systemctl \
    && ln -nsf /bin/true /usr/sbin/policy-rc.d \
    && mkdir -p /run/sshd \
    # docker-systemctl-replacement doesn’t work with template units yet:
    # https://github.com/gdraheim/docker-systemctl-replacement/issues/62
    && ln -ns /lib/systemd/system/postgresql@.service /etc/systemd/system/postgresql@10-main.service \
    && ln -s /etc/systemd/system/postgresql@10-main.service /etc/systemd/system/multi-user.target.wants/ \
    # redis fails to start with the default configuration if IPv6 is disabled:
    # https://github.com/antirez/redis/pull/5598
    && dpkg-divert --add --rename /etc/default/redis-server \
    && printf 'ULIMIT=65536\nDAEMON_ARGS="/etc/redis/redis.conf --bind 127.0.0.1"\n' > /etc/default/redis-server \
    && mkdir /etc/systemd/system/redis-server.service.d \
    && printf '[Service]\nExecStart=/usr/bin/redis-server /etc/redis/redis.conf --bind 127.0.0.1\n' > /etc/systemd/system/redis-server.service.d/override.conf \
    # Set up the vagrant user and its SSH key (globally public)
    && useradd -ms /bin/bash vagrant \
    && mkdir -m 700 ~vagrant/.ssh \
    && curl -so ~vagrant/.ssh/authorized_keys 'https://raw.githubusercontent.com/hashicorp/vagrant/be7876d83644aa6bdf7f951592fdc681506bcbe6/keys/vagrant.pub' \
    && chown -R vagrant: ~vagrant/.ssh \
    && echo 'vagrant ALL=(ALL) NOPASSWD:ALL' > /etc/sudoers.d/vagrant

CMD ["/bin/systemctl"]

EXPOSE 22
EXPOSE 9991
