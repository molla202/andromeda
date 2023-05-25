# andromeda

Andromeda discord : https://discord.gg/b5Fte2KV

![maxresdefault](https://github.com/molla202/andromeda/assets/91562185/b4d923a3-5324-4844-be78-1f2590d614d0)



# kütüphane ve güncellemeleri yapıyoruz
```
sudo apt update && sudo apt upgrade -y
sudo apt install curl git wget htop tmux build-essential jq 
make lz4 gcc unzip -y
```
```
# go kurulumu
cd $HOME
! [ -x "$(command -v go)" ] && {
VER="1.19.3"
wget "https://golang.org/dl/go$VER.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$VER.linux-amd64.tar.gz"
rm "go$VER.linux-amd64.tar.gz"
[ ! -f ~/.bash_profile ] && touch ~/.bash_profile
echo "export PATH=$PATH:/usr/local/go/bin:~/go/bin" >> ~/.bash_profile
source $HOME/.bash_profile
}
[ ! -d ~/go/bin ] && mkdir -p ~/go/bin
```
```
# cüzdan adnız kullanıcı adınız kısmını değiştirelim port yapılandırması 31 değiştirmek isterseniz baska yazınız
echo "export WALLET="cüzdan adınız"" >> $HOME/.bash_profile
echo "export MONIKER="kullanızı adınız"" >> $HOME/.bash_profile
echo "export ANDROMEDA_CHAIN_ID="galileo-3"" >> $HOME/.bash_profile
echo "export ANDROMEDA_PORT="31"" >> $HOME/.bash_profile
source $HOME/.bash_profile
```
```
# binary indiriyoruz
cd $HOME
rm -rf andromedad
git clone https://github.com/andromedaprotocol/andromedad.git
cd andromedad
git checkout galileo-3-v1.1.0-beta1
make install
```
```
# node kurulumu kullanıcı adınız kısmına moniker adınızı yazınız
andromedad config node tcp://localhost:${ANDROMEDA_PORT}657
andromedad config keyring-backend os
andromedad config chain-id galileo-3
andromedad init "kullanızı adınız" --chain-id galileo-3
```
```
# genesis ve adresbook ekleme
wget -O $HOME/.andromedad/config/genesis.json https://testnet-files.itrocket.net/andromeda/genesis.json
wget -O $HOME/.andromedad/config/addrbook.json https://testnet-files.itrocket.net/andromeda/addrbook.json
```
```
# seed ve peer ekeleme
SEEDS="5cfce64114f98e29878567bdd1adbebe18670fc6@andromeda-testnet-seed.itrocket.net:30656"
PEERS="239eeebb9c4c32f5ca91b22322fed2486aee01b5@andromeda-testnet-peer.itrocket.net:29656"
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.andromedad/config/config.toml
```
```
# port yapılandırması
sed -i.bak -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${ANDROMEDA_PORT}317\"%;
s%^address = \":8080\"%address = \":${ANDROMEDA_PORT}080\"%;
s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${ANDROMEDA_PORT}090\"%; 
s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${ANDROMEDA_PORT}091\"%; 
s%^address = \"0.0.0.0:8545\"%address = \"0.0.0.0:${ANDROMEDA_PORT}545\"%; 
s%^ws-address = \"0.0.0.0:8546\"%ws-address = \"0.0.0.0:${ANDROMEDA_PORT}546\"%" $HOME/.andromedad/config/app.toml
```
```
# port yapılandırması
sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${ANDROMEDA_PORT}658\"%; 
s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://0.0.0.0:${ANDROMEDA_PORT}657\"%; 
s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${ANDROMEDA_PORT}060\"%;
s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${ANDROMEDA_PORT}656\"%;
s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${ANDROMEDA_PORT}656\"%;
s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${ANDROMEDA_PORT}660\"%" $HOME/.andromedad/config/config.toml
```
```
# pruning
sed -i -e "s/^pruning *=.*/pruning = \"nothing\"/" $HOME/.andromedad/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"100\"/" $HOME/.andromedad/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"50\"/" $HOME/.andromedad/config/app.toml
```
```
# gas ayarları ve prometeus ayarları index kapama
sed -i 's/minimum-gas-prices =.*/minimum-gas-prices = "0.0uandr"/g' $HOME/.andromedad/config/app.toml
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.andromedad/config/config.toml
sed -i -e "s/^indexer *=.*/indexer = \"null\"/" $HOME/.andromedad/config/config.toml
```
```
# servis olusturuyoruz
sudo tee /etc/systemd/system/andromedad.service > /dev/null <<EOF
[Unit]
Description=Andromeda node
After=network-online.target
[Service]
User=$USER
WorkingDirectory=$HOME/.andromedad
ExecStart=$(which andromedad) start --home $HOME/.andromedad
Restart=on-failure
RestartSec=5
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
```
```
# snap atıyoruz
andromedad tendermint unsafe-reset-all --home $HOME/.andromedad
curl https://testnet-files.itrocket.net/andromeda/snap_andromeda.tar.lz4 | lz4 -dc - | tar -xf - -C $HOME/.andromedad
```
```
# servisleri aktif edip başlatıyoruz
sudo systemctl daemon-reload
sudo systemctl enable andromedad
sudo systemctl restart andromedad && sudo journalctl -u andromedad -f
```
```
# cüzdan olusturma ( yukarda cüzdan adı tanımladık değiştirmenize gerek yok)

andromedad keys add $WALLET
```
```
# olan cüzdanı import etme
andromedad keys add $WALLET --recover
```
```
 # validator olusturuyoruz (moniker kısmını doldurunuz cüzdan adı yazmanıza gerek yok en ustte zaten tanımladık kodla o adı çağırıyoruz zaten)                                                            
                                                                  
andromedad tx staking create-validator \
--amount 1000000uandr \
--from $WALLET \
--commission-rate 0.1 \
--commission-max-rate 0.2 \
--commission-max-change-rate 0.01 \
--min-self-delegation 1 \
--pubkey $(andromedad tendermint show-validator) \
--moniker "moniker-yazınız" \
--identity "" \
--details "" \
--chain-id galileo-3 \
--gas auto --gas-adjustment 1.5 \
-y
```
