FROM php:8.1-fpm
MAINTAINER cnx7 "zysafe@live.cn"

# 设置时区
ENV TZ=Asia/Shanghai
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone


RUN echo "deb https://mirrors.tuna.tsinghua.edu.cn/debian/ buster main" > /etc/apt/sources.list \
  && echo "deb https://mirrors.tuna.tsinghua.edu.cn/debian/ buster-updates main" >> /etc/apt/sources.list \
  && echo "deb https://mirrors.tuna.tsinghua.edu.cn/debian-security/ buster/updates main" >> /etc/apt/sources.list

RUN sed -i 's/mirrors.aliyun.com/mirrors.tuna.tsinghua.edu.cn/g' /etc/apt/sources.list
RUN cat /etc/apt/sources.list

# 更新安装依赖包
RUN apt-get update  && apt-get install  -y \
        git \
        vim \
        curl \
        wget \
        libfreetype6-dev \
        libjpeg62-turbo-dev \
        libpng-dev \
        libwebp-dev \
        libjpeg-dev \
        libxml2-dev \
		libssl-dev \
        libzip-dev \
        librabbitmq-dev \
        unzip

# 安装PHP核心拓展
RUN docker-php-ext-configure gd --with-webp=/usr/include/webp --with-jpeg=/usr/include --with-freetype=/usr/include/freetype2/ \
	&& docker-php-ext-install gd zip pdo_mysql opcache mysqli bcmath sockets soap pcntl

# 安装PECL扩展
RUN pecl install -o -f   redis \
    && docker-php-ext-enable   redis

# 安装 Swoole
RUN yes|pecl install -o -f swoole && docker-php-ext-enable swoole


# 安装 Composer
ENV COMPOSER_HOME /root/composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer
ENV PATH $COMPOSER_HOME/vendor/bin:$PATH
# 更换Composer为腾讯源
RUN composer config -g repo.packagist composer https://mirrors.cloud.tencent.com/composer/

# 清理镜像
RUN rm -rf /var/lib/apt/lists/* && rm -rf /tmp/pear

WORKDIR /data/www

# 写权限
RUN usermod -u 1000 www-data

EXPOSE 9000