FROM artemklevtsov/r-alpine:devel

## Add RStudio binaries to PATH
ENV PATH /usr/lib/rstudio-server/bin/:$PATH
RUN apk --no-cache add \ 
    ca-certificates \
    file \
    git \
    openssl \
    sudo \
    && update-ca-certificates

## RStudio build deps
RUN apk --no-cache add \
  apache-ant \
  boost-dev \
  cmake \
  fakeroot \
  libuuid \
  libbz2 \
  libxslt-dev \
  linux-pam-dev \
  openjdk8 \
  pango-dev \ 
  unzip \
  util-linux-dev \
  zlib-dev \
  && wget -q -O /etc/apk/keys/sgerrand.rsa.pub https://raw.githubusercontent.com/sgerrand/alpine-pkg-glibc/master/sgerrand.rsa.pub \
  && wget https://github.com/sgerrand/alpine-pkg-glibc/releases/download/2.23-r3/glibc-2.23-r3.apk \
  && apk add glibc-2.23-r3.apk


RUN ln -s /usr/lib/R/lib/libRblas.so /usr/lib/R/lib/libR.so \
  && wget -O rstudio.tar.gz https://github.com/rstudio/rstudio/archive/v0.99.903.tar.gz \
  && tar -xvf rstudio.tar.gz
  && cd /rstudio-0.99.903/dependencies/common \ 
  && ./install-gwt \
  && ./install-dictionaries \
  && ./install-mathjax \
#  && ./install-boost \
  && ./install-pandoc \
  && ./install-libclang \
  && ./install-packages \
#  && ./install-common
  && mkdir /rstudio-0.99.903/build \
  && cd /rstudio-0.99.903/build \
  && cmake .. -DRSTUDIO_TARGET=Server -DCMAKE_BUILD_TYPE=Release \
  && make install
