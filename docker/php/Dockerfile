FROM public.ecr.aws/docker/library/php:8.2.10-apache as base

# 講義中の待ち時間を短縮するためインストールするextensionを大幅に省いています
RUN \
apt-get upgrade \
&& apt-get update && apt-get install -y \
    curl \
    unzip \
    libzip-dev \
&& apt-get clean \
&& a2enmod remoteip \
&& a2enmod rewrite \
&& rm -rf /var/lib/apt/lists/* \
&& docker-php-ext-install -j$(nproc) \
    zip \
&& docker-php-source delete

# composerをインストール
COPY --from=public.ecr.aws/docker/library/composer:2.5.8 /usr/bin/composer /usr/local/bin/composer

# apacheの設定ファイルをコピー
COPY  ./docker/php/000-default.conf /etc/apache2/sites-available/000-default.conf

WORKDIR /var/www/laravel

FROM base as local

RUN groupadd -g 1000 user && \
    useradd -m -s /bin/bash -u 1000 -g 1000 user
USER user


# ECRの練習用
FROM base as ecr

COPY ./src/ /var/www/laravel/
RUN chmod -R 777 /var/www/laravel/storage/ /var/www/laravel/bootstrap/cache




# ECSの練習用
FROM base as ecs

COPY ./src/ /var/www/laravel/
RUN cd /var/www/laravel && composer install
RUN chmod -R 777 /var/www/laravel/storage/ /var/www/laravel/bootstrap/cache

COPY ./docker/php/docker-entrypoint.sh /
RUN chmod +x /docker-entrypoint.sh
ENTRYPOINT ["/docker-entrypoint.sh"]
CMD ["apache2-foreground"]