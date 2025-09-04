# Base de construção
FROM node:22-bullseye AS builder
LABEL maintainer="Systemize"

ENV CRONICLE_VERSION=0.9.91
WORKDIR /opt/cronicle

# Baixa e instala o Cronicle com dependências
RUN curl -L -o /tmp/Cronicle-${CRONICLE_VERSION}.tar.gz https://github.com/jhuckaby/Cronicle/archive/refs/tags/v${CRONICLE_VERSION}.tar.gz \
    && tar zxvf /tmp/Cronicle-${CRONICLE_VERSION}.tar.gz -C /tmp/ \
    && mv /tmp/Cronicle-${CRONICLE_VERSION}/* . \
    && rm -rf /tmp/* \
    && npm install

# Aplica patches customizados
COPY ./patches /tmp/patches
RUN patch -p3 < /tmp/patches/engine.patch lib/engine.js

# Copia o script de entrada
COPY docker-entrypoint.sh ./bin/

# Base de execução
FROM docker:27

# Metadados e configurações padrão
LABEL maintainer="Systemize"
ENV CRONICLE_base_app_url='http://localhost:3012'
ENV CRONICLE_WebServer__http_port=3012
ENV CRONICLE_WebServer__https_port=443
ENV EDITOR=nano
ENV CRONICLE_foreground=1
ENV CRONICLE_echo=1
ENV CRONICLE_color=1
ENV debug_level=1
ENV HOSTNAME=main

# Instala dependências necessárias
RUN apk add --no-cache --upgrade \
    nodejs npm git curl perl bash perl-pathtools tar procps nano tini python3

# Copia o Cronicle do estágio builder
COPY --from=builder /opt/cronicle/ /opt/cronicle/
WORKDIR /opt/cronicle/

# Configura o Cronicle
RUN node bin/build.js dist \
    && bin/control.sh setup \
    && rm -Rf /root/.npm

# Configuração de volumes para persistência de dados
VOLUME ["/opt/cronicle/data", "/opt/cronicle/logs", "/opt/cronicle/plugins"]

# Exposição de portas
EXPOSE 3012

# Configurações de entrada
COPY docker-entrypoint.sh /entrypoint.sh
ENTRYPOINT ["/sbin/tini", "--"]
CMD ["sh", "/entrypoint.sh"]
