# t3rn-v2 Kurulumu

## Gerekli paketleri yükle

```bash
sudo apt update
sudo apt install -y curl wget tar
```

## Ana dizini oluştur
```bash
mkdir -p $HOME/t3rn
cd $HOME/t3rn
```

**Son sürümü veya v0.53.1 sürümünü seçerek devam edin.**

## Son sürümü indir

```bash
curl -s https://api.github.com/repos/t3rn/executor-release/releases/latest | \
grep -Po '"tag_name": "\K.*?(?=")' | \
xargs -I {} wget https://github.com/t3rn/executor-release/releases/download/{}/executor-linux-{}.tar.gz
```

#### Çıkart

```bash
tar -xzf executor-linux-*.tar.gz
```

#### İndirdileni temizle

```bash
rm executor-linux-*.tar.gz
```

#### Çalıştırma izni

```bash
chmod +x $HOME/t3rn/executor/executor/bin/executor
```

## v0.53.1 sürümü indir

```bash
wget https://github.com/t3rn/executor-release/releases/download/v0.53.1/executor-linux-v0.53.1.tar.gz
```

#### Çıkart

```bash
tar -xzf executor-linux-v0.53.1.tar.gz
```

#### İndirdileni temizle

```bash
rm executor-linux-*.tar.gz
```

#### Çalıştırma izni

```bash
chmod +x $HOME/t3rn/executor/executor/bin/executor
```

## Systemd servis dosyasını oluştur 

- `BURAYA_ÖZEL_ANAHTARINIZI_YAZIN` kısmını ve diğer düzenlemek istediğinizi yerleri unutmayın.

- Sağlayıcıların rpclerini kullanacaksanız bir altındaki servisi kullanın.

- Executor son versiyonu kullanacaksanız `Environment="RPC_ENDPOINTS={\"l2rn\":[\"L2RN-RPC\"],\"arbt\":[\"ARBT-RPC\"],\"bast\":[\"BAST-RPC\"],\"opst\":[\"OPST-RPC\"],\"unit\":[\"UNIT-RPC\"],\"blst\":[\"BLST-RPC\"]}"` şeklinde değiştirmeniz gerekiyor haberiniz olsun. Blast eklenmiş.

- Yine blast çalıştırmak istemiyorum diyorsanız `Environment="NETWORKS_DISABLED=blast-sepolia"` ekleyin.

```bash
sudo tee /etc/systemd/system/t3rn-executor.service > /dev/null <<EOF
[Unit]
Description=t3rn Executor Service
After=network.target
Wants=network-online.target

[Service]
Type=simple
User=$USER
Group=$USER
WorkingDirectory=$HOME/t3rn/executor/executor/bin
ExecStart=$HOME/t3rn/executor/executor/bin/executor

Environment="ENVIRONMENT=testnet"
Environment="LOG_LEVEL=debug"
Environment="LOG_PRETTY=false"
Environment="EXECUTOR_PROCESS_BIDS_ENABLED=true"
Environment="EXECUTOR_PROCESS_ORDERS_ENABLED=true"
Environment="EXECUTOR_PROCESS_CLAIMS_ENABLED=true"
Environment="EXECUTOR_ENABLE_BATCH_BIDING=true"
Environment="EXECUTOR_PROCESS_PENDING_ORDERS_FROM_API=true"
Environment="EXECUTOR_PROCESS_ORDERS_API_ENABLED=true"
Environment="EXECUTOR_MAX_L3_GAS_PRICE=1000"
Environment="PRIVATE_KEY_LOCAL=BURAYA_ÖZEL_ANAHTARINIZI_YAZIN"
Environment="RPC_ENDPOINTS={\"l2rn\":[\"https://b2n.rpc.caldera.xyz/http\"],\"arbt\":[\"https://arbitrum-sepolia.drpc.org\",\"https://sepolia-rollup.arbitrum.io/rpc\"],\"bast\":[\"https://base-sepolia-rpc.publicnode.com\",\"https://base-sepolia.drpc.org\"],\"opst\":[\"https://sepolia.optimism.io\",\"https://optimism-sepolia.drpc.org\"],\"unit\":[\"https://unichain-sepolia.drpc.org\",\"https://sepolia.unichain.org\"]}"

Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
EOF
```

