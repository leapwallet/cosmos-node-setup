# Cosmos Node Setup

This repo explains how to set up [Cosmos](https://cosmos.network/) nodes. We created it because there are zero comprehensive articles, videos, etc. on doing this, and the official docs on how to just set up a node is confusing.

This repo is written for people who know nothing about running a Cosmos node. Concepts specific to operating Cosmos nodes such as how much storage is required, and how to connect the blockchain node to Prometheus are explained. Concepts specific to Cosmos such as what a blockchain is, and tools used such as Prometheus aren't explained. This is because the official docs are good, and you can look up only what you need to based on your experience level.

Here's the [overview](docs/overview.md).

## Contributing

### Installation

1. Install [Node.js 16](https://nodejs.org/en/download/).
2. Clone the repo using one of the following methods:

   - SSH:

     ```shell
     git@github.com:leapwallet/cosmos-node-setup.git
     ```

   - HTTPS:

     ```shell
     https://github.com/leapwallet/cosmos-node-setup.git
     ```

3. Change the directory:

   ```shell
   cd notification-system-architecture
   ```

4. Install the package manager:

   ```shell
   corepack enable
   ```

5. Install the dependencies:

   ```shell
   yarn
   ```

### Usage

Lint:

```shell
yarn lint:fix
```

## Credits

- [Juno Docs](https://docs.junonetwork.io/juno/readme)
- [Sei Docs](https://docs.seinetwork.io/introduction/overview)
- [Stride Docs](https://docs.stride.zone/docs)
- [Osmosis Docs](https://docs.osmosis.zone)
- [Commercio.network Docs](https://docs.commercio.network/)
- [NodeJumper](https://nodejumper.io/)
- [Validator Security Checklist](https://docs.evmos.org/validators/security/checklist.html)

## License

This project is under the [MIT License](LICENSE).
