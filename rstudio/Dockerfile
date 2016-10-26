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
RUN apk --no-cache add cmake fakeroot libuuid util-linux-dev libbz2 zlib-dev pango-dev libxslt-dev unzip boost-dev linux-pam-dev

RUN ln -s /usr/local/lib/R/lib/libRblas.so /usr/local/lib/R/lib/libR.so
RUN wget -O rstudio.tar.gz https://github.com/rstudio/rstudio/archive/v0.99.903.tar.gz \
  && tar -xvf rstudio.tar.gz \
  && cd rstudio-0.99.903 \
  && cd dependencies/common \
  && ./install-common \
  && mkdir build \
  && cd build \
  && cmake .. -DRSTUDIO_TARGET=Server -DCMAKE_BUILD_TYPE=Release 

