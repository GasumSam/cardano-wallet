---------------

# <img align="left" src="https://styles.redditmedia.com/t5_4bhj33/styles/communityIcon_u389y4dxqov61.png" alt="Novapago Labs" style="height: 100px; width:130px;"/> Cardano Research
### Internship project
-------------------------

# Running a Cardano Wallet

Sources:

[Cardano Developers](https://developers.cardano.org/) \
[IOHK](https://github.com/input-output-hk/cardano-wallet) (Github) \
[IOHK Wallet User Guide](https://input-output-hk.github.io/cardano-wallet/user-guide) \
[Cardano Wallet Backend API Docs](https://input-output-hk.github.io/cardano-wallet/api/edge/)\
[Faucet Cardano Testnet](https://testnets.cardano.org/en/testnets/cardano/tools/faucet/)

There's several ways to manage your address and transaction with Cardano Blockchain: 

- [Daedalus](https://daedaluswallet.io/) is probably easier way to operate and delegate with ADA. It's the official Cardano full-node wallet Desktop application. Basicaly you download a complete node first and you can use the interface to send transaction or interact with the wallet later.

- [Yoroi](https://yoroi-wallet.com/#/) is the official light-wallet, available  as a mobile apps and browser extension. The users will not force to download the entire blockchain data (Blockchain operate from backend server).

- [cardano-cli](https://github.com/input-output-hk/cardano-node), as we could seen at [Running a Stake Pool](https://github.com), Cardano nodes has the functionality to operate from cardano-cli generating keys, manages and storage them, create certificates or launch queries, among others possibilities.

- [cardano-wallet](https://github.com/input-output-hk/cardano-wallet) is probably one of the most interesting apps for us because offer functionalities via Command Line Interface (CLI) or Web API. How Daedalus work under-the-hood? With `cardano-wallet` precisely.

You can find `cardano-wallet` **REST API** documentation here: [https://input-output-hk.github.io/cardano-wallet/api/edge/](https://input-output-hk.github.io/cardano-wallet/api/edge/).

In our case, we are interesting to deep in `cardano-wallet` service, thinking to manage the tools that allows us connect our future frontend services with Cardano Blockchain. 

### Creating a `cardano-wallet`

At first step, we need a `cardano-wallet` running into our  background system.

We link you a tutorial to do that directly in your computer ([Installing cardano-wallet](https://developers.cardano.org/docs/get-started/installing-cardano-wallet/)), but one more time we prefer to deploy the services with Docker Compose.

Next [docker-compose.yml](https://github.com/input-output-hk/cardano-wallet/blob/master/docker-compose.yml) is from [IOHK Github `cardano-wallet` repository](https://github.com/input-output-hk/cardano-wallet).

```
version: "3.5"

services:
  cardano-node:
    container_name: node
    image: inputoutput/cardano-node:1.33.0
    environment:
      NETWORK:
    volumes:
      - node-${NETWORK}-db:/data
      - node-ipc:/ipc
    restart: on-failure
    logging:
      driver: "json-file"
      options:
        compress: "true"
        max-file: "10"
        max-size: "50m"

  cardano-wallet:
    container_name: wallet
    image: inputoutput/cardano-wallet:2022.1.18
    volumes:
      - wallet-${NETWORK}-db:/wallet-db
      - node-ipc:/ipc
    ports:
      - 8090:8090
    entrypoint: []
    command: bash -c "
        ([[ $$NETWORK == \"mainnet\" ]] && $$CMD --mainnet) ||
        ($$CMD --testnet /config/${NETWORK}/genesis-byron.json)
      "
    environment:
      CMD: "cardano-wallet serve --node-socket /ipc/node.socket --database /wallet-db --listen-address 0.0.0.0"
      NETWORK:
    restart: on-failure
    logging:
      driver: "json-file"
      options:
        compress: "true"
        max-file: "10"
        max-size: "50m"

volumes:
  node-mainnet-db:
  node-testnet-db:
  node-alonzo-purple-db:
  wallet-mainnet-db:
  wallet-testnet-db:
  wallet-alonzo-purple-db:
  node-ipc:
  node-config:
```
Apparently we don't need to change anything, it's ok to run both service. However, is better if we consider this:

- `cardano-wallet` needs run joined to a node who is connecting to the blockchain. So there's one service for each in docker-compose.yml
- Both service are updated to work together as better posible. This means that instead of use `image: inputoutput/cardano-wallet` or `image: inputoutput/cardano-wallet:latest` to run current image, there's possibilities that one of them give us some problems  working toghether. Both services should be test before. So respect the version who are indicated in the official github is a secure garantee. 
- We customize the `container name` to best experience managing docker container later: and include `    container_name: ...`
- How you can see in the code, it's needed detail about which network our wallet will use. In case of Linux and MacOS it just executing `NETWORK=testnet docker-compose up` to indicate the environment from command prompt. Working with Windows OS this not work. Insteat that, we can modify code directly and change `${NETWORK}` for `testnet` or `mainnet`. We can save a copy of file `.yml` with other name and execute `docker-compose --file newDockerComposeFile.yml up`. Or we can create a `.env` file in the same folder with the parameter `NETWORK=testnet` (our case).

### Running `cardano-wallet`

With a command `NETWORK=testnet` in a `.env` file created in the same folder, we execute `docker-compose up -d` PLease

Please, check with `docker ps` that everything is OK.

```
CONTAINER ID   IMAGE                                   COMMAND                  CREATED          STATUS          PORTS                    NAMES
55b6d4bae625   inputoutput/cardano-wallet:2021.12.15   "bash -c ' ([[ $NETW…"   19 seconds ago   Up 17 seconds   0.0.0.0:8090->8090/tcp   wallet
c133c787c19c   inputoutput/cardano-node:1.32.1         "entrypoint"             10 minutes ago   Up 10 minutes                            node
```
Great! Now we have our REST API Wallet running on ` port 8090`. Check our `cardano-wallet` on `http://localhost:8090/v2/network/information`.

### Managing our Wallet from `cardano-wallet` CLI

Let's go inside the Wallet Docker Container bash:
```
docker exec -it wallet bash
```
```
bash-4.4# cardano-wallet --help

...
> Available commands:
>  serve                    Serve API that listens for commands/actions.
>  recovery-phrase          About recovery phrases
>  key                      About public/private keys
>  wallet                   About wallets
>  address                  About addresses
>  transaction              About transactions
>  network                  About the network
>  stake-pool               About stake pools
>  version                  Show the program's version.

```
Now we can manage `cardano-wallet` through `CLI`, (e.g.):

```
cardano-wallet network information

Ok.
{
    "network_tip": {
        "time": "2022-01-21T10:02:32Z",
        "epoch_number": 182,
        "absolute_slot_number": 48390136,
        "slot_number": 135736
    },
    "node_era": "alonzo",
    "node_tip": {
        "height": {
            "quantity": 3255176,
            "unit": "block"
        },
        "time": "2022-01-21T10:02:00Z",
        "epoch_number": 182,
        "absolute_slot_number": 48390104,
        "slot_number": 135704
    },
    "sync_progress": {
        "status": "ready"
    },
    "next_epoch": {
        "epoch_start_time": "2022-01-24T20:20:16Z",
        "epoch_number": 183
    }
}

```
### Creating our first Wallet

To create a wallet we must first generate a wallet recovery phrase using the `cardano-wallet` in the CLI:
```
cardano-wallet recovery-phrase generate
```

You should get a 24-word mnemonic seed in return similar to this:

```
output theme piano finish mule mean harbor match essence long sibling army crew bind year also icon live street pink peanut gym trap onion
```
Execute the command `cardano-wallet wallet create from-recovery-phrase [--port INT] WALLET_NAME [--address-pool-gap INT]` where:
- `--port INT` port used for serving the wallet API. (default: 8090)
- `--address-pool-gap INT` number of unused consecutive addresses to keep track of. (default: 20)

```
cardano-wallet wallet create from-recovery-phrase NovapagoWallet

> Please enter a 15–24 word recovery phrase:
...
```
We continue with the creation process:

```
> Please enter a 15–24 word recovery phrase: output theme piano finish mule mean harbor match essence long sibling army crew bind year also icon live street pink peanut gym trap onion
> (Enter a blank line if you do not wish to use a second factor.)
> Please enter a 9–12 word second factor:
> Please enter a passphrase: **************
> Enter the passphrase a second time: **************
> Ok.
{
    "passphrase": {
        "last_updated_at": "2022-01-21T10:27:12.28097Z"
    },
    "address_pool_gap": 20,
    "state": {
        "status": "syncing",
        "progress": {
            "quantity": 0,
            "unit": "percent"
        }
    },
    "balance": {
        "reward": {
            "quantity": 0,
            "unit": "lovelace"
        },
        "total": {
            "quantity": 0,
            "unit": "lovelace"
        },
        "available": {
            "quantity": 0,
            "unit": "lovelace"
        }
    },
    "name": "NovapagoWallet",
    "delegation": {
        "next": [],
        "active": {
            "status": "not_delegating"
        }
    },
    "id": "0e8e6efe324f5b12b9539af8c5f069e09544cd4f",
    "tip": {
        "height": {
            "quantity": 0,
            "unit": "block"
        },
        "time": "2019-07-24T20:20:16Z",
        "epoch_number": 0,
        "absolute_slot_number": 0,
        "slot_number": 0
    },
    "assets": {
        "total": [],
        "available": []
    }
}

```

Now our Wallet with 20 address (by default) is created, just we need to save _**Wallet Id**_: `0e8e6efe324f5b12b9539af8c5f069e09544cd4f`

### Managing our wallet from Web API

We have a Cardano Wallet connected to the `testnet` that we can manage through `CLI` or `Web API`. 

In this case, we can check [`API Docs`](https://input-output-hk.github.io/cardano-wallet/api/edge/) and try on our browser how we can access to information. 

In our browser (e.g.):

```
http://localhost:8090/v2/wallets/
```
We can see `.json` about our Wallet:

```
    [
    {
    "passphrase":{
        "last_updated_at":"2022-01-21T10:27:12.28097Z"},
    "address_pool_gap":20,
    "state":{
        "status":"ready"},
    "balance":{
        "reward":{
            "quantity":0,
            "unit":"lovelace"},
    "total":{
        "quantity":0,
        "unit":"lovelace"},
    "available":{
        "quantity":0,
        "unit":"lovelace"}
        },
    "name":"NovapagoWallet",
    "delegation":{
        "next":[],
        "active":{
            "status":"not_delegating"
            }
        },
    "id":"0e8e6efe324f5b12b9539af8c5f069e09544cd4f",
    "tip":{
        "height":{
            "quantity":3255236,"unit":"block"
            },
    "time":"2022-01-21T10:40:42Z",
    "epoch_number":182,
    "absolute_slot_number":48392426,
    "slot_number":138026
    },
    "assets":{
        "total":[],
        "available":[]
            }
        }
    ]
```
Copy _Wallet ID_: `0e8e6efe324f5b12b9539af8c5f069e09544cd4f` and operate whitin the wallet:

- List `addresses` use `https://localhost:8090/v2/wallets/{walletId}/addresses`

```
http://localhost:8090/v2/wallets/0e8e6efe324f5b12b9539af8c5f069e09544cd4f/addresses

```
```
[
{"state":"unused","id":"addr_test1qqdndr4z5nxxpq4n569v7686mmpamxr69q7zf2uvk5zqmj4nt9tsg886753rg898yj7jwh32kqynrcxkspts87fne3wqu7gvzs","derivation_path":["1852H","1815H","0H","0","0"]},
{"state":"unused","id":"addr_test1qpkrxsleg0medjmu89m4kkmcunsk9xf7zlt4uxtfse0rl69nt9tsg886753rg898yj7jwh32kqynrcxkspts87fne3wqfztk4w","derivation_path":["1852H","1815H","0H","0","1"]},
{"state":"unused","id":"addr_test1qpqty6qv46rn57uk5erz8zhtrrtxvyegh6yy4nalq5yt94dnt9tsg886753rg898yj7jwh32kqynrcxkspts87fne3wqz9p7lp","derivation_path":["1852H","1815H","0H","0","2"]},
{"state":"unused","id":"addr_test1qzcjy6p3ue4za9cnw4xl3m7q3wd0tkqk75j737ufprmr77dnt9tsg886753rg898yj7jwh32kqynrcxkspts87fne3wqxtgyex","derivation_path":["1852H","1815H","0H","0","3"]},
{"state":"unused","id":"addr_test1qzqz4kcvwqphpam5k09xmkeyxyw7wup6ghs2gf4z7ld6dqdnt9tsg886753rg898yj7jwh32kqynrcxkspts87fne3wqke87ce","derivation_path":["1852H","1815H","0H","0","4"]},
{"state":"unused","id":"addr_test1qrx8kdp395rcx0rsyhf57zmta666l0nxd88n4pdgkcy5mvdnt9tsg886753rg898yj7jwh32kqynrcxkspts87fne3wqt5pq79","derivation_path":["1852H","1815H","0H","0","5"]},
{"state":"unused","id":"addr_test1qzse8ml0rpsd8adjujzmgf03ujp87amm3j8t9pkamjxxm89nt9tsg886753rg898yj7jwh32kqynrcxkspts87fne3wq0ttncr","derivation_path":["1852H","1815H","0H","0","6"]},
{"state":"unused","id":"addr_test1qzhlyv0rlh53a2pt6g2etnevg4muhx3sfy63rhzm7ye2rnant9tsg886753rg898yj7jwh32kqynrcxkspts87fne3wq3gwmj9","derivation_path":["1852H","1815H","0H","0","7"]},
{"state":"unused","id":"addr_test1qrlp0v2v0yfmd88gn60mjfhwsx2ga8kpe8vudxn9hl9r2p9nt9tsg886753rg898yj7jwh32kqynrcxkspts87fne3wqcfhnaf","derivation_path":["1852H","1815H","0H","0","8"]},
{"state":"unused","id":"addr_test1qrwg7ldy36vqh044uyr6ayngsxctl8ahejktmhktz9eccr9nt9tsg886753rg898yj7jwh32kqynrcxkspts87fne3wqndjw8a","derivation_path":["1852H","1815H","0H","0","9"]},
{"state":"unused","id":"addr_test1qqy4cmrkm0m7hseqvqvat8c8lnr3e7fe9x5j0yszu3ygre9nt9tsg886753rg898yj7jwh32kqynrcxkspts87fne3wqdldfyz","derivation_path":["1852H","1815H","0H","0","10"]},
{"state":"unused","id":"addr_test1qrmtkr9wz4kmy9gumv5h33326vchdy3tsu2nqdxt4wgf864nt9tsg886753rg898yj7jwh32kqynrcxkspts87fne3wq9xxwre","derivation_path":["1852H","1815H","0H","0","11"]},
{"state":"unused","id":"addr_test1qq4zgvp98p0xktja97amva2k0ghw7lpzye26aymz6gavhg4nt9tsg886753rg898yj7jwh32kqynrcxkspts87fne3wq6v08d5","derivation_path":["1852H","1815H","0H","0","12"]},
{"state":"unused","id":"addr_test1qqyeywaqt668ke08zwuc3wnpgz50fu0ger5eg20f2zsc5s9nt9tsg886753rg898yj7jwh32kqynrcxkspts87fne3wqz834x0","derivation_path":["1852H","1815H","0H","0","13"]},
{"state":"unused","id":"addr_test1qqgekmfdeeh38ttzxpmsg69qy5pw8vtfet89lzctkhlpg24nt9tsg886753rg898yj7jwh32kqynrcxkspts87fne3wq82ddja","derivation_path":["1852H","1815H","0H","0","14"]},
{"state":"unused","id":"addr_test1qrt7wpdz9l974y2ea4zy93x949wcdx3drzvn5yz62vly2l4nt9tsg886753rg898yj7jwh32kqynrcxkspts87fne3wq945a42","derivation_path":["1852H","1815H","0H","0","15"]},
{"state":"unused","id":"addr_test1qrv4jlp5qqryz0s8k4fenpz30wqu7we428t26xvwg5euqudnt9tsg886753rg898yj7jwh32kqynrcxkspts87fne3wq4e2mqf","derivation_path":["1852H","1815H","0H","0","16"]},
{"state":"unused","id":"addr_test1qqdv572ty2p95rctjv2hfm003mujdvhpqn6ynrvsr3f6ckant9tsg886753rg898yj7jwh32kqynrcxkspts87fne3wq0d56je","derivation_path":["1852H","1815H","0H","0","17"]},
{"state":"unused","id":"addr_test1qql82r86r5yj9lt8mta2rghwvvtydgxjx00nuhyuggznmf9nt9tsg886753rg898yj7jwh32kqynrcxkspts87fne3wqh5kpen","derivation_path":["1852H","1815H","0H","0","18"]},
{"state":"unused","id":"addr_test1qp7h8svq3l6tq5akqwg8n0vvt9jayujyrzn596wu608mtyant9tsg886753rg898yj7jwh32kqynrcxkspts87fne3wqejrknm","derivation_path":["1852H","1815H","0H","0","19"]}]
```

As you can see, all addresses are `unused`, so if we request a `used` address we don't receive nothing.

Let's go and take some funds for one of them. Copy Address Id (`addr_test1qqdndr4z5nxxpq4n569v7686mmpamxr69q7zf2uvk5zqmj4nt9tsg886753rg898yj7jwh32kqynrcxkspts87fne3wqu7gvzs`) and request some tADA on [Faucet](https://testnets.cardano.org/en/testnets/cardano/tools/faucet/).

```
http://localhost:8090/v2/wallets/0e8e6efe324f5b12b9539af8c5f069e09544cd4f/addresses/?state=used
```
As you can see, our selected address is now `used`.
```
[{"state":"used","id":"addr_test1qqdndr4z5nxxpq4n569v7686mmpamxr69q7zf2uvk5zqmj4nt9tsg886753rg898yj7jwh32kqynrcxkspts87fne3wqu7gvzs","derivation_path":["1852H","1815H","0H","0","0"]}]
```
 
Remember that you can try any others command following [`API Docs`](https://input-output-hk.github.io/cardano-wallet/api/edge/). A powerful tool to connect your Cardano `Wallet` to any services in your frontend.

-----------

### Attribution




Created By: 

[<img align="left" src="https://styles.redditmedia.com/t5_4bhj33/styles/communityIcon_u389y4dxqov61.png" alt="Novapago" style="height: 100px; width:130px;"/>](https://novapago.com/)\
[Novapago](https://novapago.com/) 
\
\
\
\
At the frame of:
\
[<img align="left" src="https://upload.wikimedia.org/wikipedia/commons/2/21/Universidad_de_M%C3%A1laga.jpg" alt="UMA" style="height: 150px; width:150px;"/>](https://www.uma.es)\
\
[II Blockchain Technologies Course Final Project](https://www.nics.uma.es/Blockchain/)\
[UMA Nics Lab](https://www.nics.uma.es/)\
[UMA](https://www.uma.es/)\
\
\
Student: [José Manuel Guzmán](https://es.linkedin.com/in/josemanguzman)\
Mentors: [Rubén González](https://es.linkedin.com/in/gonzalezgomezruben?trk=public_profile_samename-profile), [Adrián Portugués](https://es.linkedin.com/in/adrian-portugues-mas-17a7a1bb?trk=public_profile_browsemap), [Silvia Briones](https://linkedin.com/) and [Francisco López](https://es.linkedin.com/in/francisco-manuel-lopez?trk=public_profile_browsemap)

Period: December 2021 / January 2022