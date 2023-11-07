# Manntrachain
Guide to install validator - Mantrachain

Install
Minimum Hardware Requirements 
2CPU 4RAM 100GB
Recommended Hardware Requirements 
4CPU 8RAM 200GB
Configure OS

1. Update system.
sudo apt-get update & sudo apt upgrade

3. Create new user.
Create a new user that will be used to run the node and add it to sudo-ers:
sudo useradd -s /bin/bash -d /home/mantra/ -m -G sudo mantra
su mantra

5. Download and install binary.
Create a bin folder in $HOME folder.  By default, all binaries in this folder are globally executable for the logged in user.
mkdir -p $HOME/go/bin/
Get the MANTRA Chain binary from the MANTRA GitHub repository
wget https://github.com/MANTRA-Finance/public/raw/main/mantrachain-testnet/mantrachaind-linux-amd64.zip
Install an unzip utility and unzip the binary archive.
wget https://github.com/MANTRA-Finance/public/raw/main/mantrachain-testnet/mantrachaind-linux-amd64.zip
unzip mantrachaind-linux-amd64.zip

7. Install CosmWasm Library
More information can be found in here: ​
sudo wget -P /usr/lib https://github.com/CosmWasm/wasmvm/releases/download/v1.3.0/libwasmvm.x86_64.so
mv mantrachaind $HOME/go/bin/
8. Initialise `mantrachaind`
mantrachaind init "moniker" --chain-id mantrachain-1
curl -Ls https://ss-t.mantra.nodestake.top/genesis.json > $HOME/.mantrachain/config/genesis.json
curl -Ls https://ss-t.mantra.nodestake.top/addrbook.json > $HOME/.mantrachain/config/addrbook.json
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0uaum\"/" $HOME/.mantrachain/config/app.toml
Create systemD
sudo tee /etc/systemd/system/mantrachaind.service > /dev/null <<EOF
[Unit]
Description=entangle testnet node
After=network-online.target
[Service]
User=$USER
ExecStart=$(which mantrachaind) start
Restart=always
RestartSec=3
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
sudo systemctl daemon-reload
sudo systemctl enable mantrachaind
mantrachaind tendermint unsafe-reset-all
​
SNAP_NAME=$(curl -s https://ss-t.mantra.nodestake.top/ | egrep -o ">20.*\.tar.lz4" | tr -d ">")
curl -o - -L https://ss-t.mantra.nodestake.top/${SNAP_NAME}  | lz4 -c -d - | tar -x -C $HOME/.mantrachain
Start node
sudo systemctl restart mantrachaind
sudo journalctl -u mantrachaind -f --no-hostname -o cat

Create a wallet
mantrachaind keys add wallet

Faucet
https://faucet.testnet.mantrachain.io/

Create a validator
mantrachaind tx staking create-validator \
  --amount "1000000uaum" \
  --pubkey $(mantrachaind tendermint show-validator) \
  --moniker="name" \
  --identity"keybase" \
  --details="your details" \
  --website="your website" \
  --chain-id=mantrachain-1 \
  --commission-rate="0.05" \
  --commission-max-rate="0.20" \
  --commission-max-change-rate="0.01" \
  --min-self-delegation "1" \
  --gas-prices "0uaum" \
  --gas="auto" \
  --gas-adjustment="1.5" \
  --from wallet \
  -y

Get Validator info
mantrachaind status 2>&1 | jq .ValidatorInfo

Get denom info
mantrachaind q bank denom-metadata -oj | jq

Get sync status
mantrachaind status 2>&1 | jq .SyncInfo.catching_up

Get latest height
mantrachaind status 2>&1 | jq .SyncInfo.latest_block_height

Reset Node
mantrachaind tendermint unsafe-reset-all --home $HOME/.mantrachain --keep-addr-book

Delete Node
sudo systemctl stop mantrachaind && sudo systemctl disable mantrachaind && sudo rm /etc/systemd/system/mantrachaind.service && sudo systemctl daemon-reload && rm -rf $HOME/.mantrachain && sudo rm -rf $(which mantrachaind) 

Withdraw comission and rewards from your validator
mantrachaind tx distribution withdraw-rewards $(mantrachaind keys show wallet --bech val -a) --commission --from wallet --chain-id mantrachain-1 --gas-prices=0uaum --gas-adjustment 1.5 --gas "auto" -y 

Delegate to your validator
mantrachaind tx staking delegate $(mantrachaind keys show wallet --bech val -a) 1000000uaum --from wallet --chain-id mantrachain-1 --gas-prices=0uaum --gas-adjustment 1.5 --gas "auto" -y 

Delegate to other
mantrachaind tx staking delegate TO_VALOPER_ADDRESS 1000000uaum --from wallet --chain-id mantrachain-1 --gas-prices=0uaum --gas-adjustment 1.5 --gas "auto" -y 

Redelegate your stake to other validators
mantrachaind tx staking redelegate $(mantrachaind keys show wallet --bech val -a) TO_VALOPER_ADDRESS 1000000
