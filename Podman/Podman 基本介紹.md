# Podman 基本介紹

---

![](https://i.imgur.com/mjxUGpI.jpg)

![](https://i.imgur.com/gmOGwJx.jpg)

![](https://i.imgur.com/zyDmlcz.jpg)

```
sudo sh -c "echo 'deb http://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/xUbuntu_${VERSION_ID}/ /' > /etc/apt/sources.list.d/devel:kubic:libcontainers:stable.list"

wget -nv https://download.opensuse.org/repositories/devel:kubic:libcontainers:stable/xUbuntu_${VERSION_ID}/Release.key -O- | sudo apt-key add -

sudo apt update; sudo apt install podman -y

```

![](https://i.imgur.com/3rCiYo4.png)

```
sudo nano /etc/containers/registries.conf
sudo nano /etc/containers/registries.conf.d/000-shortnames.conf
```

![](https://i.imgur.com/dX87QfC.jpg)

![](https://i.imgur.com/ZEz2u2K.jpg)


###### tags: `Podman`
