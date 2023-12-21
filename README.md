# Issuer node docs

## Dependencies
1. Make command.
2. Docker. (and run docker desktop)

### Clone this repo
```bash
git clone https://github.com/LobsterRavioli/issuer_node.git
```
### Move to the project folder
```bash
cd issuer_node
```
### Setup environment variables 

1. Make a copy of the following environment variables files:
```bash
cp .env-api.sample .env-api;
cp .env-issuer.sample .env-issuer;
cp .env-ui.sample .env-ui;
```
2. Init environment variable .env-issuer :
```bash
ISSUER_SERVER_URL=<LOCAL IP ADDRESS>
#EXAMPLE ISSUER_SERVER_URL= http://192.168.1.3:3002
ISSUER_API_AUTH_USER=user-issuer
ISSUER_API_AUTH_PASSWORD=password-issuer
ISSUER_ETHEREUM_URL=<YOUR_RPC_PROVIDER_URI_ENDPOINT>
#EXAMPLE ISSUER_ETHEREUM_URL=https://polygon-mumbai.infura.io/v3/90372d43a2b34c169d5fed6bf77fa349 
```
You can take ISSUER_ETHEREUM_URL from rpc providers like infura.

## How to run the issuer node
Every time you want the issuer node up, you need to execute sequentially this commands.

1. Run this to clear the node.
```bash
make clean-vault;
```
2. Run this to start the node
```bash
make up;
```
If you want to shut down the node run this command: 
```bash
make down;
```

3.  Run this to create the private key (Repeat this until you get the expected output).
```bash
make private_key=<YOUR WALLET PRIVATE KEY> add-private-key;
# Expected Output:
#   docker exec issuer-vault-1 \
#           vault write iden3/import/pbkey key_type=ethereum private_key=<YOUR_WALLET_PRIVATE_KEY>
#   Success! Data written to: iden3/import/pbkey
```
4. Run this to add the private key to the vault.
```bash
make add-vault-token;

```
5. Run this to generate the did of the issuer node.
```bash
make generate-issuer-did;
#OR
make generate-issuer-did-arm; # for m1/m2 mac users
```
6. Run this to start the node-ui.
```bash
make run-ui;
#OR
make run-ui-arm; # for m1/m2 mac users
```
#### Using the UI API

Make sure to set the HTTP authentication credentials in `.env-api` to the following:

```bash
ISSUER_API_UI_AUTH_USER=user-api
ISSUER_API_UI_AUTH_PASSWORD=password-api
```

Then authenticate via the following form on <http://YOUR_LOCAL_IP_ADDRESS>

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

### How to get the QR code for JSON content

Use [qe](https://qr.io/) to generate the QR code (in the "Text" section) and paste the body provided by the UI-API.

### Issue a Credential
1. In the issuer-UI-API, navigate to the "Create a Credential" section and paste the DID of the wallet holder. (For testing purposes, the issuer-API allows you to define and issue a KYAge credential.)
2. Copy the ID provided in the response.
3. Go to the "Get Credential" section and paste the ID.
4. Visit [qe](https://qr.io/) "Text section" and paste the JSON content.
5. Proceed to the "Get Connection QR Code" section and copy the JSON content.
6. Go to [qe](https://qr.io/) "Text section" and paste the JSON content.

Allow the wallet to scan the authentication QR code first and then the issued credential QR code.
---
