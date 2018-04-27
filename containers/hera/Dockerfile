FROM ubuntu:xenial

RUN apt update -y && apt install -y \
        git \
        cmake \
        g++ \
        make \
        libleveldb-dev libsnappy-dev \
        curl \
        sudo

RUN curl -sL https://deb.nodesource.com/setup_8.x | sudo -E bash - && \
    apt install -y \
        nodejs \
        bash jq bc \
        python3 \
        python

RUN adduser --disabled-password --gecos '' builder && adduser builder sudo && printf 'builder\tALL=NOPASSWD: ALL\n' >> /etc/sudoers

USER builder
WORKDIR /home/builder

#RUN git clone --recursive https://github.com/ethereum/cpp-ethereum --branch develop --single-branch --depth 1
RUN git clone https://github.com/ethereum/cpp-ethereum -b ewasm-json-trace --single-branch
RUN cd cpp-ethereum && git submodule update --init --recursive
RUN cd cpp-ethereum/hera && git remote add jwasinger https://github.com/jwasinger/hera && git fetch jwasinger && git checkout jwasinger/meter-evm2wasm-op && git submodule update --recursive

RUN cd cpp-ethereum && echo "{}"                                         \
          | jq ".+ {\"repo\":\"$(git config --get remote.origin.url)\"}" \
          | jq ".+ {\"branch\":\"$(git rev-parse --abbrev-ref HEAD)\"}"  \
          | jq ".+ {\"commit\":\"$(git rev-parse HEAD)\"}"               \
          > ~/cpp-ethereum-version.json

RUN cd cpp-ethereum/hera && echo "{}"                                    \
          | jq ".+ {\"repo\":\"$(git config --get remote.origin.url)\"}" \
          | jq ".+ {\"branch\":\"$(git rev-parse --abbrev-ref HEAD)\"}"  \
          | jq ".+ {\"commit\":\"$(git rev-parse HEAD)\"}"               \
          > ~/hera-version.json

RUN mkdir -p build
RUN cd build && cmake ../cpp-ethereum -DCMAKE_BUILD_TYPE=RelWithDebInfo -DHERA=ON -DHERA_DEBUGGING=ON -DHERA_EVM2WASM=ON -DHERA_EVM2WASM_EVM_TRACE=ON
RUN cd build && make -j8

RUN cp cpp-ethereum/scripts/jsonrpcproxy.py ~/jsonrpcproxy.py
ADD ewasm-testnet-cpp-config.json ~/ewasm-testnet-cpp-config.json

# ADD cpp-eth.sh /cpp-eth.sh

#USER builder
#WORKDIR /home/builder

# Export the usual networking ports to allow outside access to the node
EXPOSE 8545 30303

# ENTRYPOINT ["/build/test/testeth"]
