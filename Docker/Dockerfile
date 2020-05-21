FROM debian
ADD debian.list /etc/apt/sources.list.d/
RUN apt-get update && apt-get -y upgrade && apt -y install nginx && apt-get clean && \
    cd /var/www/ && rm -rf ./* && \
    mkdir -p korusha.com/img && \
    chmod -R 754 /var/www/korusha.com/ && \
    useradd Diana && groupadd Koruna && usermod -aG Koruna Diana && \
    chown -R Diana:Koruna /var/www/korusha.com/ && \
    sed -i 's/\/var\/www\/html/\/var\/www\/korusha.com/g' /etc/nginx/sites-enabled/default && \
    sed -i 's/user www-data/user Diana/g' /etc/nginx/nginx.conf
ADD index.html /var/www/korusha.com/
ADD img.jpg /var/www/korusha.com/img/
CMD ["nginx", "-g", "daemon off;"]
