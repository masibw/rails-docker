FROM ruby:2.6.0
ENV DOCKERIZE_VERSION v0.6.1

RUN apt-get update -qq \
  && apt-get install -y apt-transport-https \
  && curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add - \
  && echo "deb https://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list \
  && curl -sL https://deb.nodesource.com/setup_14.x | bash - \
  && apt-get update -qq \
  && apt-get install -y nodejs \
  && apt-get install build-essential libpq-dev yarn imagemagick wget -y \
  && wget https://github.com/jwilder/dockerize/releases/download/$DOCKERIZE_VERSION/dockerize-linux-amd64-$DOCKERIZE_VERSION.tar.gz \
  && tar -C /usr/local/bin -xzvf dockerize-linux-amd64-$DOCKERIZE_VERSION.tar.gz \
  && rm dockerize-linux-amd64-$DOCKERIZE_VERSION.tar.gz

RUN mkdir /myapp
WORKDIR /myapp

# 依存しているライブラリのインストール
ADD Gemfile /myapp/Gemfile
ADD Gemfile.lock /myapp/Gemfile.lock
RUN gem install bundler -v 2.1.4 \
  && bundle install
ADD package.json /myapp/package.json
RUN yarn install

# Railsはディレクトリを自動生成してくれないのでunix domain socket通信に必要なファイルを作っておく
RUN mkdir /myapp/tmp
RUN mkdir /myapp/tmp/pids
RUN mkdir /myapp/tmp/sockets

EXPOSE 3000