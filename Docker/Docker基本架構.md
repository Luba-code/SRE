# Docker

![](https://i.imgur.com/9TZmiYc.jpg)

* docker這個工具，主要是利用linux的核心技術產生很多個application container，主要利用以下幾個技術:
1. Namespace chroot cgroup overlay2 slirp4netns bridge 來產生application container
2. 那產生了Application container，這台軟體貨櫃，可以同時執行DataBase，WebApplication，DHCP/DNS/Email等Server，webapplication有很多種像淨銷存系統、會計系統等等，當一台軟體貨櫃發現不夠用的話，可以新增更多台軟體貨櫃，這時候就需要用到kubernetes了，用kubernetes來管理這些軟體貨櫃。

#  為什麼會有docker的工具的產生呢，主要目的就是為了降低Hunman errors!

* 那Human errors就是人的因素所產生的錯誤，只要降低了Human errors，就是在幫公司省成本，而且可以更穩定的使用Application container

---

* Docker

![](https://i.imgur.com/mMuIdmX.png)

* Dockerdaemom接收我們打的高階指令，做出container

![](https://i.imgur.com/5s4dRgZ.png)

* Docker的發展過程 emterprise edition(就是年年收顧問費)

![](https://i.imgur.com/WzkT6Bq.png)

* Dockerless這個詞的出現

![](https://i.imgur.com/sedpZjU.png)

* kubernetes已經可以不用透過docker，就可以完全控管container

![](https://i.imgur.com/LFKnsAZ.png)

* RedHat的Linux演變

![](https://i.imgur.com/JTjfDAS.png)

出現了Podman這個可以取代Docker的工具，就不會有Docker背景程序造成的網路漏洞

* Docker的反擊

![](https://i.imgur.com/WCuXZGd.png)

---

* runC:只負責產生container，產生完就消失的背景程序

![](https://i.imgur.com/x8Co9Wq.png)

* 準備container的檔案系統

![](https://i.imgur.com/ZuA5v4x.png)

* 看檔案系統的命令來源

![](https://i.imgur.com/wD6Dh9g.png)

* 下載runC

![](https://i.imgur.com/SG2aKwv.png)

* 讓runc產生設定檔並且去更改

![](https://i.imgur.com/u4Z7AIj.png)

可以看到執行`runc spec` 就會產生出container的設定檔，更改readonly 和 電腦名稱 

![](https://i.imgur.com/ggw1Ubb.png)

![](https://i.imgur.com/f8CWRTJ.png)

![](https://i.imgur.com/0rvbtIy.png)

* 開始使用runc產生container

![](https://i.imgur.com/necliS3.png)

* 可以看到hostname就是我們設定的bb8，那也有網路環境，只是沒網卡，ps看到第一個程序是我們使用貝殼的container，那就代表這是個application container

![](https://i.imgur.com/5iGWide.png)

![](https://i.imgur.com/CIf0KjN.png)

* 可以看到我們能在rootfs檔案系統下建立檔案，但不能刪除記憶體資訊，可以看到檔案室唯讀

---

* Containerd，就是管理container的一生

![](https://i.imgur.com/vKnhso9.png)

* 從dockerdaemom把高階命令翻譯給containerd，containerd會去抓IMAGE檔，產生出rootfs檔案系統目錄，和container設定檔，再由runc使用這兩個檔案產生出container。

![](https://i.imgur.com/rGsucT2.png)

實際操作頁面

![](https://i.imgur.com/4q3uhMn.png)

---

* Docker

![](https://i.imgur.com/N3kjfX0.png)

![](https://i.imgur.com/XCozIbu.png)

用pstree看docker有在背景執行，因為有設定rc-update docker

![](https://i.imgur.com/qB32BrH.png)

* 使用docker隔離出sleep程序為一個appliction container

![](https://i.imgur.com/gwQoBM1.png)

![](https://i.imgur.com/BMabBko.png)

用pstree可以看在container存在之時，shim這個程序是有再啟動的，目的是為了監控container的所有一切回傳給containerd，一旦移除掉container，就會跟著消失
