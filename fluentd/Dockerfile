FROM fluent/fluentd:v1.1
LABEL maintainer jaemyunlee


RUN apk add --update --virtual .build-deps \
        sudo build-base ruby-dev \
 && sudo gem install \
        fluent-plugin-elasticsearch --no-rdoc --no-ri --version 2.9.2\
 && sudo gem install \
        fluent-plugin-concat --no-rdoc --no-ri --version 2.2.2\
 && sudo gem sources --clear-all \
 && apk del .build-deps \
 && rm -rf /var/cache/apk/* \
           /home/fluent/.gem/ruby/2.3.0/cache/*.gem
