# stafi-kurulum
stafi

<sudo su
cd /root
sudo apt update && sudo apt upgrade -y
sudo apt install make clang pkg-config libssl-dev build-essential git jq ncdu bsdmainutils -y < "/dev/null">

cd $HOME
wget -O go1.18.2.linux-amd64.tar.gz https://go.dev/dl/go1.18.2.linux-amd64.tar.gz
rm -rf /usr/local/go && tar -C /usr/local -xzf go1.18.2.linux-amd64.tar.gz && rm go1.18.2.linux-amd64.tar.gz
echo 'export GOROOT=/usr/local/go' >> $HOME/.bashrc
echo 'export GOPATH=$HOME/go' >> $HOME/.bashrc
echo 'export GO111MODULE=on' >> $HOME/.bashrc
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> $HOME/.bashrc && . $HOME/.bashrc
go version

git clone --branch public-testnet-v3 https://github.com/stafihub/stafihub

cd $HOME/stafihub && make install

stafihubd init NodeName --chain-id stafihub-public-testnet-3

sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.01ufis\"/" $HOME/.stafihub/config/app.toml
sed -i '/\[grpc\]/{:a;n;/enabled/s/false/true/;Ta};/\[api\]/{:a;n;/enable/s/false/true/;Ta;}' $HOME/.stafihub/config/app.toml
PEERS="6e9d988bf9812b02c46dcec474591bd10f81916f@45.94.58.160:26656,06c57407aea673fca396b01581a2d92957d48c4a@149.102.143.60:26656,5a6d8e1904c88c9f72d35df63b15d14504aaf030@164.92.159.170:26656,5e88d0d6866cd2f386e885de6eb0a1e3bd4f45c5@38.242.237.130:26656,1eaff7a3defa35de2b29f28d4729317d783f606c@149.102.139.101:26656,724430a2cf42b94f5da6b24d4741c7418fefa24e@194.60.201.153:26656,aae1ac9ef12897d7dc8755240cbdc41ee1171a55@38.242.215.200:26656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.stafihub/config/config.toml

echo "[Unit]
Description=StaFiHub Node
After=network.target
[Service]
User=$USER
Type=simple
ExecStart=$(which stafihubd) start
Restart=on-failure
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target" > $HOME/stafihubd.service
sudo mv $HOME/stafihubd.service /etc/systemd/system
sudo tee <<EOF >/dev/null /etc/systemd/journald.conf
Storage=persistent
EOF
sudo systemctl restart systemd-journald
sudo systemctl daemon-reload
sudo systemctl enable stafihubd
sudo systemctl restart stafihubd

stafihubd keys add walletName

stafihubd keys add walletName --recover

stafihubd status 2>&1 | jq .SyncInfo


stafihubd tx staking create-validator \
--moniker="NodeName" \
--amount=48885000ufis \
--gas auto \
--fees=5000ufis \
--pubkey=$(stafihubd tendermint show-validator) \
--chain-id=stafihub-public-testnet-3 \
--commission-max-change-rate=0.01 \
--commission-max-rate=0.20 \
--commission-rate=0.10 \
--min-self-delegation=1 \
--from=WalletName \
--yes

Explorer link: https://testnet-explorer.stafihub.io/stafi-hub-testnet
