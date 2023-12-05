# Polygon ID Issuer Node

[![Checks](https://github.com/0xPolygonID/sh-id-platform/actions/workflows/checks.yml/badge.svg)](https://github.com/0xPolygonID/sh-id-platform/actions/workflows/checks.yml)
[![golangci-lint](https://github.com/0xPolygonID/sh-id-platform/actions/workflows/golangci-lint.yml/badge.svg)](https://github.com/0xPolygonID/sh-id-platform/actions/workflows/golangci-lint.yml)

This is a set of tools and APIs for issuers of zk-proof credentials, designed to be extensible. It allows an authenticated user to create schemas for issuing and managing credentials of identities. It also provides a [user interface](ui/README.md) to manage issuer schemas, credentials, issuer state and connections.

This repository is for anyone to create their own [issuer node](https://0xpolygonid.github.io/tutorials/issuer-node/issuer-node-overview/) for Polygon ID.

---

### Docker Setup Guide

Running the app with Docker allows for minimal installation and a quick setup. This is recommended **for evaluation use-cases only**, such as local development builds.

#### (Optional) Quick Start Steps

These steps can be followed to get up and running with all features as quickly as possible.

> **NOTE:** For more detailed step-by-step instructions and guides to commands and examples, you may skip to the next section.

1. Copy `.env-api.sample` as `.env-api` and `.env-issuer.sample` as `.env-issuer`. Please see the [configuration](#configuration) section for more details.
1. Run `make up`. This launches 3 containers with Postgres, Redis and Vault. Ignore the warnings about variables, since those are set up in the next step.
1. **If you are on an Apple Silicon chip (e.g. M1/M2), run `make run-arm`**. Otherwise, run `make run`. This starts up the issuer API, whose frontend can be accessed via the browser (default <http://localhost:3001>).
1. [Add](#import-wallet-private-key-to-vault) your Ethereum private key to the Vault.
1. [Add](#add-vault-to-configuration-file) the Vault to the config.
1. [Create](#create-issuer-did) your issuer DID.
1. _(Optional)_ To run the UI with its own API, first copy `.env-ui.sample` as `.env-ui`. Please see the [configuration](#development-ui) section for more details.
1. _(Optional)_ Run `make run-ui` (or `make run-ui-arm` on Apple Silicon) to have the Web UI available on <http://localhost:8088> (in production mode). Its HTTP auth credentials are set in `.env-ui`. The UI API also has a frontend for API documentation (default <http://localhost:3002>).

#### Docker Guide Requirements

- Unix-based operating system (e.g. Debian, Arch, Mac OS)
- [Docker Engine](https://docs.docker.com/engine/) `1.27+`
- Makefile toolchain `GNU Make 3.81`

> **NOTE:** There is no compatibility with Windows environments at this time.

To help expedite a lot of the Docker commands, many have been abstracted using `make` commands. Included in the following sections are the equivalent Docker commands that show what is being run.

#### Create Docker Configuration Files

Make a copy of the following environment variables files:

```bash
# FROM: ./

cp .env-api.sample .env-api;
cp .env-issuer.sample .env-issuer;
# (Optional - For issuer UI)
cp .env-ui.sample .env-ui;

```

#### Node Issuer Configuration

The `.env-issuer` will be loaded into the [Docker compose initializer](/infrastructure/local/docker-compose.yml)

Any of the following RPC providers can be used:

- [Chainstack](https://chainstack.com/)
- [Ankr](https://ankr.com/)
- [QuickNode](https://quicknode.com/)
- [Alchemy](https://www.alchemy.com/)
- [Infura](https://www.infura.io/)

If it is desired to run a free public forwarding URL, see [Getting A Public URL](#getting-a-public-url).

**NOTA: USE A LOCAL IP ADDRESS HERE*

Configure `.env-issuer` with the following details (or amend as desired).

```bash
# ...

# See Section: Getting A Public URL
ISSUER_SERVER_URL=<LOCAL IP ADDRESS>
# Defaults for Basic Auth in Base64 ("user-issuer:password-issuer" = "dXNlci1pc3N1ZXI6cGFzc3dvcmQtaXNzdWVy")
# If you just want to get started, don't change these
ISSUER_API_AUTH_USER=user-issuer
ISSUER_API_AUTH_PASSWORD=password-issuer
# !!!MUST BE SET or other steps will not work
ISSUER_ETHEREUM_URL=<YOUR_RPC_PROVIDER_URI_ENDPOINT>
```

> **NOTE:** In case the Vault was loaded multiple times and a fresh start is needed, the following will remove remnant data:

```bash
# FROM: ./
make clean-vault;
```

#### Start Redis Postgres & Vault

This will start the necessary local services needed to store the wallet private key to the Hashicorp vault and allow storing data associated to the issuer.

```bash
# FROM: ./
make up;
```

To remove all services, run the following (ignore the warnings):

```bash
# FROM: ./
make down; 
```

#### Import Wallet Private Key To Vault

In order to secure the wallet private key so that the issuer can use it to issue credentials, it must be stored in the Hashicorp Vault.

> **NOTE:** Make sure the wallet that is provided has Testnet Matic to be able to send transactions.
> **NOTE: Repeat this command until you get the expected Output **

```bash
# FROM: ./

# Make sure to verify that the issuer-vault-1 is full initialized to avoid: "Error writing data to iden3/import/pbkey: Error making API request."
make private_key=<YOUR_WALLET_PRIVATE_KEY> add-private-key;
# (Equivalent)
#   docker exec issuer-vault-1 vault write iden3/import/pbkey key_type=ethereum private_key=<YOUR_WALLET_PRIVATE_KEY>;

# Expected Output:
#   docker exec issuer-vault-1 \
#           vault write iden3/import/pbkey key_type=ethereum private_key=<YOUR_WALLET_PRIVATE_KEY>
#   Success! Data written to: iden3/import/pbkey
```

#### Setup Vault

##### Using root vault token
This will get the vault token from the Hashicorp vault docker instance and add it to our `./env-issuer` file.

```bash
# FROM: ./
make add-vault-token;
```


#### Create Issuer DID

> **NOTE:** This can also be done via the [UI API](#using-the-ui-api).

This will create a new issuer DID by creating a new Docker instance of the issuer, generating the DID of the issuer, storing it in the database, then deleting the instance.
This command will not generate a new DID if one already exists stored in Vault. If you want to generate a new DID, you must first delete the DID from Vault. For that, run the following command: `make delete-did`

**For _NON-Apple-M1/M2/Arm_ (ex: Intel/AMD):**

```bash
# FROM: ./

# NON-Apple-M1/M2/Arm Command:
make generate-issuer-did;

```

**For _Apple-M1/M2/Arm_:**

```bash
# FROM: ./

# Apple-M1/M2/Arm Command:
make generate-issuer-did-arm;
```

#### Start Issuer API

Now that the issuer API is configured, it can be started.

**For _NON-Apple-M1/M2/Arm_ (ex: Intel/AMD):**

```bash
# FROM: ./

make run;
```

**For _Apple-M1/M2/Arm_:**

```bash
# FROM: ./

make run-arm;
# (Equivalent)
#   COMPOSE_DOCKER_CLI_BUILD=1 DOCKER_FILE="Dockerfile-arm" docker compose -p issuer -f /Users/username/path/to/sh-id-platform/infrastructure/local/docker-compose.yml up -d api;

# Expected Output:
#   COMPOSE_DOCKER_CLI_BUILD=1 DOCKER_FILE="Dockerfile-arm" docker compose -p issuer -f /Users/username/path/to/sh-id-platform/local/docker-compose.yml up -d api;
#   WARN[0000] Found orphan containers ([issuer-vault-1 issuer-postgres-1 issuer-redis-1]) for this project. If you removed or renamed this service in your compose file, you can run this command with the --remove-orphans flag to clean it up. 
```

Navigating to <http://localhost:3001> shows te issuer API's frontend:

![Issuer API frontend](docs/assets/img/3001.png)

#### (Optional) Configure UI

This step is required to run the separate UI application, which allows intuitive and convenient management of schemas, credentials, connections and issuer state.

> **NOTE:** Running and using the UI is optional, since it implements funcionality already exposed via the [UI API](#using-the-ui-api). It is highly recommended though, because it makes issuer management far simpler and more intuitive.

```bash
# FROM: ./

cp .env-ui.sample .env-ui;
```

Configure the `.env-ui` file with the following details (or amend as desired):

```bash
ISSUER_UI_BLOCK_EXPLORER_URL=https://mumbai.polygonscan.com
ISSUER_UI_AUTH_USERNAME=user-ui
ISSUER_UI_AUTH_PASSWORD=password-ui
```

#### Start API UI, UI, Notifications server & Publisher

This will start the UI API that exposes endpoints to manage schemas, credentials, connections and issuer state, as well as the UI that relies on it.

**For _NON-Apple-M1/M2/Arm_ (ex: Intel/AMD):**

```bash
# FROM: ./

make run-ui;
```

**For _Apple-M1/M2/Arm_:**

```bash
# FROM: ./
make run-ui-arm;
```

Now navigate to <http://localhost:3002> to see the UI API's frontend:

![Issuer UI API frontend](docs/assets/img/3002.png)

#### Using the UI API

Make sure to set the HTTP authentication credentials in `.env-api` to the following:

```bash
# ...

ISSUER_API_UI_AUTH_USER=user-api
ISSUER_API_UI_AUTH_PASSWORD=password-api
```

Then authenticate via the following form on <http://localhost:3002>:

![Issuer UI API Authentication](docs/assets/img/3002-auth.png)

This allows you to make a request via any of the endpoints using this frontend.

![Issuer UI API Get Credentials](docs/assets/img/3002-credentials.png)

#### (Optional) Using the UI

This service is running on <http://localhost:8088>.

> **NOTE:** If you are using Chrome, you might get the HTTP auth modal showing and disappearing quickly. To remedy this, use the following URL: <http://user-api:password-api@localhost:8088/>.

File containing the basic auth credentials: `.env-ui`

```bash
# ...

ISSUER_UI_AUTH_USERNAME=user-ui
ISSUER_UI_AUTH_PASSWORD=password-ui
```

![Issuer UI](docs/assets/img/8088.png)


---

