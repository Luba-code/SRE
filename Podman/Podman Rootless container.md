# Podman Rootless container

---

![](https://i.imgur.com/Q3tk6hR.jpg)

![](https://i.imgur.com/8zTXd3t.jpg)

`podman run --rm -d --publish 8080:80 --volume ${PWD}/html:/usr/share/nginx/html nginx`

![](https://i.imgur.com/kAndgBO.jpg)

![](https://i.imgur.com/CoTrNcN.jpg)

![](https://i.imgur.com/86ZbA3D.jpg)

![](https://i.imgur.com/BhwlCAi.jpg)

![](https://i.imgur.com/RpozNAF.jpg)

![](https://i.imgur.com/1RWIHtL.jpg)

```
podman network create --driver=bridge --subnet=192.168.188.0/24 --gateway=192.168.188.254  mynet2
cat /home/rbean/.config/cni/net.d/mynet2.conflist | grep '"subnet":' | tr -d ' '
```

![](https://i.imgur.com/nFga6jt.jpg)

![](https://i.imgur.com/Vn6F5Ij.jpg)

![](https://i.imgur.com/xprOZWA.jpg)

![](https://i.imgur.com/SYHScjB.jpg)

![](https://i.imgur.com/v9UzSAx.jpg)

```
podman pod create -n mypod
podman pod list
podman ps -a --pod
```

![](https://i.imgur.com/yoaxCLo.jpg)

![](https://i.imgur.com/BGgCxb4.jpg)

```
podman run -dt --pod mypod alpine 
podman ps --pod | grep mypod
podman run -d --pod mypod nginx
```

![](https://i.imgur.com/VwjObzz.jpg)

```
podman generate kube mypod -f mypod.yaml
head -n 10 mypod.yaml 
```

![](https://i.imgur.com/6RAy6fp.jpg)


![](https://i.imgur.com/SPVxcc4.jpg)

`podman play kube mypod.yaml`

![](https://i.imgur.com/k8coJ8i.jpg)

###### tags: `Podman`
