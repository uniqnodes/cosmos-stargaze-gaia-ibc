## Install Gaia
`git clone https://github.com/cosmos/gaia.git`  
`cd gaia`  
`git fetch origin --tags`  
`git checkout v4.2.0`  
`make install`  
`gaiad version` (v4.2.0 olmalı)  

`sudo nano ~/.bashrc`  
en alta şunları ekle  
```
export STARSD_NODE="tcp://localhost:26657"
export STARSD_CHAIN_ID="bellatrix-1"
export STARSD_TRUST_NODE="true"

export GAIAD_NODE="http://54.236.165.83:26657"
export GAIAD_CHAIN_ID="gaia-1a"
export GAIAD_TRUST_NODE="true"
```  

## Gaia Key  
`gaiad keys add <my_gaia_relayer_key_name> --recover`  
starsd validator key oluştururken verdiği seedi yapıştır.  

## Installing Cosmos Relayer  
`cd..`  
`git clone https://github.com/cosmos/relayer.git`  
`cd relayer`  
`make install`  
`rly config init`  
`sudo nano ~/.relayer/config/config.yaml`  
yaml içini bu şekilde değiştir  
```
global:
  api-listen-addr: :5183
  timeout: 5m
  light-cache-size: 20
chains:
- key: <my_bellatrix_relayer_key_name>
  chain-id: bellatrix-1
  rpc-addr: http://127.0.0.1:26657
  account-prefix: stars
  gas-adjustment: 1.5
  gas-prices: 0.025ustarx
  trusting-period: 48h
- key: <my_gaia_relayer_key_name>
  chain-id: gaia-1a
  rpc-addr: http://54.236.165.83:26657
  account-prefix: cosmos
  gas-adjustment: 1.5
  gas-prices: 0.025uatomz
  trusting-period: 48h
paths: {}
```  

## Prepare to transfer  
`rly keys add bellatrix-1` (seed ve adresi sakla)  
`rly keys add gaia-1a` (seed ve adresi sakla)  
`starsd tx bank send <my_bellatrix_relayer_key_name> <relayer-starsx-address>  1000000ustarx --gas-prices 0.025ustarx`  
`gaiad tx bank send <my_gaia_relayer_key_name> <relayer-atomz-address>  1000000uatomz --gas-prices 0.025uatomz`  
`rly light init bellatrix-1 -f`  
`rly light init gaia-1a -f`  
`rly paths generate bellatrix-1 gaia-1a transfer --port=transfer`  
`rly paths list -d` (hepsinde tik varsa transfer için hazır)  
`nohup rly start transfer &`  
`sudo tail -f nohup.out` (ilk 4 satırdan sonra bekler. 5.satır gelene kadar bekle.)  

## IBC Transfer  
`gaiad tx ibc-transfer transfer transfer channel-1 <validator-starsx-address> 1uatomz --from <my_gaia_relayer_key_name> --gas-prices 0.025uatomz`  
`starsd q bank balances <validator-starsx-address>`  
balances için şöyle bir veri varsa işlem tamamdır  
```
- amount: "1"
  denom: ibc/1EA781XDD09B2F0FXDE3FACAD9BC61A73BAB6F09X7143C282B6XE231C51DX03X
```  

* Kanıtlar onchain yapılıyor başka bir işleme gerek yok.
