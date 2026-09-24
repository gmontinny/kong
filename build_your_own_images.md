# Construindo imagens Docker que contêm o Kong Gateway

A Kong Software usa o [repositório github docker-kong](https://github.com/Kong/docker-kong/)
para construir imagens Docker que contêm o Kong Gateway. Usaremos o
[docker-kong](https://github.com/Kong/docker-kong/) como implementação de referência
para descrever como construir suas próprias imagens Docker que contêm o Kong Gateway.

Não fornecemos um Dockerfile com o argumento `FROM` parametrizado para permitir o uso de sua
própria imagem base, pois isso prejudica nossa capacidade de ter imagens públicas
aceitas rapidamente pelo Dockerhub. Se desejar, você pode clonar o [repositório github docker-kong](https://github.com/Kong/docker-kong/)
e ajustar o Dockerfile para o tipo de pacote desejado para usar sua imagem base e versão de pacote
desejadas, depois usar o target `build_v2` no nosso `Makefile` para construir sua imagem. Este
documento adota a abordagem de percorrer o conteúdo dos Dockerfiles
para que você possa criar e manter os seus próprios.

Para construir sua imagem Docker, você precisará fornecer:

1. Uma imagem base de sua escolha
1. Um script de entrypoint que executa o Kong Gateway
1. Um Dockerfile que instala o Kong Gateway a partir de um local que você especifica

## Imagem base
Você pode usar imagens derivadas de RHEL ou Ubuntu; a Kong Software publica pacotes `.deb` e
`.rpm` em nosso [repositório público de pacotes](https://packages.konghq.com/).

## Script de entrypoint

Obtenha o [script de entrypoint](https://raw.githubusercontent.com/Kong/docker-kong/master/docker-entrypoint.sh)
do [repositório github docker-kong](https://github.com/Kong/docker-kong/) e coloque-o no
diretório onde você planeja executar o comando para construir sua imagem Docker.

## Crie um Dockerfile para instalar o Kong Gateway

### Decida como obter o pacote do Kong Gateway
A Kong Software fornece pacotes `.deb` e `.rpm` via nosso [repositório público de pacotes](https://packages.konghq.com/).
Decida se você quer que seu Dockerfile:

1. Baixe o pacote desejado de https://packages.konghq.com, ou
2. Baixe o pacote desejado de outro repositório de pacotes que você especificar, ou
3. Instale o pacote desejado localmente a partir do disco.

Se você escolher 1 ou 2, execute o comando `touch kong.rpm` no diretório para o qual
seu Dockerfile fará o download do arquivo; isso garante que o arquivo baixado
terá o usuário, grupos e permissões corretos.

Se você escolher 2 ou 3, baixe o pacote que deseja instalar e coloque-o
no local desejado.

### Escreva um Dockerfile para instalar o pacote do Kong Gateway
Use o template abaixo para criar seu Dockerfile. Colchetes angulares (`<>`) indicam
valores que você precisa fornecer. Comentários que começam com "# Descomente" indicam que
você precisa descomentar as linhas relevantes ao seu contexto.

O template é baseado nos Dockerfiles do [repositório github docker-kong](https://github.com/Kong/docker-kong/)
e criado manualmente. Verifique os Dockerfiles para alterações.

```
FROM <sua-imagem-base>

ARG KONG_VERSION=<versao-do-Kong-Gateway>=
ENV KONG_VERSION $KONG_VERSION

# Descomente a linha ARG KONG_SHA256 para construir um contêiner usando um pacote .deb ou .rpm
# Para pacotes .deb, o SHA está em
# https://cloudsmith.io/~kong/repos/gateway-<gateway-major-version><gateway-minor-version>/packages/detail/deb/kong/<gateway-version>/a=amd64;xc=main;d=debian%252F<os_version>;t=binary/
# Para pacotes .rpm, o SHA está em
# https://cloudsmith.io/~kong/repos/gateway-<gateway-major-version><gateway-minor-version>/packages/detail/rpm/kong/<gateway-version>/a=x86_64;d=el%252F<os_version>;t=binary/
# ARG KONG_SHA256="<SHA-do-.deb-ou-.rpm>"

# Descomente para baixar o pacote de um repositório remoto
# ARG ASSET=remote

# Descomente para instalar o pacote a partir do disco local
# ARG ASSET=local

ARG EE_PORTS

# Descomente se estiver instalando um .rpm
# COPY kong.rpm /tmp/kong.rpm

# Descomente se estiver instalando um .deb
# COPY kong.deb /tmp/kong.deb

# Descomente se estiver instalando um .deb.tar.gz
# COPY kong.deb.tar.gz /tmp/kong.deb.tar.gz

# hadolint ignore=DL3015
# Descomente a seção a seguir se estiver instalando um .rpm
# Edite a linha DOWNLOAD_URL para instalar de um repositório diferente de
# packages.konghq.com
# RUN set -ex; \
#     if [ "$ASSET" = "remote" ] ; then \
#       VERSION=$(grep '^VERSION_ID' /etc/os-release | cut -d = -f 2 | sed -e 's/^"//' -e 's/"$//' | cut -d . -f 1) \
#       && KONG_REPO=$(echo ${KONG_VERSION%.*} | sed 's/\.//') \
#       && DOWNLOAD_URL="https://packages.konghq.com/public/gateway-$KONG_REPO/rpm/el/$VERSION/x86_64/kong-$KONG_VERSION.el$VERSION.x86_64.rpm" \
#       && curl -fL $DOWNLOAD_URL -o /tmp/kong.rpm \
#       && echo "$KONG_SHA256  /tmp/kong.rpm" | sha256sum -c -; \
#     fi \
#     && yum install -y /tmp/kong.rpm \
#     && rm /tmp/kong.rpm \
#     && chown kong:0 /usr/local/bin/kong \
#     && chown -R kong:0 /usr/local/kong \
#     && ln -s /usr/local/openresty/bin/resty /usr/local/bin/resty \
#     && ln -s /usr/local/openresty/luajit/bin/luajit /usr/local/bin/luajit \
#     && ln -s /usr/local/openresty/luajit/bin/luajit /usr/local/bin/lua \
#     && ln -s /usr/local/openresty/nginx/sbin/nginx /usr/local/bin/nginx \
#     && kong version

# Descomente a seção a seguir se estiver instalando um .deb
# Edite a linha DOWNLOAD_URL para instalar de um repositório diferente de
# packages.konghq.com
# RUN set -ex; \
#     apt-get update; \
#     apt-get install -y curl; \
#     if [ "$ASSET" = "remote" ] ; then \
#       CODENAME=$(cat /etc/os-release | grep VERSION_CODENAME | cut -d = -f 2) \
#       && KONG_REPO=$(echo ${KONG_VERSION%.*} | sed 's/\.//') \
#       && DOWNLOAD_URL="https://packages.konghq.com/public/gateway-$KONG_REPO/deb/ubuntu/pool/$CODENAME/main/k/ko/kong_$KONG_VERSION/kong_${KONG_VERSION}_amd64.deb" \
#       && curl -fL $DOWNLOAD_URL -o /tmp/kong.deb \
#       && echo "$KONG_SHA256  /tmp/kong.deb" | sha256sum -c -; \
#     fi \
#     && apt-get update \
#     && apt-get install --yes /tmp/kong.deb \
#     && rm -rf /var/lib/apt/lists/* \
#     && rm -rf /tmp/kong.deb \
#     && chown kong:0 /usr/local/bin/kong \
#     && chown -R kong:0 /usr/local/kong \
#     && ln -s /usr/local/openresty/bin/resty /usr/local/bin/resty \
#     && ln -s /usr/local/openresty/luajit/bin/luajit /usr/local/bin/luajit \
#     && ln -s /usr/local/openresty/luajit/bin/luajit /usr/local/bin/lua \
#     && ln -s /usr/local/openresty/nginx/sbin/nginx /usr/local/bin/nginx \
#     && kong version \
#     && apt-get purge curl -y

COPY docker-entrypoint.sh /docker-entrypoint.sh

USER kong

ENTRYPOINT ["/docker-entrypoint.sh"]

EXPOSE 8000 8443 8001 8444 $EE_PORTS

STOPSIGNAL SIGQUIT

HEALTHCHECK --interval=60s --timeout=10s --retries=10 CMD kong-health

CMD ["kong", "docker-start"]
```

### Execute o comando docker

Execute o comando `docker build --no-cache -t kong-<sua-imagem-base>
<caminho-para-imagem-construida>` para construir a imagem docker.
