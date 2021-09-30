# Docker Image

---

![](https://i.imgur.com/kCQaKvz.jpg)

![](https://i.imgur.com/Lf28k67.jpg)

```
echo 'FROM alpine
RUN echo "top.secret" > /password.txt
RUN rm /password.txt' > Dockerfile
```
![](https://i.imgur.com/fQFHnvT.jpg)

`docker build --no-cache -t myring  .`

最後的.就是指目前這個位置的目錄

![](https://i.imgur.com/xvJGKH2.jpg)

![](https://i.imgur.com/sj04qNJ.jpg)

![](https://i.imgur.com/hRSf0v9.jpg)

```
docker save myring > myring.tar
mkdir imglayers; tar -xf myring.tar -C imglayers/
tree imglayers/
```
![](https://i.imgur.com/bddoN7Q.jpg)

![](https://i.imgur.com/mzcn8hh.jpg)

![](https://i.imgur.com/605NDX8.jpg)

`cat imglayers/manifest.json | jq`

![](https://i.imgur.com/ap9xdcO.jpg)

![](https://i.imgur.com/nZJJkm1.jpg)

![](https://i.imgur.com/nFUMC5D.jpg)

![](https://i.imgur.com/PktrbTF.jpg)

![](https://i.imgur.com/3xdh12n.jpg)

立可白檔在container篇，有說到

![](https://i.imgur.com/0un0lon.jpg)

![](https://i.imgur.com/eEcfV0B.jpg)

```
echo 'package main
import (
	"fmt"
	"log"
	"net/http"
)
func handler(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintln(w, "Hello, 世界")
}
func main() {
	http.HandleFunc("/", handler)
	log.Fatal(http.ListenAndServe(":8888", nil))
}' > main.go
```

```
go mod init mygo
CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -o main
```

![](https://i.imgur.com/H3RIxEQ.jpg)

```
echo 'FROM scratch
ADD main /
CMD ["/main"] ' > Dockerfile
```

`docker build -t goweb  .`

![](https://i.imgur.com/tbTN7ke.jpg)

![](https://i.imgur.com/X8tUrK2.jpg)

![](https://i.imgur.com/FgzWRe0.jpg)

```
dktag golang | grep -E "^1.1.-*alpine$"
echo 'FROM golang:1.16-alpine AS build
WORKDIR /src/
COPY main.go /src/
RUN go mod init mygo && CGO_ENABLED=0 go build -o /bin/demo

FROM scratch
COPY --from=build  /bin/demo  /bin/demo
ENTRYPOINT ["/bin/demo"] ' > Dockerfile
docker build -t goweb .
```

dktag 主要是為了找版本，程式由陳老師所寫

```
#!/bin/bash
# Returns the tags for a given docker image.
# Based on http://stackoverflow.com/a/32622147/

print_help_and_exit() {
	name=`basename "$0"`
	echo "Usage:"
	echo "  ${name} alpine"
	echo "  ${name} phusion/baseimage"
	exit
}

repo="$1"

[ "$#" != 1  ] && print_help_and_exit
[ "$" = "-h" ] && print_help_and_exit
[ "$" = "-help" ] && print_help_and_exit
[ "$" = "--help" ] && print_help_and_exit

if [[ "${repo}" != */* ]]; then
	repo="library/${repo}"
fi

# v2 API does not list all tags at once, it seems to use some kind of pagination.
#url="https://registry.hub.docker.com/v2/repositories/${repo}/tags/"
##echo "${url}"
#curl -s -S "${url}" | jq '."results"[]["name"]' | sort

# v1 API lists everything in a single request.
url="https://registry.hub.docker.com/v1/repositories/${repo}/tags"
#echo "${url}"
curl -s -S "${url}" | jq '.[]["name"]' | sed 's/^"\(.*\)"$/\1/' | sort
```

![](https://i.imgur.com/gksLbzH.jpg)

![](https://i.imgur.com/aixDOz5.jpg)

`docker history busybox`

![](https://i.imgur.com/DzdAFSt.jpg)

![](https://i.imgur.com/ywijHmL.jpg)

```
echo 'FROM alpine:3.13.4
RUN apk update && apk upgrade && apk add --no-cache nano sudo wget curl \
    tree elinks bash shadow procps util-linux coreutils binutils findutils grep && \
    wget https://busybox.net/downloads/binaries/1.28.1-defconfig-multiarch/busybox-x86_64 && \
    chmod +x busybox-x86_64 && mv busybox-x86_64 bin/busybox1.28 && \
    mkdir -p /opt/www && echo "let me go" > /opt/www/index.html

CMD ["/bin/bash"] ' > base/Dockerfile
```

![](https://i.imgur.com/99bkflR.jpg)

`docker build --no-cache  -t alpine.base base/`

![](https://i.imgur.com/XWGvpdM.jpg)

![](https://i.imgur.com/dJf1Kty.jpg)

`docker run --rm --name b1 -d -p 80:80 alpine.base busybox1.28 httpd -f -h /opt/www`

![](https://i.imgur.com/c6orGjS.jpg)

```
echo $' 
FROM alpine.base
RUN apk update && \
    apk add --no-cache openssh-server tzdata && \
    # 設定時區
    cp /usr/share/zoneinfo/Asia/Taipei /etc/localtime && \
    ssh-keygen -t rsa -P "" -f /etc/ssh/ssh_host_rsa_key && \
    echo -e \'Welcome to Alpine 3.13.4\\n\' > /etc/motd && \ 
    # 建立管理者帳號 bigred   
    adduser -s /bin/bash -h /home/bigred -G wheel -D bigred && echo \'%wheel ALL=(ALL) NOPASSWD: ALL\' >> /etc/sudoers && \
    echo -e "bigred\\nbigred\\n" | passwd bigred &>/dev/null && [ "$?" == "0" ] && echo "bigred ok"
 
EXPOSE 22
 
ENTRYPOINT ["/usr/sbin/sshd"]
CMD ["-D"] ' > plus/Dockerfile 
```

![](https://i.imgur.com/PCol6j5.jpg)

![](https://i.imgur.com/P0bjCQT.jpg)

![](https://i.imgur.com/OCPy816.jpg)

`docker run  --rm --name s1 -h s1 -d -p 22100:22 alpine.plus`

![](https://i.imgur.com/77wS5ED.jpg)

![](https://i.imgur.com/AF9YEId.jpg)

![](https://i.imgur.com/zTl4QDQ.jpg)

```
docker save alpine.plus > alpine.plus.tar
docker rmi alpine.plus
docker load < alpine.plus.tar
```

![](https://i.imgur.com/3aROGw9.jpg)

![](https://i.imgur.com/EMfBpxS.jpg)

```
docker export s2 > s2.tar
cat s2.tar | docker import - alpine.plus &>/dev/null
```

![](https://i.imgur.com/gA9gnIX.jpg)

對照沒export前

![](https://i.imgur.com/TI3VTSC.jpg)

![](https://i.imgur.com/NinO0lh.jpg)

`docker run --name s2 -h s2 -d alpine.plus /usr/sbin/sshd -D`

![](https://i.imgur.com/DbUUF4e.jpg)


###### tags: `Docker`
