FROM node:21.6-bookworm as node-build

COPY ./src /application

WORKDIR /application

RUN apt-get update && \
  npm install && \
  npm run build

# 本番用ビルド
FROM nginx:1.25.4-bookworm

WORKDIR /application

RUN apt-get update

COPY ./.docker/nginx/ecs/conf.d/default.conf /etc/nginx/conf.d/default.conf
COPY --from=node-build /application/public /application/public

EXPOSE 80

