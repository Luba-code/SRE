# Process Security in Linux 

---

![](https://i.imgur.com/HJFdCTV.jpg)

![](https://i.imgur.com/kflfIYU.jpg)

![](https://i.imgur.com/6jJ0V7e.jpg)

```
cat /boot/config-$(uname -r) | grep CONFIG_SECCOMP
```

![](https://i.imgur.com/PBDxXsY.jpg)

![](https://i.imgur.com/GuYVrfE.jpg)

`strace -qcf mkdir /tmp/x`

![](https://i.imgur.com/e0BhIaq.jpg)

whitelist.go 

![](https://i.imgur.com/vVJMCHT.jpg)

![](https://i.imgur.com/QorAKzj.jpg)

![](https://i.imgur.com/BBQ7iNG.jpg)

![](https://i.imgur.com/CvZKGLx.jpg)

![](https://i.imgur.com/EuWOV56.jpg)

---

![](https://i.imgur.com/6bhEITj.jpg)

![](https://i.imgur.com/AJ2UWVE.jpg)

![](https://i.imgur.com/vewq8oU.jpg)

`sudo chown root myfile; sudo chmod 4755 myfile`

![](https://i.imgur.com/sy71jg7.jpg)


`sudo find / -user root -perm -4000 2>/dev/null | grep -E '^/bin|^/usr/bin'`

---

![](https://i.imgur.com/lSd3j1y.jpg)

![](https://i.imgur.com/Q9DmaSD.jpg)

`sudo setcap  cap_setuid+ep  /home/rbean/python3`

![](https://i.imgur.com/TjGo9n0.jpg)

```
getcap -r / 2>/dev/null
```

![](https://i.imgur.com/uZjuIW3.jpg)

![](https://i.imgur.com/YSGqLKu.jpg)

###### tags: `Container`
