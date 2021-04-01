# mssql-2019-fts-tools
# Maintainers: David Woakes
# GitRepo: https://github.com/woakesd/mssql-docker

# Base OS layer: Latest Ubuntu LTS
FROM ubuntu:18.04

# Install prerequistes since it is needed to get repo config for SQL server
RUN export DEBIAN_FRONTEND=noninteractive && \
    apt-get update && \
    apt-get install -yq curl apt-transport-https gnupg libcap2-bin && \
    useradd -g 0 -u 10001 mssql && \
    # Get official Microsoft repository configuration
    curl https://packages.microsoft.com/keys/microsoft.asc | apt-key add - && \
    curl https://packages.microsoft.com/config/ubuntu/18.04/mssql-server-2019.list | tee /etc/apt/sources.list.d/mssql-server.list && \
    curl https://packages.microsoft.com/config/ubuntu/18.04/prod.list | tee /etc/apt/sources.list.d/mssql-prod.list && \
    apt-get update && \
    # Install SQL Server from apt
    ACCEPT_EULA=Y apt-get install -y mssql-server-fts mssql-tools && \
    # Cleanup the Dockerfile
    apt-get clean && \
    rm -rf /var/lib/apt/lists && \
    # setup for running as mssql
    mkdir -p /var/opt/mssql && \
    chmod 770 /var/opt/mssql && \
    chgrp -R 0 /var/opt/mssql && \
    # Grant sql the permissions to connect to ports <1024 as a non-root user
    /sbin/setcap 'cap_net_bind_service+ep' /opt/mssql/bin/sqlservr && \
    # Allow dumps from the non-root process
    /sbin/setcap 'cap_sys_ptrace+ep' /opt/mssql/bin/paldumper && \
    /sbin/setcap 'cap_sys_ptrace+ep' /usr/bin/gdb && \
    # Add an ldconfig file because setcap causes the os to remove LD_LIBRARY_PATH
    # and other env variables that control dynamic linking
    mkdir -p /etc/ld.so.conf.d && touch /etc/ld.so.conf.d/mssql.conf && \
    echo -e "# mssql libs\n/opt/mssql/lib" >> /etc/ld.so.conf.d/mssql.conf && \
    ldconfig

COPY permissions_check.sh /opt/mssql/bin/

# expose SQL Server port
EXPOSE 1433

# Run SQL Server process
USER mssql
ENTRYPOINT ["/opt/mssql/bin/permissions_check.sh"]
CMD ["/opt/mssql/bin/sqlservr"]
