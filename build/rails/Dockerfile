FROM node:14.15.3-alpine as node

# 依存関係のインストールを行うだけのもの
FROM ruby:2.6.0-alpine as builder
RUN apk --update --no-cache add --virtual build-dependencies \
 shadow sudo busybox-suid mariadb-connector-c-dev tzdata alpine-sdk

WORKDIR /rails

COPY --from=node /usr/local/bin/node /usr/local/bin/node
COPY --from=node /usr/local/include/node /usr/local/include/node
COPY --from=node /usr/local/lib/node_modules /usr/local/lib/node_modules
COPY --from=node /opt/yarn-* /opt/yarn
RUN ln -s /usr/local/bin/node /usr/local/bin/nodejs && \
    ln -s /usr/local/lib/node_modules/npm/bin/npm-cli.js /usr/local/bin/npm && \
    ln -s /opt/yarn/bin/yarn /usr/local/bin/yarn

# 依存しているライブラリのインストール
ADD Gemfile Gemfile.lock ./
ENV BUNDLE_JOBS=4
RUN gem install bundler -v 2.1.4 \
  && bundle install --without development test

#yarn install
ADD package.json yarn.lock ./
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
RUN apk --update --no-cache -q add shadow sudo busybox-suid execline tzdata mariadb-connector-c-dev libstdc++ && \
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
