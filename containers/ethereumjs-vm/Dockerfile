from node:alpine

WORKDIR /ethereum

RUN apk update && apk upgrade && apk add --no-cache g++ python python-dev git make bash && git clone https://github.com/jwasinger/ethereumjs-vm && cd ethereumjs-vm && git checkout temp-trace-equivalence && npm install

ENTRYPOINT ["node", "/ethereum/ethereumjs-vm/tests/tester.js" ]
