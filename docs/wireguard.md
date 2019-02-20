# WireGuard 节点

## 安装 WireGuard

```shell
sudo add-apt-repository ppa:wireguard/wireguard
sudo apt update
sudo apt install wireguard -y
```

## 配置路由表

假设网卡名为`ens3`

```
sudo iptables -t filter -A FORWARD -i wg0 -j ACCEPT
sudo iptables -t filter -A FORWARD -o wg0 -m state --state RELATED,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -i wg0 -o ens3 -j ACCEPT
sudo iptables -A FORWARD -i ens3 -o wg0 -m state --state ESTABLISHED,RELATED -j ACCEPT
sudo iptables -t nat -A POSTROUTING -o ens3 -j MASQUERADE
```

## 生成密钥

```shell
umask 077 && wg genkey > private.key
wg pubkey < private.key > public.key
wg genpsk > preshared.key
```
其中 private key 写在配置文件里，public key 在 web 端增加服务器时填写

## 增加配置文件

```shell
mkdir -p /etc/wireguard
vi /etc/wireguard/wg0.conf
```
```
[Interface]
Address = 10.100.0.1/16 
PrivateKey = 8REGzY7PA3p81VN9KQ4mKM7d8oFZBu2wD7Pbs8ppPkW= 
ListenPort = 50000
```
```shell
chmod 777 -R /etc/wireguard
```

## 启动 WireGuard

```shell
sudo wg-quick up /etc/wireguard/wg0.conf
```

## WireGuard添加开机启动

```shell
systemctl start wg-quick@wg0.service
```

## 克隆S端

使用[此项目](https://github.com/gyteng/shadowsocks-manager-wireguard)作为 s 端即可
```shell
git clone https://github.com/gyteng/shadowsocks-manager-wireguard.git
cd shadowsocks-manager-wireguard
```
## 启动 S 端

```shell
node index --gateway 10.100.0.1
           --manager 0.0.0.0:6789
           --password 123456
           --interface wg0
           --db /your/data.json
```

## pm2后台管理

```shell
pm2 --name wg -f start node -x -- /root/shadowsocks-manager-wireguard/index.js --gateway 10.100.0.1 --manager 0.0.0.0:6789 --password 123456 --interface wg0 --db /your/data.json
```

