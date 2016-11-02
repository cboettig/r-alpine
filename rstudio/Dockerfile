FROM cboettig/r-alpine:latest

## Add RStudio binaries to PATH
ENV PATH /usr/lib/rstudio-server/bin/:$PATH

## RStudio runtime dependencies
RUN apk --no-cache add \ 
    ca-certificates \
    file \
    git \
    openssl \
    sudo \
  && update-ca-certificates \ 
## Install RStudio build deps & compile RStudio from source
  && apk --no-cache add --virtual build-deps \
    apache-ant \
#    boost-dev \
    cmake \
    fakeroot \
    libuuid \
    libbz2 \
    libxslt-dev \
    linux-headers \
    linux-pam-dev \
    openjdk8 \
    pango-dev \ 
    python-dev \
    unzip \
    util-linux-dev \
    zlib-dev \
#  && wget -q -O /etc/apk/keys/sgerrand.rsa.pub https://raw.githubusercontent.com/sgerrand/alpine-pkg-glibc/master/sgerrand.rsa.pub \
#  && wget https://github.com/sgerrand/alpine-pkg-glibc/releases/download/2.23-r3/glibc-2.23-r3.apk \
#  && apk add glibc-2.23-r3.apk \
   && apk add libexecinfo-dev --update-cache --repository http://dl-cdn.alpinelinux.org/alpine/edge/main --allow-untrusted \
#  && apk add 'boost-dev<1.59' --update-cache --repository http://dl-cdn.alpinelinux.org/alpine/v3.1/main --allow-untrusted \
  && ln -s /usr/lib/R/lib/libRblas.so /usr/lib/R/lib/libR.so \
  && wget -O rstudio.tar.gz https://github.com/rstudio/rstudio/archive/v0.99.903.tar.gz \
  && tar -xvf rstudio.tar.gz \
  && cd /rstudio-0.99.903/dependencies/common \ 
  && ./install-gwt \
  && ./install-dictionaries \
  && ./install-mathjax \
  && ./install-pandoc \
  && ./install-libclang \
  && ./install-packages \
  && ./install-boost || true \
  && mkdir /rstudio-0.99.903/build \
  && cd /rstudio-0.99.903/build \
  && cmake .. -DRSTUDIO_TARGET=Server -DCMAKE_BUILD_TYPE=Release 

## Currently this runs into strange problems complining RStudio's C++ files....
## When fixed, combine with previous & following RUN commands for a single layer.
RUN cd /rstudio-0.99.903/build \
  && make install

# Clean up from source install 
RUN apk del --purge build-dep \
  && rm -rf /rstudio-0.99.903 \
  && rm /glibc-*.apk rstudio-*.tar.gz  

############ Additional setup #########


## Set up RStudio server, s6 init system, etc
RUN adduser -S rstudio-server \
## Link stand-alone pandoc & templates
  && ln -s /usr/lib/rstudio-server/bin/pandoc/pandoc /usr/local/bin \
  && ln -s /usr/lib/rstudio-server/bin/pandoc/pandoc-citeproc /usr/local/bin \
  && wget https://github.com/jgm/pandoc-templates/archive/1.15.0.6.tar.gz \
  && mkdir -p /opt/pandoc/templates && tar zxf 1.15.0.6.tar.gz \
  && cp -r pandoc-templates*/* /opt/pandoc/templates && rm -rf pandoc-templates* \
  && mkdir /root/.pandoc && ln -s /opt/pandoc/templates /root/.pandoc/templates \
## Ensure that if both httr and httpuv are installed downstream, oauth 2.0 flows still work correctly.
  && echo '\n\
\n# Configure httr to perform out-of-band authentication if HTTR_LOCALHOST \
\n# is not set since a redirect to localhost may not work depending upon \
\n# where this Docker container is running. \
\nif(is.na(Sys.getenv("HTTR_LOCALHOST", unset=NA))) { \
\n  options(httr_oob_default = TRUE) \
\n}' >> /etc/R/Rprofile.site \
  && echo "PATH=\"/usr/lib/rstudio-server/bin/:\${PATH}\"" >> /etc/R/Renviron.site \
## A default user system configuration. 
  && adduser -S rstudio \
  && echo "rstudio:rstudio" | chpasswd \
## Use s6
  && wget -P /tmp/ https://github.com/just-containers/s6-overlay/releases/download/v1.11.0.1/s6-overlay-amd64.tar.gz \
  && tar xzf /tmp/s6-overlay-amd64.tar.gz -C / \
  && echo 'exec /usr/lib/rstudio-server/bin/rserver --server-daemonize 0' >> /etc/services.d/rstudio/run

## Configure user based on runtime ENV
COPY userconf.sh /etc/cont-init.d/conf

EXPOSE 8787
CMD ["/init"]
