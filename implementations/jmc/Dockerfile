FROM ubuntu:24.04
LABEL description="JSON Model Compiler Setup for Benchmarking"

# versions:
ARG nodejs=stable
ARG cre2=master
ARG nre2=
ARG jmc=main
ARG jsu=main

# unversions - the version of these stem from ubuntu:
# - gcc/g++
# - python
# - pcre2
# - re2
# - jansson

# system update and installs
RUN apt update ; \
    apt upgrade -y ; \
    apt install -y --no-install-recommends --fix-missing \
        vim git ca-certificates libtool m4 time automake make pkgconf gcc g++ texinfo jq curl \
        libpcre2-dev libre2-dev libjansson-dev \
        python3 python-is-python3 python3-venv python3-pip ; \
    apt clean

# compile cre2 into /usr/local
RUN git clone -b $cre2 https://github.com/marcomaggi/cre2.git
RUN cd cre2 ; \
    sh autogen.sh ; \
    ./configure --enable-maintainer-mode ; \
    make ; \
    make install

# setup python environment, native pip complains about direct installations otherwise
RUN python -m venv /venv
RUN /venv/bin/pip install -U pip

# application
RUN mkdir /app
WORKDIR /app

# nodejs >= 20.0.0
ENV NVM_DIR=/usr/local/nvm
RUN mkdir -p $NVM_DIR
RUN curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh | bash
RUN . $NVM_DIR/nvm.sh && \
	nvm install $nodejs && \
    nvm alias default $nodejs && \
    nvm use default
RUN ln -s $NVM_DIR/versions/node/v* $NVM_DIR/versions/node/default
ENV NODE_PATH=/app/node_modules
ENV PATH=$NVM_DIR/versions/node/default/bin:$PATH

# js runtime
RUN echo '{"name":"jmc-benchmark","type":"module"}' | jq > package.json
RUN npm install re2${nre2:+@$nre2}

# actual application stuff
# direct install json-model and json-schema-utils from github
RUN git clone -b $jmc https://github.com/clairey-zx81/json-model.git
RUN git clone -b $jsu https://github.com/zx80/json-schema-utils.git

RUN /venv/bin/pip install -e ./json-model ./json-schema-utils
RUN npm install file:./json-model/json_model/runtime/js

# setup benchmarking scripts
COPY . /app
ENV LD_LIBRARY_PATH=/usr/local/lib
ENTRYPOINT ["/app/memory-wrapper.sh", "/app/generate-and-run.sh"]
