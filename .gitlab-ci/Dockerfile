FROM rewardenv/php-fpm:7.4-magento2-web-debian

ARG GITHUB_USER
ARG GITHUB_TOKEN
ARG BITBUCKET_PUBLIC_KEY
ARG BITBUCKET_PRIVATE_KEY
ARG MAGENTO_PUBLIC_KEY
ARG MAGENTO_PRIVATE_KEY
ARG MAGENTO_LANGUAGES
# It's good to use but pre 2.2 this flag is not implemented.
ARG MAGENTO_STATIC_CONTENT_DEPLOY_FORCE=true
ARG MAGENTO_DEV_STATIC_SIGN=true
ARG MAGENTO_DEV_JS_MERGE=true
ARG MAGENTO_DEV_JS_MINIFY=true
ARG MAGENTO_DEV_JS_BUNDLING=false
ARG MAGENTO_DEV_CSS_MERGE=true
ARG MAGENTO_DEV_CSS_MINIFY=true
ARG MAGENTO_DEV_HTML_MINIFY=false

USER www-data

COPY --chown=www-data:www-data . .

RUN set -eux \
    && composer global config http-basic.repo.magento.com ${MAGENTO_PUBLIC_KEY} ${MAGENTO_PRIVATE_KEY} \
    && composer global config http-basic.github.com ${GITHUB_USER} ${GITHUB_TOKEN} \
    && composer global config bitbucket-oauth.bitbucket.org ${BITBUCKET_PUBLIC_KEY} ${BITBUCKET_PRIVATE_KEY} \
    && composer config --no-plugins allow-plugins.magento/* true \
    && composer config --no-plugins allow-plugins.laminas/laminas-dependency-plugin true \
    && composer config --no-plugins allow-plugins.dealerdirect/phpcodesniffer-composer-installer true \
    && composer config --no-plugins allow-plugins.cweagans/composer-patches true \
    && composer install --no-interaction --no-progress \
    && composer clear-cache \
    && rm -f /var/www/html/app/etc/env.php \
    && bin/magento setup:di:compile \
    && composer dump-autoload -o \
    && cp -a /home/www-data/.config/composer/auth.json /var/www/html/var \
    && mkdir -p /var/www/html/var/composer_home  \
    && cp -a /home/www-data/.config/composer/composer.json /var/www/html/var/composer_home/ \
    # fix: https://github.com/magento/magento2/issues/33802
    && rm -f /var/www/html/app/etc/env.php \
    && mkdir -p /var/www/html/pub/static \
    && printf "<?php\nprintf(\"php-version: %%g </br>\", phpversion());\nprintf(\"build-date: $(date '+%Y/%m/%d %H:%M:%S')\");\n?>\n" > pub/version.php
#    && bin/magento app:config:import --no-interaction \
#    && bin/magento deploy:mode:set production --skip-compilation \
#    && if [ "${MAGENTO_STATIC_CONTENT_DEPLOY_FORCE}" = "true" ]; \
#      then bin/magento setup:static-content:deploy --jobs=$(nproc) -fv ${MAGENTO_LANGUAGES:-}; \
#      else bin/magento setup:static-content:deploy --jobs=$(nproc) -v ${MAGENTO_LANGUAGES:-}; \
#    fi

# TODO: Remove composer auth file from containers and add Variables in runtime.
