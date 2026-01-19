FROM php:8.3-fpm
LABEL maintainer="tanghang <474352380@qq.com>"

# 设置时区
ENV TZ=Asia/Shanghai
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

# 配置国内镜像源 (使用阿里云)
RUN sed -i 's/deb.debian.org/mirrors.aliyun.com/g' /etc/apt/sources.list.d/debian.sources

# 更新安装依赖包
RUN apt-get update && apt-get install -y --no-install-recommends \
        git \
        vim \
        curl \
        wget \
        lsof \
        procps \
        libfreetype6-dev \
        libjpeg62-turbo-dev \
        libpng-dev \
        libwebp-dev \
        libjpeg-dev \
        libxml2-dev \
        libssl-dev \
        libzip-dev \
        librabbitmq-dev \
        unzip \
        libpq-dev \
        unixodbc-dev \
        unixodbc \
        libcurl4-openssl-dev \
        libc-ares-dev \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# 安装PHP核心拓展
RUN docker-php-ext-configure gd --with-webp --with-jpeg --with-freetype \
    && docker-php-ext-install -j$(nproc) gd zip pdo_mysql opcache mysqli bcmath sockets soap pcntl

# 安装PECL扩展 - Redis
RUN pecl install -o -f redis \
    && docker-php-ext-enable redis

# 安装 Swoole 5.x (支持 Hyperf 3.x)
RUN pecl install -o -f swoole \
    && docker-php-ext-enable swoole

# 安装 Composer
ENV COMPOSER_HOME=/root/composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer
ENV PATH=$COMPOSER_HOME/vendor/bin:$PATH
# 更换Composer为腾讯源
RUN composer config -g repo.packagist composer https://mirrors.cloud.tencent.com/composer/

WORKDIR /data/www

# 写权限
RUN usermod -u 1000 www-data

EXPOSE 9000 9501-9510