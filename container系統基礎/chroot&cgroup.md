# chroot

---

* chroot=change root

![](https://i.imgur.com/o0PnEJu.png)

先建一個rootfd目錄，使用curl下載alpine根目錄的壓縮檔，
.gz是壓縮檔.tar打包檔
1. curl 除了可以下載網站檔案之外，也可以砍網頁
2. 打 `curl 網站名稱` 就可以砍那個網站的html 例: 

![](https://i.imgur.com/DPxrzDD.png)

繼續操作chroot前置作業，解壓縮檔案 tar xvf 把壓縮訊息導入垃圾桶，然後壓縮完檔案順便刪檔

![](https://i.imgur.com/9AtZjgY.png)

打`sudo chroot rootfs/ /bin/sh`

進來rootfs裡面的目錄你會發現裡面的目錄就是你壓縮mini包裡面的目錄，算是小版根目錄，檢查ps指令結果如圖

![](https://i.imgur.com/lp5sP62.png)

裡面會是空的因為ps這指令是查proc檔案內的資訊，下載來的mini包proc是空的，所以才會是什麼程序都沒有

* 假如你要在rootfs裡面架起你的網站，必須得有httpd這個指令，所以得去busybox官網打包完整的文件點Binaries那個

![](https://i.imgur.com/SUPVBLO.png)

找到busybox裡面的檔案有httpd

![](https://i.imgur.com/tWDgKhU.png)

找`busybox-x86_64` 用wget把載下來/的http這個服務加到~/rootfs/bin裡面 才能在 rootfs環境裡架起網站

![](https://i.imgur.com/o3rF96y.png)

![](https://i.imgur.com/Xl27HQX.png)

![](https://i.imgur.com/tj6ZoAz.png)

---

* 用chroot新增使用者規格

![](https://i.imgur.com/AcClPSE.png)

`sudo chroot --userspec=bigred:bigred  rootfs/ /bin/sh`指使用了bigred這使用者和群組進去rootfs

`whoami` 會跑出 uknow 因為在rootfs的帳號名單裡面，沒有bigred使用者，沒有就自己加入bigred再來看看，(一定要exit再用root權限進去增加使用者)

![](https://i.imgur.com/JF4qYTs.png)

加入完再次用 bigred(使用者):bigred(群組) 進入rootfs

![](https://i.imgur.com/3fNIU3g.png)

那這裡可以想一下，假如我用chroot使用bigred進入rootfs，在裡面架起一個網站，假如有駭客攻擊這個網站，那我離開了rootfs，在打diff指令就可以看駭客在我rootfs的網站動了什麼手腳。

---

# cgroup

![](https://i.imgur.com/g4UhAle.png)

---

![](https://i.imgur.com/9bILTxl.png)

看一下cgroup(controlgroup) module裡面的檔案有哪些，其中cpu，cpuset是用來給cpu操作的看一下

`sudo mkdir /sys/fs/cgroup/memory/demo`

我在cgroup的/memory下新增一個demo目錄，它的內容是

![](https://i.imgur.com/OtZdz3U.png)

再回到上一層看memory目錄裡的內容是

![](https://i.imgur.com/8KQmUSE.png)

### 兩個目錄內容一模一樣，因為cgroup模組有個特性，它產生的子目錄都會繼承上一層目錄的內容

* 以demo做練習使用cgroup/memory

![](https://i.imgur.com/Dpo1xwQ.png)

1. 寫一個程式，第一行先判斷有沒有demo目錄，沒有就幫忙建立一個，然後給定一個限定的大小給memory.lilmit_in_bytes，最重要的 `cat /dev/zero | head -c $1 |tail` (解釋在圖上)
2. 執行自己寫的程式，它就會跑82M這參數進去能通過就會自動換行，給的參數如果大小超出15000000(自己設定的)就會出現錯誤，幫你kill -2 

![](https://i.imgur.com/krJTdma.png)

---

* 這次以cpu.cpuset來做練習

![](https://i.imgur.com/8l9e3y6.png)

第一行看能用cpu的時間，share值越大使用的時間越多
後面我們一樣新增子目錄low

![](https://i.imgur.com/Dl6ia01.png)

我們重新echo一個share值給low 
同樣我們設定一個high給它一個很高的share值2048
我們再去設定要用哪個cpu

![](https://i.imgur.com/mpoz2vC.png)

因為我們虛擬機alpineCPU是2核所以我們有0、1兩個cpu，但我們自己建的first裡面是沒指定的，所以我們給它指定第一個使用

![](https://i.imgur.com/7SWqVHC.png)

輸入個yes指令讓它移到背景執行並且執行出的訊息都隱藏起來
把我剛剛兩個的yes指令給到我第一個cpu

![](https://i.imgur.com/IfBZs6G.png)

使用top看結果

![](https://i.imgur.com/KBpcMpw.png)

兩個都佔cpu 0 各50%左右的時間
但當我給它們一個套low一個套high
剛剛設定low=512 high=2048 512/(512+2048)差不多20% 2048/(512+2048)差不多80%

![](https://i.imgur.com/x517v0e.png)

再用top看結果

![](https://i.imgur.com/5ihZkkB.png)

cpu使用比率跟自己設的差不多
