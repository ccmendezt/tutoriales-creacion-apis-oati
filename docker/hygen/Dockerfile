# Se hace uso de una imagen base de Ubuntu 22.04
FROM ubuntu:22.04

# Actualizamos los repositorios
RUN apt-get update

# Actualizamos el sistema
RUN apt-get upgrade -y

# Herramientas del sistema

RUN apt-get install -y curl python2.7 python3 git tar

# NVM (NODE VERSION MANAGER)(NODE_VERSION V20.10.0)
ENV NVM_DIR /usr/local/nvm
RUN curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.1/install.sh | bash
ENV NODE_VERSION v20.10.0
RUN /bin/bash -c "source $NVM_DIR/nvm.sh && nvm install $NODE_VERSION && nvm use --delete-prefix $NODE_VERSION"
ENV NODE_PATH $NVM_DIR/versions/node/$NODE_VERSION/lib/node_modules
ENV PATH      $NVM_DIR/versions/node/$NODE_VERSION/bin:$PATH

# GOLANG

# Cambiamos al directorio /tmp
WORKDIR /tmp

# Descargamos el archivo y le damos un nombre de destino
RUN curl -OL https://golang.org/dl/go1.21.5.linux-amd64.tar.gz
RUN tar -C /usr/local -xzf go1.21.5.linux-amd64.tar.gz
RUN rm -f go1.21.5.linux-amd64.tar.gz

ENV GOROOT /usr/local/go
ENV GOPATH $HOME/go
ENV PATH $PATH:/usr/local/go/bin:$HOME/go/bin
RUN mkdir -p $GOPATH/src $GOPATH/bin \
    && mkdir -p $GOPATH/src/github.com/udistrital/

# instalacion de beego y bee
RUN mkdir -p $GOPATH/src/github.com/beego \
    && cd $GOPATH/src/github.com/beego \
    && git clone --depth=1 --branch=v1.12.3 https://github.com/beego/bee.git \
    && git clone --depth=1 --branch=v1.12.3 https://github.com/beego/beego.git \
    && cd bee && go install \
    && cd ../beego && go install

# Instalación de golang-golang-x-tools
RUN apt-get install -y golang-golang-x-tools

# HYGEN
WORKDIR $GOPATH/src/github.com/udistrital/
RUN npm i -g hygen && hygen init self
RUN cd ./_templates/ \ 
    && git clone https://github.com/udistrital/plantilla_api_crud.git \
    && git clone https://github.com/udistrital/plantilla_api_mid.git \
    && git clone https://github.com/udistrital/plantillas_cliente.git

# Comando para mantener el contenedor en ejecucion de forma indefinida
CMD ["tail", "-f", "/dev/null"]