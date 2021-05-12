# Container 
---

* Container運作構造

![](https://i.imgur.com/fcArdf3.png)

1. 最常看到的貨櫃軟體 Docker 和應用的kubernetes

![](https://i.imgur.com/Jpd05UG.png)

### kubernetes: ![](https://i.imgur.com/ewQPcx9.png)

![](https://i.imgur.com/9IRq7oe.png)

1. 使用kernel的模組以下5個，slirp是早期以前的撥號網路，如果要用目前(2021)window的系統是做不出來的，除非安裝WLS

![](https://i.imgur.com/wKRTnGJ.png)

* Namespace隔離出來的application的構造

![](https://i.imgur.com/MZYbeKY.png)

*  看一下host上開啟的虛擬主機作業系統Namespace模組，開始使用container技術嘗試在虛擬機上隔離出電腦

![](https://i.imgur.com/Eoeos26.png)

![](https://i.imgur.com/x9nJUbj.png)

1. 使用unshare這個指令 隔離出一個電腦名稱的軟體貨櫃(uts)
2. 去看他namespace模組，只有一個uts
3. 改變自己這個軟體貨櫃的電腦名稱

* 在軟體貨櫃看PID

![](https://i.imgur.com/TrUihX0.png)


![](https://i.imgur.com/EnvNXuJ.png)

1. 把pid和fork再加進去，ps指令看下去還是原本主機系統的程序，因為是在自己虛擬機上隔離的program

* 加入mount(蓋掉)

![](https://i.imgur.com/HE6pk9C.png)

![](https://i.imgur.com/pvCdKjS.png)

1. 我們加了mount指令，把原本虛擬機的ps全部蓋掉了，所以ps指令才只有顯示兩個程序
2. 用mount 去看proc會有兩個一個是原本主機的，另一個是這軟體貨櫃的
3. 嘗試用root刪掉pid 1 會發現刪不掉，這代表在這軟體貨櫃裡root已經不是大哥了(權限不是最高)

* slirp廣播網路

![](https://i.imgur.com/GgF2o4P.png)

從container撥號給slirp4netns(中華電信)，讓它去幫你轉接網際網路
* eth0是中華電信高級網路卡(switch)

![](https://i.imgur.com/9Ez8Pag.png)

* 所以我們要去虛擬機安裝slirp和新增tao網卡

![](https://i.imgur.com/x0EhD5N.png)

![](https://i.imgur.com/PLUpY9C.png)

1. 再開一個ssh連進虛擬機用pstree或ps指令找到隔離出去的軟體貨櫃pid程序
2. 安裝slirp
3. 增加網卡ta0給被隔離的軟體貨櫃的ps 
4. 就可以回ps軟體貨櫃用curl 網站看看，不能用ping，因為slirp沒有ping，當時只能用打電話給中華電信，中華電信有個網卡接你的這支電話的號碼再接到switch(eth0)幫你連上網際網路，所以根本沒有ping 

---

* 結合之前的chroot，把軟體貨櫃的環境整合起來

![](https://i.imgur.com/RaDJvyC.png)

![](https://i.imgur.com/972saqX.png)

![](https://i.imgur.com/1kDTBsP.png)

![](https://i.imgur.com/46S97Xk.png)

1.之前練習chroot是用alpine官網裡的minirootfs根目錄，這次練習的rootfs是複製原本虛擬機的/bin/sh和它的相依檔，一樣隔離出軟體貨櫃，環境給它自己複製出的r2d2/rootfs，進到這個軟體貨櫃裡面，打各種指令會發現都失效了

![](https://i.imgur.com/yuLWB3g.png)

1.因為alpine大部分指令都是從busybox來，所以把busybox複製進來 
2. `/bin/busybox --install -s`打上這串指令，讓那些指令產生捷徑檔
3. 接下來打指令就可以使用了

![](https://i.imgur.com/hTqV0Ln.png)

---

* overlay2

![](https://i.imgur.com/smFmixi.png)

![](https://i.imgur.com/amIDRUn.png)

* 套用overlay2檔案格式，剛剛兩層都有both.txt檔，但是ls指令卻是看到upper層的both.txt，原因以圖來了解

![](https://i.imgur.com/kcdyV1N.jpg)

*  可以看出overlay2是由上往下看的方式，就算有同名的檔案，也是會看到的上層，而且最下面的那層是唯讀，不能做更改

![](https://i.imgur.com/8SAnqv5.png)

![](https://i.imgur.com/cNcW3cM.png)

*  拔掉upper的both.txt，ls -al merged/看不到both.txt，因為是看upper的both.txt，剛剛刪掉了所以沒有，再看ls -al upper/會有是因為both.txt變成一個white out型態，檔案權限變成c----------
*  merged被拔掉overlay2，就只是空資料夾，它就是被掛載的資料夾，實際上得操作都是在overlay2裡面進行，只要被掛載上去，就是對overlay2作業
-

