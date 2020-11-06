# Ethereum Lite Explorer by Alethio
The **Lite Explorer**  is a client-side only web application that connects directly to a [Ethereum JSON RPC](https://github.com/ethereum/wiki/wiki/JSON-RPC) compatible node.
This means you can have your own private Ethereum Explorer should you wish so.
No need for servers, hosting or trusting any third parties to display chain data.

## Technical Details

The project is built on a React/MobX and TypeScript stack, using the [Alethio CMS](https://github.com/Alethio/cms), which allows us to add extensions dynamically through 3rd party plugins.
The basic functionality of the explorer is implemented via a series of open-source [core plugins](https://github.com/Alethio/explorer-core-plugins), which we also use internally for our [aleth.io](https://aleth.io) platform. Please refer to [Alethio CMS](https://github.com/Alethio/cms) for documentation on the plugin system.

### Prerequisites
Please make sure you have the following installed and running properly
- [Docker](https://www.docker.com/)
- [A Kubernetes Cluster] (https://labs.play-with-k8s.com/) 
- A JSON-RPC enabled and accessible Ethereum Client, some examples:
    * [An Infura Account](#with-infura)
    * [Parity Light Client](#with-parity-light-client)
    * [Ganache](#with-ganache)
    * [Besu](#with-Besu) - private chain example

### Application Configuration Files

The application requires a JSON configuration file which is loaded at runtime. 

Here are 3 sample config files as starting point.

| Config name | Description |
| --- | --- |
| config.default.json | Default configuration file which contains the core plugins of the app that are enough to run the explorer. |
| config.ibft2.json | Configuration file that has the default core plugins plus an extra one useful for [IBFT2 based chains](https://pegasys.tech/another-day-another-consensus-algorithm-why-ibft-2-0/) that decodes the extraData field of a block. |
| config.memento.json | Configuration file that has the default core plugins plus the memento plugins to use the Memento API as a data source |

The possibility to change the URL of the RPC enabled Ethereum node is done through the `eth-lite` core plugin.
See the [`nodeUrl`](https://github.com/Alethio/ethereum-lite-explorer/blob/master/config.default.json#L16) attribute for the plugin which has the default value set to `https://mainnet.infura.io/`.

### Running in Docker
You can run the Lite Explorer in Docker by using the already published images on [Docker Hub](https://hub.docker.com/r/alethio/ethereum-lite-explorer).
The config file in the Docker images have the default values from the `config.default.json` sample file.
By default it will connect to `https://mainnet.infura.io/`.

The simplest command to run it is
```sh
$ docker run -p 80:80 alethio/ethereum-lite-explorer
```
which will start a container on port 80 of your computer with a nginx embedded to serve the pre-build explorer. You can now open [localhost](http://localhost) in your browser and use it.

There are 3 env vars that can be passed in at runtime:

| ENV var | Description |
|---|---|
| APP_NODE_URL | URL of RPC enabled node. (e.g. `https://host:port`, also supports Basic Auth by prepending `user:pass@` to the `host`). This overrides in the config file the `nodeUrl` attribute of the `eth-lite` core plugin. |
| APP_BASE_URL | It is used ONLY in `index.html` for `og:tags` (e.g. `https://my.app.tld`). Overrides build time defined value. |
| APP_BASE_PATH | Enables serving the app on a sub-path instead of the domain root (e.g. some/path/to/app).
 
 Example: If serving the app from https://my.tld/path/to/app: $ APP_BASE_URL="https://my.tld" APP_BASE_PATH="path/to/app"
  

For example if you want to connect to your node on localhost with all default configs run the following command:
```sh
$ docker run -p 80:80 -e APP_NODE_URL="http://localhost:8545" alethio/ethereum-lite-explorer
```
If more customization is needed, a full configuration file can be mounted in the application root (e.g. in the `/usr/share/nginx/html` folder).
```sh
$ docker run -p 80:80 -v /your-config-dir/config.json:/usr/share/nginx/html/config.json alethio/ethereum-lite-explorer
```
### Running in Kubernetes
You can deploy the Lite Explorer in Kubernetes using the following steps:

Helm charts is also available in `./charts/`.

```shell
cd ./charts/alethio-lite-explorer
helm install alethio-lite-explorer ./alethio-lite-explorer
```
values.yaml file has Default values for alethio-lite-explorer. You need to declare variables or modify values.yaml to be passed into your templates.

### ***DEPLOYING WITH HELM ***

Helmfile deploys an Hyperledger Besu node connected to the Gorli testnet and an Alethio Lite Explorer. To Deploy
 
```
 helmfile sync
```
 If LoadBalancer IP is not available while deploying the chart a second helmfile sync may be needed. 

 
Service (type=LoadBalancer) that exposes the RPC port (default=8545) on the node service loadbalancer's external IP. an ingress-nginx controller to handle ingress traffic. Alethio Lite Explorer that is accessible through an Ingress

to debug

```
helmfile --debug sync
```




### Example Setups
#### With Memento
[Memento](https://github.com/Alethio/memento) is Alethio's open source tool for scraping and indexing Ethereum data from any web3-compatible node.
The biggest advantage of using Memento as a data source is the indexed data which allows a faster access as well as the ability to show transactions on the account page.

**Easiest way to run with Memento** is to follow the steps from [Running in Docker](#running-in-docker) and mount `config.memento.js` as config file.

#### With Infura
- [Sign-up](https://infura.io/register) for an account or [sign-in](https://infura.io/login) into your Infura account.

- From the control panel, obtain your endpoint url for the network you are interested in (mainnet, ropsten, kovan, rinkeby, goerli). 
It will looks similar to `https://mainnet.infura.io/v3/aa11bb22cc33.....`.

- Update `config.infura.json` file and set the `nodeUrl` attribute for the `eth-lite` plugin to your Infura endpoint.

**Easiest way to run with Infura** is to follow the steps from [Running in Docker](#running-in-docker) and mount `config.infura.js` as config file.

####  With Parity Light Client
This will allow you to run both your own node and explorer. No third-party dependencies.
It will be slower to browse older data because it is fetching it real time from other ethereum peer nodes but it's fast to sync and low in resource usage.

[Install Parity Ethereum](https://wiki.parity.io/Setup) through one of the convenient methods and start it with the `--light` cli flag.

As a simple step, if you have Docker, you could just run

```sh
$ docker run -d --restart always --name parity-light -p 127.0.0.1:8545:8545 parity/parity:stable --light --jsonrpc-interface all
```

Update `config.dev.json` file and set the `nodeUrl` attribute for the `eth-lite` plugin to `http://127.0.0.1:8545`.

#### With Ganache
First of all, if you do not have it, download and install [Ganache](https://truffleframework.com/ganache) which will give you your own personal test chain.

After setting up and starting Ganache, update the `config.dev.json` file and set the `nodeUrl` attribute for the `eth-lite` plugin to `http://127.0.0.1:7545`.

#### With Besu
This is a great way to use a full featured client, and to see how the explorer works with a private network. 
If you want to deploy to other networkks. From the control panel, obtain your endpoint url for the network you are interested in (mainnet, ropsten, kovan, rinkeby, goerli). 
It will looks similar to `https://mainnet.infura.io/v3/aa11bb22cc33.....`.

**Easiest way to run alethio** is to follow the steps from [Running in Docker](#running-in-docker) and mount `config.js` as config file. 

**Easiest way to run Besu** is to run a Besu node in a Docker container. The Docker image does not run on Windows.

Use this Docker image to run a single Besu node without installing Besu. Besu Docker images are at 
https://hub.docker.com/r/hyperledger/besu/tags lists the available tags for the image.

To run a Besu node in a container connected to the Ethereum MainNet and this is the default mode:

docker run hyperledger/besu:latest

##### Exposing Ports

Expose ports for P2P peer discovery, GraphQL, metrics, and HTTP and WebSockets JSON-RPC. You need to expose the ports to use the default ports or the ports specified using --rpc-http-port, --p2p-port, --rpc-ws-port, --metrics-port, --graphql-http-port, and --metrics-push-port options.

To run Besu exposing local ports for access:

```
docker run -p <localportJSON-RPC>:8545 -p <localportWS>:8546 -p <localportP2P>:30303 hyperledger/besu:latest --rpc-http-enabled --rpc-ws-enabled
```

***example***

To enable JSON-RPC HTTP calls to `127.0.0.1:8545` and P2P discovery on `127.0.0.1:13001`:

```bash
docker run -p 8545:8545 -p 13001:30303 hyperledger/besu:latest --rpc-http-enabled
```

##### Starting Besu
[Run Besu from Docker Image](https://besu.hyperledger.org/en/stable/HowTo/Get-Started/Run-Docker-Image/). You can specify Besu environment variables with the docker image instead of the command line options

```
docker run -p 30303:30303 -p 8545:8545 -e BESU_RPC_HTTP_ENABLED=true -e BESU_NETWORK=goerli hyperledger/besu:latest
```

!!! important

Do not mount a volume at the default data path (`/opt/besu`). Mounting a volume at the default
data path interferes with the operation of Besu and prevents Besu from safely launching.

To run a node that maintains the node state (key and database),
[`--data-path`](https://github.com/hyperledger/besu-docs/blob/master/docs/Reference/CLI/CLI-Syntax.md#data-path) must be set to a location other than `/opt/besu` and a storage volume mounted at that location.

When running in a Docker container, [`--nat-method`](https://github.com/hyperledger/besu-docs/blob/master/docs/HowTo/Find-and-Connect/Specifying-NAT.md) must be set to `DOCKER` or `AUTO` (default). Do not set [`--nat-method`](https://github.com/hyperledger/besu-docs/blob/master/docs/HowTo/Find-and-Connect/Specifying-NAT.md) to `NONE` or `UPNP`.

***Run a node for testing**
To run a node that mines blocks at a rate suitable for testing purposes with WebSockets enabled:

```
docker run -p 8546:8546 --mount type=bind,source=/<myvolume/besu/testnode>,target=/var/lib/besu hyperledger/besu:latest --miner-enabled --miner-coinbase fe3b557e8fb62b89f4916b721be55ceb828dbd73 --rpc-ws-enabled --network=dev --data-path=/var/lib/besu
```

***To run a node on Goerli***

docker run -p 30303:30303 --mount type=bind,source=/<myvolume/besu/goerli>,target=/var/lib/besu hyperledger/besu:latest --network=goerli --data-path=/var/lib/besu

***Alethio Ethereum Lite Explorer***

Using Docker is the easiest way to get started using the Ethereum Lite Explorer with Hyperledger Besu 

Use the Alethio Ethereum Lite Explorer to explore blockchain data at the block, transaction, and account level.

The Alethio Ethereum Lite Explorer is a web application that connects to any Ethereum JSON-RPC-enabled node. The Explorer does not require an online server, hosting, or trusting third parties to display the blockchain data.

[Using Docker is the easiest way to get started using the Ethereum Lite Explorer with Hyperledger Besu](https://besu.hyperledger.org/en/stable/HowTo/Deploy/Lite-Block-Explorer/)

To run the Ethereum Lite Explorer using the Docker image:

1. Start Besu with the --rpc-http-enabled option. To run Besu in development mode:

```
besu --network=dev --miner-enabled --miner-coinbase=0xfe3b557e8fb62b89f4916b721be55ceb828dbd73 --rpc-http-cors-origins="all" --host-allowlist="*" --rpc-http-enabled --data-path=/tmp/tmpDatdir
```

2. Run the alethio/ethereum-lite-explorer Docker image specifying the JSON-RPC HTTP URL (http://localhost:8545 in this example):

```
docker run --rm -p 8080:80 -e APP_NODE_URL=http://localhost:8545 alethio/ethereum-lite-explorer
```

3. Open http://localhost:8080 in your browser to view the Lite Explorer. We are using port 8080 to run the Ethereum Lite Explorer so the EthStats Lite can use port 80, allowing you to run both at the same time.

***Alethio EthStats Lite (network monitor)***

See active nodes or add your own on the following running deployments of the EthStats Network Monitor

Mainnet - ethstats.io
Görli Testnet - https://stats.goerli.net/
