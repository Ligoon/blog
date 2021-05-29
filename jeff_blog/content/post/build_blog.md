---
title: "[Hugo] 用 Hugo 建立部落格"
date: 2021-05-28T20:51:10+08:00
thumbnailImagePosition: left
thumbnailImage: https://ppt.cc/fRsPKx
coverImage: https://ppt.cc/fKwEKx
coverMeta: out

---

這篇用來記錄如何在 Windows 環境中安裝 WSL(Windows Subsystem for linux) 以及相關套件，並且使用 WSL 架設 Hugo 部落格最後發布到 Github pages<!--more-->
Hugo 是用 GO 語言所編寫的靜態網頁生成器，用 Hugo 架站的好處是速度賊快，號稱世界上最快的網站構建框架，原本筆者最一開始(大概兩年前吧)是使用 jekyll 來架設，但因為環境更新重裝後整個壞掉，筆者安裝了兩天還是搞不定於是決定放棄 jekyll 並且移植到 Hugo，真心覺得 Hugo 安裝以及環境巨簡單且乾淨。

<!-- toc -->

## 在 windows 安裝 WSL 
### 啟用 Windows 內的 WSL 設定
在 \[開始] 搜尋 windows powershell，並按右鍵以系統管理員身分執行，輸入以下指令：(註: 如果只是要安裝 WSL1 可以重新啟動，如果要安裝 WSL2 可以先跳至更新 WSL2 (2021) 的步驟，稍後再重新啟動)
```
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
```
### 更新 WSL2 (2021)
如果只想安裝 version1，可以直接跳到安裝 linux distro的步驟。

原本 windows 的 WSL 為 version1，近期改版成 version2，接下來紀錄如何升級到 WSL2 或安裝 WSL2。首先一樣在 windows powershell 執行以下指令啟動虛擬化功能:
```
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```
執行完後重新啟動，並且至[安裝教學](https://docs.microsoft.com/zh-tw/windows/wsl/install-win10)裡的步驟 4 下載 linux 套件，接著在 powershell 執行以下指令來設定之後安裝的 WSL 都是 version2:
```
wsl --set-default-version 2
```
之後就可以跳至下一個步驟安裝 linux distro

筆者原本之前安裝的是 WSL1 Ubuntu，經過一整天的踩雷，最後重新安裝了另一個版本的 linux distro (Ubuntu-20.04)，因為不知為何就是無法升級成 WSL2，總之以下紀錄採雷過程可能用到的指令:
(註:解除註冊後需要再去 Windows 設定->應用程式，來刪除 Ubuntu 比較乾淨)
```
wsl -l -v                   # 顯示目前電腦裡安裝的 wsl 清單以及版本
wsl --unregister Ubuntu     # 解除 distro 註冊(Ubuntu 為第一個指令顯示的 WSL 名字)
wsl --set-version Ubuntu 2  # 升級版本(Ubuntu 為第一個指令顯示的 WSL 名字)
```


### 安裝 linux distro
到 Microsoft Store 選擇適合的 WSL 散發套件，筆者選擇安裝 Ubuntu 20.04 LTS

![](https://i.imgur.com/JmH28qf.jpg)

安裝完成後開啟 App，第一次進入會需要設定使用者名稱以及密碼。除了使用 Ubuntu App 進入外，也可以開啟 Windows 的命令提示字元，使用 ```bash``` 或 ```wsl``` 進入(或者使用 windows terminal)。**Windows 本機的磁碟資料都會存放在 Linux 系統的 mnt 資料夾內。** 使用指令 ```cd /mnt``` 進入資料夾。

### 安裝 windows terminal / zsh / oh-my-zsh 套件
**此步驟非必要**。如果希望終端機可以好看一點，可以考慮安裝，另外可以參考 [youtube 教學](https://www.youtube.com/watch?v=235G6X5EAvM&ab_channel=TheDigitalLife)。如果不安裝可以直接跳至安裝 Hugo 步驟。windows terminal 一樣到 Microsoft Store 安裝 (強烈建議安裝，可以少掉之後設定很多不必要的 bug)。所有套件都安裝完後效果如下：

![](https://i.imgur.com/j2HBJfD.jpg)

#### (一) Windows 安裝 Powerline 字體
對於 Windows 用戶而言，必須使用 Powerline 的字體才能顯示出特定的 icon，可以到 [github Page](https://github.com/powerline/fonts) 或 [nerd font](https://www.nerdfonts.com/)，找到喜歡的字體的 ttf 檔 (筆者是使用 Hack)，下載後在 Windows 上安裝，**不要裝在 WSL 的系統，因為 WSL 是用 windows 終端機的字體**。

![](https://i.imgur.com/TqmL6B2.jpg)

#### (二) 安裝 zsh / oh-my-zsh
接下來的步驟可以參考：[(WSL) 環境設定](https://hackmd.io/@tf-z1zFMTIC8ADhxEcGJEA/BJByCIUHf?type=view) 以及上面提到的 youtube 教學影片。開啟 ubuntu terminal，先用指令 `$ sudo apt update` 更新 apt，然後再下載 zsh / oh-my-zsh :
```
sudo apt install zsh -y
sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```
安裝完後可進入`vim ~/.zshrc`設定喜歡的主題，或者可以跳至下一步安裝 powerlevel10k 主題。

其他參考指令：
```
zsh --version                       # 檢查版本
chsh -s $(which zsh)                # 將 zsh 設為預設 shell
chsh -s /bin/bash                   # 還原成原本預設的 shell
sudo apt-get --purge remove zsh     # 刪除 zsh
```

#### (三) 安裝 powerlevel10k / 修改 LS_COLORS
```
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/themes/powerlevel10k"
```
下載完後重新啟動 terminal 就可以開始設定樣式，如果沒有跳出設定的視窗可以使用這個指令 `p10k configure`。在設定完後看起來大致上完美，其實不然，如果你使用 `ls` 指令會發現產生出來的顏色巨雞巴醜，解決的方法是設定 LS_COLORS，作法：到[這裡](https://github.com/seebi/dircolors-solarized)選一個喜歡的主題下載，例如：
```
curl https://raw.githubusercontent.com/seebi/dircolors-solarized/master/dircolors.ansi-dark --output ~/.dircolors
```
接著在 `~./zshrc` 裡面加上這行 ([參考資料](https://medium.com/gammastack/setup-wsl-on-windows-part-2-f64a9fd546ac))：
```
## set colors for LS_COLORS
eval `dircolors ~/.dircolors`
```

## 安裝 Hugo


## 開始建立部落格