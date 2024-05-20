# bonusblock
Testi teşvikli değil fakat ileriki aşamaları ödüllü olma ihtimali yüksek. İyi bir yatırım aldılar. Değerlendirmek gerekir diye düşündüm.
Twitter: https://twitter.com/bonus_block
Web Site: https://www.bonusblock.io/
## Sistem Gereksinimleri
```
4 CPU
8 RAM
400 SSD
```
Bunlar ekibin tavsiye ettiği, ama daha düşük bir sisteme de kurulabilir. Örneğin;
```
2 CPU
4 RAM
150 SSD
```
Her şey hazırsa haydi başlayalım
### Sistemi Güncelleyelim
```
sudo apt update
sudo apt-get install git curl build-essential make jq gcc snapd chrony lz4 tmux unzip bc -y
```
### Go Yükleyelim
```
rm -rf $HOME/go
sudo rm -rf /usr/local/go
cd $HOME

curl https://dl.google.com/go/go1.20.1.linux-amd64.tar.gz | sudo tar -C/usr/local -zxvf -
cat <<'EOF' >>$HOME/.profile
export GOROOT=/usr/local/go
export GOPATH=$HOME/go
export GO111MODULE=on
export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin
EOF

source $HOME/.profile
go version
```
## Nodu Kuralım
```
cd $HOME
rm -rf BonusBlock-chain/
git clone https://github.com/BBlockLabs/BonusBlock-chain

cd BonusBlock-chain/
git fetch --all
git pull --all
git tag v0.1.39
git checkout v0.1.39

make install

bonus-blockd version
```
## İnitalizasyon İşlemlerini Yapalım
```
Dikkat:monikerName kısmını kendi moniker adınızla değiştirin!!!
bonus-blockd init monikerName --chain-id=blocktopia-01

Genesis dosyasını indirelim:
curl -Ls https://ss-t.bonusblock.nodestake.top/genesis.json > $HOME/.bonusblock/config/genesis.json

Addrbook ekleyelim:
curl -Ls https://ss-t.bonusblock.nodestake.top/addrbook.json > $HOME/.bonusblock/config/addrbook.json
```
## Servis Dosyası Oluşturalım
``` 
sudo tee /etc/systemd/system/bonus-blockd.service > /dev/null << EOF
[Unit]
Description=Bonusblock Node
After=network-online.target
[Service]
User=$USER
ExecStart=$(which bonus-blockd) start
Restart=on-failure
RestartSec=10
LimitNOFILE=10000
[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl enable bonus-blockd
```
## Hızlı Senkronizasyon İçin Snapshot
```
SNAP_NAME=$(curl -s https://ss-t.bonusblock.nodestake.top/ | egrep -o ">20.*\.tar.lz4" | tr -d ">")
curl -o - -L https://ss-t.bonusblock.nodestake.top/${SNAP_NAME}  | lz4 -c -d - | tar -x -C $HOME/.bonusblock

sudo systemctl restart bonus-blockd
journalctl -u bonus-blockd -f
```
## Cüzdan Oluşturalım

Dikkat:<walletname> kısmına kendi cüzdan adınızı yazın!!!
```
bonus-blockd keys add <walletname>
```
### Linkten faucete gidip test tokeni alalım: [this](https://faucet.bonusblock.io/)

## Senkronizasyon durumumuzu kontrol edelim. Çıktının false olması gerekiyor. Eğer false çıktısı verdiyse validatör işlemine geçelim.
```
bonus-blockd status 2>&1 | jq .SyncInfo
```
## Çıktımız false ise validatör işlemlerini tamamlayalım
```
bonus-blockd tx staking create-validator \
--amount 900000ubonus \
--pubkey $(bonus-blockd tendermint show-validator) \
--moniker <yourMonikerName> \
--identity <yourKeybaseId> \
--details <yourDetails> \
--website <yourWebsite> \
--chain-id blocktopia-01 \
--commission-rate 0.05 \
--commission-max-rate 0.20 \
--commission-max-change-rate 0.01 \
--min-self-delegation 1 \
--from <yourWalletName> \
--gas-adjustment 1.4 \
--gas auto \
-y
```

