# Utilizamos la imagen base apache-ssl-ftp-ssh
FROM apache-ssl-ftp-ssh:latest

# Damos información sobre la imagen que estamos creando
LABEL \
    version="2.0" \
    description="Ubuntu + Apache2 + ProFTPD + OpenSSH" \
    maintainer="jzurro <jzurro@birt.eus>"

# Instalamos los paquetes necesarios
RUN apt-get update && \
    apt-get install -y proftpd openssh-server git

# Creamos los usuarios
RUN useradd -m -d /var/www/html/sitioPrimero -s /usr/sbin/nologin jzurro1 && \
    echo "jzurro1:jzurro1" | chpasswd && \
    mkdir -p /var/www/html/sitioPrimero && \
    chown -R jzurro1:jzurro1 /var/www/html/sitioPrimero && \
    chmod -R 755 /var/www/html/sitioPrimero

RUN useradd -m -d /var/www/html/sitioSegundo -s /bin/bash jzurro2 && \
    echo "jzurro2:jzurro2" | chpasswd && \
    mkdir -p /var/www/html/sitioSegundo && \
    chown -R jzurro2:jzurro2 /var/www/html/sitioSegundo && \
    chmod -R 755 /var/www/html/sitioSegundo

# Copiamos todos los archivos desde el directorio "ficheros" al contenedor
COPY ficheros/ /ficheros/

# Movemos los archivos a sus ubicaciones adecuadas
RUN mkdir -p /etc/proftpd/ssl && \
    mv /ficheros/proftpd.conf /etc/proftpd/proftpd.conf && \
    mv /ficheros/tls.conf /etc/proftpd/tls.conf && \
    mv /ficheros/proftpd.key /etc/ssl/private/proftpd.key && \
    mv /ficheros/proftpd.pem /etc/ssl/certs/proftpd.pem && \
    mv /ficheros/sshd_config /etc/ssh/sshd_config && \
    chmod 600 /etc/ssl/private/proftpd.key && \
    chmod 644 /etc/ssl/certs/proftpd.pem

# Clonamos el repositorio Git para el FTP anónimo
RUN mkdir -p /tmp/gitclone && \
    mkdir -p /var/ftp/anonimo && \
    mv /ficheros/id_rsa /tmp/id_rsa && \
    chmod 600 /tmp/id_rsa && \
    ssh-keyscan -t rsa github.com > /tmp/known_hosts && \
    GIT_SSH_COMMAND="ssh -i /tmp/id_rsa -o UserKnownHostsFile=/tmp/known_hosts -o StrictHostKeyChecking=yes" \
    git clone git@github.com:deaw-birt/proyecto-html.git /tmp/gitclone && \
    mv /tmp/gitclone/* /var/ftp/anonimo/ && \
    rm -rf /tmp/gitclone /tmp/id_rsa /tmp/known_hosts

# Establecemos permisos
RUN chmod -R 755 /var/www/html && \
    chmod -R 755 /var/ftp/anonimo

# Exponemos los puertos
EXPOSE 21
EXPOSE 50000-50030
EXPOSE 2222

# Comando por defecto al iniciar el contenedor
CMD ["sh", "-c", "service ssh start && service proftpd start && apachectl -D FOREGROUND"]