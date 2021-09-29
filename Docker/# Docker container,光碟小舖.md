# Docker container,光碟小舖

---

![](https://i.imgur.com/tS1av67.png)

1. 從光碟小舖下載image
2. run這個image產生container，並在裡面做自己的改裝
3. 把這個container燒成image
4. 最後上傳回去自己的小舖

![](https://i.imgur.com/nWpHS8q.jpg)

![](https://i.imgur.com/M04WgTl.jpg)

![](https://i.imgur.com/Y58nav3.jpg)

`docker run --name b1 -it busybox  /bin/sh `

![](https://i.imgur.com/WjfZnE9.jpg)

![](https://i.imgur.com/GyAn2xV.jpg)

![](https://i.imgur.com/aOzLTv2.jpg)

![](https://i.imgur.com/mKroePT.jpg)

busybox的binary的載點

https://busybox.net/downloads/binariesbusybox.net/downloads/binaries

![](https://i.imgur.com/iRaX1wC.jpg)

![](https://i.imgur.com/BTfFBYO.jpg)

![](https://i.imgur.com/0qjrto1.jpg)

![](https://i.imgur.com/0PAm9wQ.jpg)

光碟小舖上的a1成功推上去

![](https://i.imgur.com/ixVa9m1.jpg)

![](https://i.imgur.com/9q7Nshu.jpg)

`docker run --name testa1 -d <docker.io的帳號>/a1 httpd -f -p 8888 -h www`

![](https://i.imgur.com/rqVwfuF.jpg)

`docker run -d --restart=always --name n1 nginx`

假如沒給container命名的話，docker會自動給你前4碼當這container名稱

![](https://i.imgur.com/q1OR4gc.jpg)

![](https://i.imgur.com/PlfUozP.jpg)

![](https://i.imgur.com/0fjiHAu.jpg)


###### tags: `Docker`
