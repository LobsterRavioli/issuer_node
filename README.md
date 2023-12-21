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
3.  Run this to create the private key
```bash
make private_key=<YOUR WALLET PRIVATE KEY>;
```
4. Run this to add the private key to the vault (Repeat this until you get the expected output)
```bash
make add-vault-token;
# Expected Output:
#   docker exec issuer-vault-1 \
#           vault write iden3/import/pbkey key_type=ethereum private_key=<YOUR_WALLET_PRIVATE_KEY>
#   Success! Data written to: iden3/import/pbkey
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

---
