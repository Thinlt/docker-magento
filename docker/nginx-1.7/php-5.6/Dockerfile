FROM php:5.6-fpm

LABEL maintainer="Tit <mrthinlt@gmail.com>"

ENV WEB_USER www-data
ENV WEB_GROUP www-data

# Inatall Nginx
RUN apt-get update && apt install -y \
	gnupg2 ca-certificates lsb-release build-essential zlibc zlib1g zlib1g-dev \
	vim procps wget curl

ENV VERSION_PCRE=pcre-8.38
ENV VERSION_OPENSSL=openssl-1.0.2d
ENV VERSION_NGINX=nginx-1.7.12
ENV SOURCE_OPENSSL=https://www.openssl.org/source/
ENV SOURCE_PCRE=ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/
ENV SOURCE_NGINX=http://nginx.org/download/

RUN BP=$(pwd) \
	&& wget -P ./build $SOURCE_PCRE$VERSION_PCRE.tar.gz \
	&& wget -P ./build $SOURCE_OPENSSL$VERSION_OPENSSL.tar.gz --no-check-certificate \
	&& wget -P ./build $SOURCE_NGINX$VERSION_NGINX.tar.gz \
	&& cd build \
	&& tar xzf $VERSION_NGINX.tar.gz \
	&& tar xzf $VERSION_OPENSSL.tar.gz \
	&& tar xzf $VERSION_PCRE.tar.gz \
	# build static openssl
	&& cd $VERSION_OPENSSL \
	&& mkdir staticlibssl \
	&& make clean \
	&& ./config no-shared \
	&& make depend \
	&& make \
	&& make install_sw \
	&& cd $BP/build/$VERSION_NGINX \
	&& ./configure --with-pcre=$BP/build/$VERSION_PCRE \
		--with-http_ssl_module \
		--with-openssl=$BP/build/$VERSION_OPENSSL \
		--sbin-path=/usr/sbin/nginx \
		--conf-path=/etc/nginx/nginx.conf \
		--pid-path=/var/run/nginx.pid \
		--error-log-path=/var/log/nginx/error.log \
		--http-log-path=/var/log/nginx/access.log \
		--with-http_gzip_static_module \
		--with-http_stub_status_module \
		--with-ipv6 \
		--with-file-aio \
	&& make install \
	&& rm -rf $BP/build \
	# forward request and error logs to docker log collector
	&& mkdir -p /var/log/nginx \
	&& ln -sf /dev/stdout /var/log/nginx/access.log \
	&& ln -sf /dev/stderr /var/log/nginx/error.log

# Copy nginx config
#COPY nginx.conf /etc/nginx/nginx.conf
COPY default.conf /etc/nginx/conf.d/default.conf

# END Install Nginx

# Install PHP
RUN apt-get update && apt-get install -y \
        libfreetype6-dev \
        libjpeg62-turbo-dev \
        libmcrypt-dev \
        libpng-dev \
        libxslt1-dev \
        libicu-dev \
        zlib1g-dev \
        curl \
        git \
    && docker-php-ext-install -j$(nproc) mcrypt xsl intl pdo_mysql soap zip opcache bcmath \
    && docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
    && docker-php-ext-install -j$(nproc) gd

# Clean Install
RUN apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Install composer
RUN curl https://getcomposer.org/installer -o composer-setup.php \
    && php composer-setup.php --install-dir=/usr/local/bin/ --filename=composer \
    && php -r "unlink('composer-setup.php');"

# Mailhog
ENV PATH /usr/local/go/bin:$PATH
RUN curl -Lsf 'https://storage.googleapis.com/golang/go1.8.3.linux-amd64.tar.gz' | tar -C '/usr/local' -xvzf - \
	&& go get github.com/mailhog/mhsendmail \
	&& cp /root/go/bin/mhsendmail /usr/bin/mhsendmail \
	&& echo 'sendmail_path = /usr/bin/mhsendmail --smtp-addr mailhog:1025' > /usr/local/etc/php/php.ini \
	&& echo 'memory_limit = -1' >> /usr/local/etc/php/php.ini

# SSL
COPY server.crt /etc/ssl/certs/ssl-cert-snakeoil.pem
COPY server.key /etc/ssl/private/ssl-cert-snakeoil.key

# Expose ports
EXPOSE 80 443

# Run server script
COPY entrypoint /usr/local/bin/
# Fix permission denied of execute files
RUN chmod +x /usr/local/bin/docker-php-entrypoint \
	&& chmod +x /usr/local/bin/entrypoint


WORKDIR /var/www/html

STOPSIGNAL SIGTERM

CMD ["entrypoint"]
