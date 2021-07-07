ARG NODE_VERSION=12.18.3
FROM node:${NODE_VERSION}-alpine
RUN apk add --no-cache make pkgconfig gcc g++ python libx11-dev libxkbfile-dev
ARG version=latest
WORKDIR /home/theia
ADD $version.package.json ./package.json
ARG GITHUB_TOKEN
RUN yarn --pure-lockfile && \
    NODE_OPTIONS="--max_old_space_size=4096" yarn theia build && \
    yarn theia download:plugins && \
    yarn --production && \
    yarn autoclean --init && \
    echo *.ts >> .yarnclean && \
    echo *.ts.map >> .yarnclean && \
    echo *.spec.* >> .yarnclean && \
    yarn autoclean --force && \
    yarn cache clean

FROM node:${NODE_VERSION}-alpine
# See : https://github.com/theia-ide/theia-apps/issues/34
RUN apk add --no-cache shadow
RUN usermod --uid 1001 node && \
    groupmod -g 1001 node && \
    addgroup -g 1000 ubuntu && \
    addgroup -g 998 docker && \
    adduser --uid 1000 -G ubuntu -s /bin/sh -D ubuntu && \
    usermod -aG docker ubuntu
RUN chmod g+rw /home && \
    mkdir -p /home/project /home/theia && \
    chown -R ubuntu:ubuntu /home/theia && \
    chown -R ubuntu:ubuntu /home/project;
RUN apk add --no-cache git openssh bash git-crypt docker docker-compose
ENV HOME /home/theia
WORKDIR /home/theia
COPY --from=0 --chown=ubuntu:ubuntu /home/theia /home/theia
EXPOSE 3000
ENV SHELL=/bin/bash \
    THEIA_DEFAULT_PLUGINS=local-dir:/home/theia/plugins
ENV USE_LOCAL_GIT true
USER ubuntu
ENTRYPOINT [ "node", "/home/theia/src-gen/backend/main.js", "/home/project", "--hostname=0.0.0.0" ]
