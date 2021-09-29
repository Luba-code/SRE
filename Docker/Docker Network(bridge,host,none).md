# Docker Network(bridge,host,none)

---

![](https://i.imgur.com/toxx8NL.jpg)

![](https://i.imgur.com/eaXzbEC.jpg)

![](https://i.imgur.com/n2tJYn1.jpg)


`brctl show`

`sudo iptables -t nat -L | grep 172.17`

`VMware用的內網IP都是192.168.xxx.xxx,Docker用的是172.17.xxx.xxx`

![](https://i.imgur.com/llSI9cu.jpg)

![](https://i.imgur.com/9gMOiAV.jpg)

`docker run --rm busybox cat /etc/resolv.conf`

![](https://i.imgur.com/7RSQwSh.jpg)

上一篇有介紹到的主機port號可以連到裡面的container的port

`docker run --rm --name n1 -p 8080:80 -d nginx`

![](https://i.imgur.com/HnEwQEC.jpg)

![](https://i.imgur.com/kEqebQm.jpg)

```
docker network create mynet1
docker network inspect --format='{{range .IPAM.Config}}{{.Subnet}}{{end}}'  mynet1
docker network create --driver=bridge --subnet=192.168.166.0/24 --gateway=192.168.166.254  mynet2
docker network ls
brctl show
sudo iptables -t nat -L -n | grep MASQUERADE 
```
![](https://i.imgur.com/xbW6T58.jpg)

```
docker run --rm --net mynet1 --name a1 -h ssn763  alpine cat /etc/hosts
docker run --rm --net mynet2 --name a2 -h ssn764  alpine hostname -i
```

![](https://i.imgur.com/P64G0zO.jpg)

![](https://i.imgur.com/xxNij4A.jpg)

```
docker run --net mynet2 --ip=192.168.166.3 --name  a2  -h  ddg52  -idt alpine sh
docker  network  connect  mynet1  a2
docker  exec  a2 cat /etc/hosts
docker  exec  a2 cat /etc/resolv.conf
```

![](https://i.imgur.com/keAHROA.jpg)

```
docker run --net mynet1 --name a1 -h cg62  -itd alpine sh
docker  exec -it a2 sh
```

這裡是進去a2 ping a1

```
ping –c 4 cg62.mynet1
ping –c 4 cg62
ping –c 4 ddg52
ping -c 4 192.168.166.3
```

![](https://i.imgur.com/nl2zDrv.jpg)

ping a1和自己 也成功

![](https://i.imgur.com/C15gkGV.jpg)

![](https://i.imgur.com/BG1kJPc.jpg)

![](https://i.imgur.com/fEzOBtH.jpg)

![](https://i.imgur.com/pf1QxUJ.jpg)

```
docker run --rm  -it  --net=host  busybox  ifconfig
docker run --rm  -it  --net=host  busybox  hostname
sudo docker run --name=mybox --net=host -h mybox busybox hostname
```

![](https://i.imgur.com/9ZDOOJo.jpg)

```
docker run --name=n1 --net=host -itd nginx
curl http://localhost
```

![](https://i.imgur.com/Isq08ek.jpg)

![](https://i.imgur.com/TwntR3o.jpg)

![](https://i.imgur.com/8ABNScX.jpg)

```
sudo modprobe tun
sudo tunctl -b -u bigred
ifconfig tap0
```

![](https://i.imgur.com/hJUWttL.jpg)

```
$'#!/bin/bash
[ "$#" != 2 ] && echo "dknet ctn net" && exit 1

ifconfig $2 &>/dev/null
[ "$?" != "0" ] && echo "$2 not exist" && exit 1

x=$(docker inspect -f \'{{.State.Pid}}\' $1 2>/dev/null)
[ "$x" == "" ] && echo "$1 not exist" && exit 1

[ ! -d /var/run/netns ] && sudo mkdir -p /var/run/netns

if [ ! -f /var/run/netns/$x ]; then
   sudo ln -s /proc/$x/ns/net /var/run/netns/$x
   sudo ip link set $2 netns $x
fi

exit 0
```

![](https://i.imgur.com/GFBqrIJ.jpg)

![](https://i.imgur.com/0MhytsW.jpg)


###### tags: `Docker`
