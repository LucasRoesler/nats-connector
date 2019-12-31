# nats-connector

[![Go Report Card](https://goreportcard.com/badge/github.com/openfaas-incubator/nats-connector)](https://goreportcard.com/report/github.com/openfaas-incubator/nats-connector) [![Build
Status](https://travis-ci.com/openfaas-incubator/nats-connector.svg?branch=master)](https://travis-ci.org/openfaas-incubator/nats-connector) [![GoDoc](https://godoc.org/github.com/openfaas-incubator/nats-connector?status.svg)](https://godoc.org/github.com/openfaas-incubator/nats-connector) [![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![OpenFaaS](https://img.shields.io/badge/openfaas-serverless-blue.svg)](https://www.openfaas.com)

The NATS connector connects OpenFaaS functions to NATS topics.

## Building

```
export TAG=0.2.0
make build push
```

## Try it out

Test functions have been created to help you verify the installation of `nats-connector`.  See [`contrib/test-functions`](./contrib/test-functions).

### Deploy on Kubernetes

The following instructions show how to run and test `nats-connector` on Kubernetes.

1. Deploy the receiver functions, the receiver function must have the `topic` annotation:

   ```bash
   export OPENFAAS_URL="http://localhost:8080" # Set your gateway via env variable or the -g flag
   faas-cli deploy --name receive-message --image openfaas/receive-message:latest --fprocess='./handler' --annotation topic="nats-test"
   ```

   Alternatively, you can deploy with the `stack.yml` provided in this repo
   ```
   cd contrib/test-functions
   faas-cli deploy -f stack.yml
   ```

2. Deploy the connector with:

   ```bash
   kubectl apply -f ./yaml/kubernetes/connector-dep.yml
   ```

3. Now publish a message on the `nats-test` topic.  We have provided a simple function to help do this

   ```bash
   faas-cli deploy --name publish-message --image openfaas/publish-message:latest --fprocess='./handler' --env nats_url=nats://nats.openfaas:4222
   ```
   If you used the `stack.yml` in step 1, then this function is already deployed.

   Invoke the publisher
   ```bash
   faas-cli invoke publish-message <<< "test message"
   ```

4. Verify that the receiver was invoked by checking the logs

   ```bash
   faas-cli logs receive-message

   2019-12-29T19:06:50Z 2019/12/29 19:06:50 received "test message"
   ```

### Configuring the NATS topics
If you want to configure the topics that the connector listens to, then edit the `topics` env variable in  [yaml/kubernetes/connector-dep.yaml](./yaml/kubernetes/connector-dep.yaml)
