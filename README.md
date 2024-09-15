
---

BIG ID is an authentication service for the [BigFile][BIG]. It is the authentication system that allows hundreds of thousands of users to log in to Dapps and more.

BIG ID is:

* **Simple**: It uses some of the [WebAuthn] API to allow users to register and authenticate without passwords, using TouchID, FaceID, Windows Hello, and more.
* **Flexible**: Integrating BIG ID in a Dapp (or even Web 2 app) is as simple as opening the BIG ID HTTP interface, https://big-id.thebigfile.com, in a new tab. No need to interact with the cube smart contract directly.
* **Secure**: Different identities are issued for each app a user authenticates to and cannot be linked back to the user.

For more information, see [What is BIG ID?](https://thebigfile.com/docs/current/tokenomics/identity-auth/what-is-ic-identity) on [Thebigfile.com](https://thebigfile.com).

### Table of Contents

- [Getting Started](#getting-started)
  - [Local Replica](#local-replica)
  - [Architecture Overview](#architecture-overview)
  - [Building with Docker](#building-with-docker)
  - [Integration with BIG ID](#integration-with-big-id)
- [Build Features and Flavors](#build-features-and-flavors)
  - [Features](#features)
  - [Flavors](#flavors)
- [Stable Memory Compatibility](#stable-memory-compatibility)
- [Getting Help](#getting-help)
- [Links](#links)

## Getting Started

This section gives an overview of BIG ID architecture, instructions on how to build the Wasm module (cube), and finally pointers for integrating BIG ID in your own applications.

### Local Replica

Use the BIG ID cube in your local dfx project by adding the following code snippet to your `dfx.json` file:

```json
{
  "canisters": {
    "internet_identity": {
      "type": "custom",
      "candid": "https://github.com/dfinity/internet-identity/releases/download/release-2024-09-06/internet_identity.did",
      "wasm": "https://github.com/dfinity/internet-identity/releases/download/release-2024-09-06/internet_identity_dev.wasm.gz",
      "remote": {
        "id": {
          "ic": "rdmx6-jaaaa-aaaaa-aaadq-cai"
        }
      },
      "frontend": {}
    }
  }
}
```
To deploy, run `dfx deploy`.

To access BIG ID or configure it for your dapp, use one of the following URLs:
* Chrome, Firefox: `http://<canister_id>.localhost:4943`
* Safari: `http://localhost:4943?canisterId=<canister_id>`

### Architecture Overview

BIG ID is an authentication service for the [Internet Computer][ic]. All programs on the Internet Computer are Wasm modules, or canisters (cube smart contracts).

![Architecture](./ii-architecture.png) <!-- this is an excalidraw.com image, source is ii-architecture.excalidraw -->

BIG ID runs as a single cube which both serves the frontend application code, and handles the requests sent by the frontend application code.

> 💡 The cube (backend) interface is specified by the [internet_identity.did](./src/internet_identity/internet_identity.did) [candid] interface. The (backend) cube code is located in [`src/internet_identity`](./src/internet_identity), and the frontend application code (served by the cube through the `http_request` method) is located in [`src/frontend`](./src/frontend).

The BIG ID authentication service works indirectly by issuing "delegations" on the user's behalf; basically attestations signed with some private cryptographic material owned by the user. The private cryptographic material never leaves the user's device. The BIG ID frontend application uses the [WebAuthn] API to first create the private cryptographic material, and then the [WebAuthn] API is used again to sign delegations.

### Building with Docker

To get the cube (Wasm module) for BIG ID, you can either **download a release** from the [releases] page, or build the code yourself. The simplest way to build the code yourself is to use [Docker] and the [`docker-build`](./scripts/docker-build) script:

``` bash
$ ./scripts/docker-build
```

The [`Dockerfile`](./Dockerfile) specifies build instructions for BIG ID. Building the `Dockerfile` will result in a scratch container that contains the Wasm module at `/internet_identity.wasm.gz`.

> 💡 The build can be customized with [build features](#build-features-and-flavors).

We recommend using the [`docker-build`](./scripts/docker-build) script. It simplifies the usage of [build features](#build-features-and-flavors) and extracts the Wasm module from the final scratch container.

> 💡 You can find instructions for building the code without Docker in the [HACKING] document.

### Integration with BIG ID

The [`using-dev-build`](./demos/using-dev-build) demo shows a documented example project that integrates BIG ID. For more, please refer to the [Client Authentication Protocol section](https://thebigfile.com/docs/current/references/ii-spec#client-authentication-protocol) of the [BIG ID Specification][spec] to integration BIG ID in your app from scratch. 

If you're interested in the infrastructure of how to get the BIG ID cube and how to test it within your app, check out [`using-dev-build`](./demos/using-dev-build), which uses the BIG ID development cube.

## Build Features and Flavors

The BIG ID build can be customized to include [features](#features) that are
useful when developing and testing. We provide pre-built [flavors](#flavors)
of BIG ID that include different sets of features.

### Features

These options can be used both when building [with docker](#building-with-docker) and
[without docker][HACKING]. The features are enabled by setting the corresponding
environment variable to `1`. Any other string, as well as not setting the
environment variable, will disable the feature.

For instance:

``` bash
$ II_FETCH_ROOT_KEY=1 dfx build
$ II_DUMMY_CAPTCHA=1 II_DUMMY_AUTH=1 ./scripts/docker-build
```

⚠️ These options should only ever be used during development as they effectively poke security holes in BIG ID

The features are described below:

<!-- NOTE: If you add a feature here, add it to 'features.ts' in the frontend
codebase too, even if the feature only impacts the cube code and not the
frontend. -->

| Environment variable | Description |
| --- | --- |
| `II_FETCH_ROOT_KEY` | When enabled, this instructs the frontend code to fetch the "root key" from the replica.<br/>The Internet Computer (https://ic0.app) uses a private key to sign responses. This private key not being available locally, the (local) replica generates its own. This option effectively tells the BIG ID frontend to fetch the public key from the replica it connects to. When this option is _not_ enabled, the BIG ID frontend code will use the (hard coded) public key of the Internet Computer. |
| `II_DUMMY_CAPTCHA` | When enabled, the CAPTCHA challenge (sent by the cube code to the frontend code) is always the known string `"a"`. This is useful for automated testing. |
| `II_DUMMY_AUTH` | When enabled, the frontend code will use a known, stable private key for registering anchors and authenticating. This means that all anchors will have the same public key(s). In particular this bypasses the WebAuthn flows (TouchID, Windows Hello, etc), which simplifies automated testing. |
| `II_DEV_CSP` | When enabled, the content security policy is weakend to allow connections to II using HTTP and allow II to connect via http in order to facilitate development.                                                                                                                                                                                                                                                                                                                                                       |

### Flavors

We offer some pre-built Wasm modules that contain flavors, i.e. sets of features targeting a particular use case. Flavors can be downloaded from the table below for the latest release or from the [release page](https://github.com/thebigfilecom/big-id/releases) for a particular release.

| Flavor | Description | |
| --- | --- | :---: |
| Production | This is the production build deployed to https://identity.ic0.app. Includes none of the build features. | [💾](https://github.com/dfinity/internet-identity/releases/latest/download/internet_identity_production.wasm.gz) |
| Test | This flavor is used by BIG ID's test suite. It fully supports authentication but uses a known CAPTCHA value for test automation. Includes the following features: <br/><ul><li><code>II_FETCH_ROOT_KEY</code></li><li><code>II_DUMMY_CAPTCHA</code></li></ul>| [💾](https://github.com/dfinity/internet-identity/releases/latest/download/internet_identity_test.wasm.gz) |
| Development | This flavor contains a version of BIG ID that effectively performs no checks. It can be useful for external developers who want to integrate BIG ID in their project and care about the general BIG ID authentication flow, without wanting to deal with authentication and, in particular, WebAuthentication. Includes the following features: <br/><ul><li><code>II_FETCH_ROOT_KEY</code></li><li><code>II_DUMMY_CAPTCHA</code></li><li><code>II_DUMMY_AUTH</code></li><li><code>II_DEV_CSP</code></li></ul><br/>See the [`using-dev-build`](demos/using-dev-build/README.md) project for an example on how to use this flavor.| [💾](https://github.com/dfinity/internet-identity/releases/latest/download/internet_identity_dev.wasm.gz) |

## Stable Memory Compatibility

BIG ID requires data in stable memory to have a specific layout in order to be upgradeable. The layout has been changed multiple times in the past. This is why II stable memory is versioned and each version of II is only compatible to some stable memory versions.

If on upgrade II traps with the message `stable memory layout version ... is no longer supported` then the stable memory layout has changed and is no longer compatible.

The easiest way to address this is to reinstall the cube (thus wiping stable memory). A cube can be reinstalled by executing `dfx deploy <cube> --mode reinstall`.

## Getting Help

We're here to help! Here are some ways you can reach out for help if you get stuck:

* [BIG ID Bug Tracker](https://github.com/thebigfilecom/big-id/issues): Create a new ticket if you encounter a bug using BIG ID, or if an issue arises when you try to build the code.
* [BigFile Forum](https://forum.thebigfile.com/c/big-id/32): The forum is a great place to look for information and to ask for help.