- **VEYA TEK ÖZEL RPC KULLANACAKSAN** (tenderly-alchemy rpcleri gibi)

- `ARB-RPC` , `BASE-RPC` , `OP-RPC` , `UNI-RPC` yerine rpclerini yapıştır. `\` önemli silmeyin.

```bash
sudo tee /etc/systemd/system/t3rn-executor.service > /dev/null <<EOF
[Unit]
Description=t3rn Executor Service
After=network.target
Wants=network-online.target

[Service]
Type=simple
User=$USER
Group=$USER
WorkingDirectory=$HOME/t3rn/executor/executor/bin
ExecStart=$HOME/t3rn/executor/executor/bin/executor

Environment="ENVIRONMENT=testnet"
Environment="LOG_LEVEL=debug"
Environment="LOG_PRETTY=false"
Environment="EXECUTOR_PROCESS_BIDS_ENABLED=true"
Environment="EXECUTOR_PROCESS_ORDERS_ENABLED=true"
Environment="EXECUTOR_PROCESS_CLAIMS_ENABLED=true"
Environment="EXECUTOR_ENABLE_BATCH_BIDING=true"
Environment="EXECUTOR_PROCESS_PENDING_ORDERS_FROM_API=false"
Environment="EXECUTOR_PROCESS_ORDERS_API_ENABLED=false"
Environment="EXECUTOR_MAX_L3_GAS_PRICE=1000"
Environment="PRIVATE_KEY_LOCAL=BURAYA_ÖZEL_ANAHTARINIZI_YAZIN"
Environment="RPC_ENDPOINTS={\"l2rn\":[\"https://b2n.rpc.caldera.xyz/http\"],\"arbt\":[\"ARB-RPC\"],\"bast\":[\"BASE-RPC\"],\"opst\":[\"OP-RPC\"],\"unit\":[\"UNI-RPC\"]}"

Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
EOF
```

## Systemd'yi yenile

```bash
sudo systemctl daemon-reload
```

## Servisi etkinleştir

```bash
sudo systemctl enable t3rn-executor
```

## Servisi başlat

```bash
sudo systemctl start t3rn-executor
sudo journalctl -u t3rn-executor -f
```

## Durumu kontrol et

```bash
sudo systemctl status t3rn-executor
```

## Diğer komutlar:

Servis durumunu kontrol etmek için: `sudo systemctl status t3rn-executor`

Servisi durdurmak için:            `sudo systemctl stop t3rn-executor`

Servisi yeniden başlatmak için:    `sudo systemctl restart t3rn-executor`

Servis loglarını görmek için:      `sudo journalctl -u t3rn-executor -f`

Son olarak eğer çalıştırmak istemediğiniz ağ varsa servis'in bir alt satırına bunu ekleyebilirsiniz. Örnektir blast,arb,uni iptal eder. `Environment="NETWORKS_DISABLED=blast-sepolia,arbitrum-sepolia,unichain-sepolia"`

# t3rn-v2 Güncelleme Komutları

## Servisi durdur

```bash
sudo systemctl stop t3rn-executor
```

## Çalışma dizinine git

```bash
cd $HOME/t3rn
```

## Eski executor klasörünü yedekle veya aynı gün 2 güncelleme olursa sil

```bash
mv executor executor_backup_$(date +%Y%m%d)
```

veya

```bash
rm -rf executor
```

## Son sürümü indir

```bash
curl -s https://api.github.com/repos/t3rn/executor-release/releases/latest | \
grep -Po '"tag_name": "\K.*?(?=")' | \
xargs -I {} wget https://github.com/t3rn/executor-release/releases/download/{}/executor-linux-{}.tar.gz
```

## Çıkart

```bash
tar -xzf executor-linux-*.tar.gz
```

## İndirilen arşivi temizle

```bash
rm executor-linux-*.tar.gz
```

## Çalıştırma izinlerini ayarla

```bash
chmod +x $HOME/t3rn/executor/executor/bin/executor
```

## Servisi yeniden başlat

```bash
sudo systemctl start t3rn-executor
```

## Logları kontrol et
```bash
sudo journalctl -u t3rn-executor -f
```
