<p style="font-size:14px" align="right">
<a href="https://t.me/Aerdropmobile" target="_blank">Join our telegram <img src="https://user-images.githubusercontent.com/50621007/183283867-56b4d69f-bc6e-4939-b00a-72aa019d1aea.png" width="30"/></a>
<a href="https://twitter.com/AlldiiR" target="_blank">Join our twitter <img src="https://user-images.githubusercontent.com/108946833/184274157-08210464-fa03-493d-b01c-2420c67a524f.jpg" width="30"/></a>
</p>
<p align="center">
  <img height="150" height="auto" src="https://user-images.githubusercontent.com/38981255/185550018-bf5220fa-7858-4353-905c-9bbd5b256c30.jpg">
</p>

# Tutorial Garap Node Point Network
## Spek VPS

|  Komponen |  Persyaratan Minimum |
| ------------ | ------------ |
| CPU  | 4 or more physical CPU cores  |
| RAM | At least 32GB of memory (RAM) |
| Penyimpanan  | At least 500GB of SSD disk storage |
| koneksi | At least 100mbps network bandwidth |

Sebelum Memulai jalanin NODE, kalian harus mempunyai dulu Faucet, dan isi form untuk melanjutkan ketahap berikutnya, dan add RPC XPOINT Ke metamask
- ISI Form : https://pointnetwork.io/testnet-form
```console 
Network Title: Point XNet Triton
RPC URL: https://xnet-triton-1.point.space/
Chain ID: 10721
SYMBOL: XPOINT
```

## Buat Nama Node & Setting vars
```
NODENAME=namanode
```
namanode Ganti Dengan nama Validator lu bang bebas

```
echo "export NODENAME=$NODENAME" >> $HOME/.bash_profile
if [ ! $WALLET ]; then
	echo "export WALLET=wallet" >> $HOME/.bash_profile
fi
echo "export EVMOS_CHAIN_ID=point_10721-1" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

## Update packages & Install dependencies

```
sudo apt update && sudo apt upgrade -y
sudo apt install curl build-essential git wget jq make gcc tmux -y
```

## Install Go

```
if ! [ -x "$(command -v go)" ]; then
  ver="1.18.2"
  cd $HOME
  wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
  sudo rm -rf /usr/local/go
  sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
  rm "go$ver.linux-amd64.tar.gz"
  echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> ~/.bash_profile
  source ~/.bash_profile
fi
```

## Download and build binaries

```
cd $HOME
git clone https://github.com/pointnetwork/point-chain && cd point-chain
git checkout xnet-triton
make install
```

## Install Node

```
evmosd config chain-id $EVMOS_CHAIN_ID
evmosd config keyring-backend file
```

## Init App

```
evmosd init $NODENAME --chain-id $EVMOS_CHAIN_ID
```

## Download genesis and addrbook

```
wget https://raw.githubusercontent.com/pointnetwork/point-chain-config/main/testnet-xNet-Triton-1/config.toml
wget https://raw.githubusercontent.com/pointnetwork/point-chain-config/main/testnet-xNet-Triton-1/genesis.json
mv config.toml genesis.json ~/.evmosd/config/
```

### Validasi:

```
evmosd validate-genesis
```

## Create Service

```
sudo tee /etc/systemd/system/evmosd.service > /dev/null <<EOF
[Unit]
Description=evmos
After=network-online.target

[Service]
User=$USER
ExecStart=$(which evmosd) start --home $HOME/.evmosd
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

## Running Node

```
sudo apt install screen
```

```
sudo ufw allow ssh
```

```
sudo ufw enable
```

```
screen -S point
```

```
sudo systemctl daemon-reload
sudo systemctl enable evmosd
sudo systemctl restart evmosd && sudo journalctl -u evmosd -f -o cat
```

**Kalo udah jalan langsung CTRL+AD**


## Buat dompet

Untuk membuat dompet baru Anda dapat menggunakan perintah di bawah ini Masukan Pharse Metamask Kalian dan Jangan lupa simpan mnemonicnya Validator

```
evmosd keys add $WALLET
```

(OPSIONAL) Untuk memulihkan dompet Anda menggunakan frase seed

```
evmosd keys add $WALLET --recover
```

Untuk mendapatkan daftar dompet saat ini

```
evmosd keys list
```

## Save Info Wallet

```
EVMOS_WALLET_ADDRESS=$(evmosd keys show $WALLET -a)
```
Masukan Pharse Wallet
```
EVMOS_VALOPER_ADDRESS=$(evmosd keys show $WALLET --bech val -a)
```
Masukan Pharse Wallet
```
echo 'export EVMOS_WALLET_ADDRESS='${EVMOS_WALLET_ADDRESS} >> $HOME/.bash_profile
echo 'export EVMOS_VALOPER_ADDRESS='${EVMOS_VALOPER_ADDRESS} >> $HOME/.bash_profile
source $HOME/.bash_profile
```

### Check Status (Jika Sudah False dan Token Sudah Landing baru Buat Validator)

```
evmosd status 2>&1 | jq .SyncInfo
```

### Check Saldo

```
evmosd query bank balances $POINT_WALLET_ADDRESS
```
Jika Command di atas Error `$POINT_WALLET_ADDRESS` menjadi `Address Kalian`

## Create Validator Nya

```
evmosd tx staking create-validator \
  --amount 1000000000000000000000apoint \
  --from $WALLET \
  --commission-max-change-rate "0.01" \
  --commission-max-rate "0.2" \
  --commission-rate "0.07" \
  --min-self-delegation "1000000000000000000000" \
  --pubkey $(evmosd tendermint show-validator) \
  --moniker $NODENAME \
  --chain-id $EVMOS_CHAIN_ID
```
## Delete Node (Command ini digunakan kalo udah selesai nodenya) 

```
sudo systemctl stop evmosd
sudo systemctl disable evmosd
sudo rm /etc/systemd/system/evmos* -rf
sudo rm $(which evmosd) -rf
sudo rm $HOME/.evmosd -rf
sudo rm $HOME/point-chain -rf
```
