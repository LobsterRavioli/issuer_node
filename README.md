# Polygon ID Issuer Node

[![Checks](https://github.com/0xPolygonID/sh-id-platform/actions/workflows/checks.yml/badge.svg)](https://github.com/0xPolygonID/sh-id-platform/actions/workflows/checks.yml)
[![golangci-lint](https://github.com/0xPolygonID/sh-id-platform/actions/workflows/golangci-lint.yml/badge.svg)](https://github.com/0xPolygonID/sh-id-platform/actions/workflows/golangci-lint.yml)

This is a set of tools and APIs for issuers of zk-proof credentials, designed to be extensible. It allows an authenticated user to create schemas for issuing and managing credentials of identities. It also provides a [user interface](ui/README.md) to manage issuer schemas, credentials, issuer state and connections.

This repository is for anyone to create their own [issuer node](https://0xpolygonid.github.io/tutorials/issuer-node/issuer-node-overview/) for Polygon ID.

---

## Dependencies
1. make command.
2. Docker for running commands.
## Initial setup

	1. Make a copy of the following environment variables files:
```bash
cp .env-api.sample .env-api;
cp .env-issuer.sample .env-issuer;
cp .env-ui.sample .env-ui;
```
2. Init environment variable .env.issuer :
```bash
ISSUER_SERVER_URL=<LOCAL IP ADDRESS>
#EXAMPLE ISSUER_SERVER_URL= http://192.168.1.3:3002
ISSUER_API_AUTH_USER=user-issuer
ISSUER_API_AUTH_PASSWORD=password-issuer
ISSUER_ETHEREUM_URL=<YOUR_RPC_PROVIDER_URI_ENDPOINT>
#EXAMPLE ISSUER_ETHEREUM_URL=https://polygon-mumbai.infura.io/v3/90372d43a2b34c169d5fed6bf77fa349 
```
You can take ISSUER_ETHEREUM_URL from rpc providers like infura.

3. Run project services (key storage and data storage):
```bash
make up; 
```
4. Import wallet private key
```bash
make private_key=<YOUR_WALLET_PRIVATE_KEY> add-private-key;
#EXAMPLE make private_key=  add-private-key;
```
5. Add vault token:
```bash
make add-vault-token;
```
6. Generate issuer did
###### For Intel/Amd processors
```bash
make generate-issuer-did;
```
###### For apple m1/m2 users
```bash
make generate-issuer-did-arm;
```

7. run this
```bash
make down;
```

### To start the node after the setup
1. run this
```bash
make clean-vault;
```
2. run this
```bash
make up;
```
3.  run this
```bash
make private_key=<YOUR WALLET PRIVATE KEY>;
```
4. run this (Repeat this until you get the expected output)
```bash
make add-vault-token;
# Expected Output:
#   docker exec issuer-vault-1 \
#           vault write iden3/import/pbkey key_type=ethereum private_key=<YOUR_WALLET_PRIVATE_KEY>
#   Success! Data written to: iden3/import/pbkey
```
5. run this
```bash
make generate-issuer-did;
#OR
make generate-issuer-did-arm; # for m1/m2 mac users
```
6. run this
```bash
make run-ui;
#OR
make run-ui-arm; # for m1/m2 mac users

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

