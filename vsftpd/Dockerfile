FROM debian:bookworm AS base

# Install packages.
RUN set -xe; \
    apt-get -yqq update; \
    apt-get -yqq install vsftpd \
    ;

# Create FTP user
RUN useradd -d /home/ftpuser -p "$(openssl passwd -6 ftpuserpass)" ftpuser

# Configure vsftpd.
RUN mkdir -p /var/run/vsftpd/empty && \
    touch /var/log/vsftpd.log
RUN echo "ftpuser" > /etc/vsftpd.userlist

COPY ./vsftpd.conf /etc/vsftpd/vsftpd.conf

COPY --chmod=755 ./wrapper.sh /wrapper.sh
