# Podman Rootful Container

---

![](https://i.imgur.com/SC7cWIE.jpg)

![](https://i.imgur.com/em4ceri.jpg)

`podman的預設IP是10.88.xxx.xxx,DNS server 是172.16.xxx.xxx`

![](https://i.imgur.com/IVaVMKN.jpg)

![](https://i.imgur.com/KhZv5KA.jpg)

```
sudo apt install bridge-utils
docker network create --driver=bridge --subnet=192.168.166.0/24 --gateway=192.168.166.254  mynet
cat /etc/cni/net.d/mynet.conflist | grep subnet
```

![](https://i.imgur.com/q80GE8s.jpg)

![](https://i.imgur.com/kUIidJK.jpg)

###### tags: `Podman`
