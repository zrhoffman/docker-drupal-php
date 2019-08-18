FROM alpine:edge

LABEL name=zrhoffman/drupal-php:fpm-prod version=0.0.0

ENV TIMEZONE=America/Denver

RUN set -o errexit -o nounset -o pipefail;\
\
    #necessary to avoid https://github.com/Yelp/dumb-init/issues/73
    apk upgrade --no-cache;\
\
    #Avoids
    #`WARNING: Ignoring APKINDEX.b53994b4.tar.gz: No such file or directory`
    #https://github.com/gliderlabs/docker-alpine/issues/207#issuecomment-241308729
    apk update;\
\
    apk add --no-cache\
        php7-fpm\
\
        #for composer
        php7-phar\
        php7-json\
        php7-openssl\
        php7-mbstring\
\
        #for drush/drush
        php7-dom\
        php7-tokenizer\
\
        #for hirak/prestissimo
        php7-curl\
\
        #for `composer create-project`
        git\
\
        #for drupal/core
        php7-gd\
        php7-pdo_pgsql\
        php7-pdo_sqlite\
        php7-session\
        php7-simplexml\
        php7-xml\
        php7-xmlwriter\
\
        #for symfony
        php7-apcu\
\
        #Other extensions
        php7-opcache\
\
        #Non-PHP dependency for drush/drush (sql:cli)
        postgresql-client\
\
        #PHP extensions for for contrib modules:
\
        #for memcache
        php7-memcached\
\
        #for maserminds/html5
        php7-ctype\
\
        #for typo3/phar-stream-wrapper
        php7-fileinfo\
\
        #Non-PHP dependency for cweagans/composer-patches
        patch\
    ;\
\
    #To let us set the timezone. The package name is tzdata
    apk add --no-cache --virtual timezones tzdata;\
\
    #Set the timezone
    cp /usr/share/zoneinfo/"$TIMEZONE" /etc/localtime;\
    echo "$TIMEZONE" > /etc/timezone;\
    apk del timezones;\
\
    #Install composer
    wget -O - 'https://getcomposer.org/installer' |\
        php -- --install-dir=/usr/local/bin --filename=composer;\
\
    {\
        echo 'alias drupal=./vendor/bin/drupal;';\
        echo 'alias drush=./vendor/bin/drush;';\
    }  >> /etc/profile;\
\
    adduser\
        --disabled-password\
        --shell=/bin/sh\
        --uid=1000\
        php\
    ;\
\
    addgroup --gid=82 www-data;\
\
    #Add php to the www-data group so we can share the FPM socket with the user
    #running nginx in the web container
    addgroup php www-data;\
\
    #"global" will use /home/php/.composer/ as the install directory
    su -c 'composer global require --update-no-dev -- hirak/prestissimo; composer clear-cache' php;\
\
    mkdir -p\
        #Project files will be mounted here
        /app\
\
        #PHP-FPM socket volume goes here
        /var/run/php;\
\
    chown php:www-data /app /var/run/php;

COPY ./etc /etc

CMD php-fpm7;
