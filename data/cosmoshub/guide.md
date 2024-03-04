### Update system and install build tools

```
sudo apt update
sudo apt-get install git curl build-essential make jq gcc snapd chrony lz4 tmux unzip bc -y
```

### Install Go

```
rm -rf $HOME/go
sudo rm -rf /usr/local/go
cd $HOME
curl https://dl.google.com/go/go1.20.5.linux-amd64.tar.gz | sudo tar -C/usr/local -zxvf -
cat <<'EOF' >>$HOME/.profile
export GOROOT=/usr/local/go
export GOPATH=$HOME/go
export GO111MODULE=on
export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin
EOF
source $HOME/.profile
go version
```

### Install Node

```
cd $HOME
rm -rf gaia
git clone https://github.com/cosmos/gaia.git
cd gaia
git checkout v14.1.0
make install
gaiad version
```

### Initialize Node
Replace NodeName with your own moniker.
```
gaiad init NodeName --chain-id=cosmoshub-4
```

### Download Genesis
```
curl -Ls https://ss.cosmos.nodestake.org/genesis.json > $HOME/.gaia/config/genesis.json 
```

### Download addrbook

```
curl -Ls https://ss.cosmos.nodestake.org/addrbook.json > $HOME/.gaia/config/addrbook.json
```

### Create Service

```
sudo tee /etc/systemd/system/gaiad.service > /dev/null <<EOF
[Unit]
Description=gaiad Daemon
After=network-online.target
[Service]
User=$USER
ExecStart=$(which gaiad) start
Restart=always
RestartSec=3
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
sudo systemctl daemon-reload
sudo systemctl enable gaiad
```

### Download Snapshot(optional)
```
SNAP_NAME=$(curl -s https://ss.cosmos.nodestake.org/ | egrep -o ">20.*\.tar.lz4" | tr -d ">")
curl -o - -L https://ss.cosmos.nodestake.org/${SNAP_NAME}  | lz4 -c -d - | tar -x -C $HOME/.gaiad
```

### Launch Node

```
sudo systemctl restart gaiad
journalctl -u gaiad -f
```
