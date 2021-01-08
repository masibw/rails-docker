FROM node:14.15.3-alpine as node

# 依存関係のインストールを行うだけのもの
FROM ruby:2.6.0-alpine as builder
RUN apk --update --no-cache add --virtual build-dependencies \
 shadow=4.5-r0 sudo=1.8.23-r4 busybox-suid=1.28.4-r3 mariadb-connector-c-dev=3.0.4-r1 tzdata=2020a-r0 alpine-sdk=1.0-r0

WORKDIR /rails

COPY --from=node /usr/local/bin/node /usr/local/bin/node
COPY --from=node /usr/local/include/node /usr/local/include/node
COPY --from=node /usr/local/lib/node_modules /usr/local/lib/node_modules
COPY --from=node /opt/yarn-* /opt/yarn
RUN ln -s /usr/local/bin/node /usr/local/bin/nodejs && \
    ln -s /usr/local/lib/node_modules/npm/bin/npm-cli.js /usr/local/bin/npm && \
    ln -s /opt/yarn/bin/yarn /usr/local/bin/yarn

# 依存しているライブラリのインストール
COPY Gemfile Gemfile.lock ./
ENV BUNDLE_JOBS=4
RUN gem install bundler -v 2.1.4 \
  && bundle install --without development test

#yarn install
COPY package.json yarn.lock ./
RUN yarn install

# assets precompile
COPY Rakefile Rakefile
COPY app/javascript app/javascript
COPY app/assets app/assets
COPY bin bin
COPY config config
RUN RAILS_ENV=production bundle exec rails assets:precompile

RUN apk del build-dependencies

# 実際にRailsを動作させるもの
FROM ruby:2.6.0-alpine
ENV DOCKERIZE_VERSION v0.6.1

# パッケージ全体を軽量化して、railsが起動する最低限のものにする
RUN apk --update --no-cache -q add shadow=4.5-r0 sudo=1.8.23-r4 busybox-suid=1.28.4-r3 execline=2.5.0.0-r0 tzdata=2020a-r0 mariadb-connector-c-dev=3.0.4-r1 libstdc++=6.4.0-r9 && \
    cp /usr/share/zoneinfo/Asia/Tokyo /etc/localtime

RUN mkdir /myapp
WORKDIR /myapp

# gemやassets:precompileの終わったファイルはbuilderからコピーしてくる
COPY --from=builder /usr/local/bundle /usr/local/bundle

COPY --from=builder /rails/public/assets/ /myapp/public/assets/
COPY --from=builder /rails/public/packs/ /myapp/public/packs/

## Railsはディレクトリを自動生成してくれないのでunix domain socket通信に必要なファイルを作っておく
RUN mkdir /myapp/tmp
RUN mkdir /myapp/tmp/pids
RUN mkdir /myapp/tmp/sockets
