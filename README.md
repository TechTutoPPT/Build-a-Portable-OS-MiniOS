# Build-a-Portable-OS-MiniOS

[![](https://github.com/TechTutoPPT/Build-a-Portable-OS-MiniOS/blob/main/cover.PNG)](https://youtu.be/Pl0ANCG3Euo)

本人使用便攜系統是有兩個極端的原因, 一是希望借用一下電腦硬體作一些測試或做一些簡單的工作, 如測試一下某Docker應用, 管理一下文件等, 都是想不打亂內置系統為前題.
而另一個原因就內置系統被損毀用於救機, 例如重新設置啟動引導, 分割磁碟, 格式化分區, 重裝系統等.
為此我選用便攜系統的要求就是能做到上述功能外, 還要容量細小, 啟動進入系統時間快, 能有較好的硬件驅動支援能力, 能上網查找及下戴資料.
而現在介紹的MiniOS便一一都能實現! 除此之外它在啟動選項中更為我帶來以下驚喜:
1.Resume Previous Session這選項如果是首次選用的話是新創一個增量保存的工作區(即所有設定或檔案更改會被存保下來), 如果已創建過工作區便是繼續啟用上次那個工作區.
2.Start a New Session創建一個新的工作區, 如果不是首個工作區, 即表示不使用對上一次那個工作區重新另建一個全新的工作區(這裡我發現了一個有趣的做法, 稍後再說)
3.Choose Session During Startup假如你已創建多個工作區, 可以用這選項去選用啟動那個工作區.
4. Fresh Start全新最初始的狀態下啟動系統, 操作後不會保留任何設定及檔案.
5. Copy to RAM與第4項作用相同, 只是啟動時將整個系統抄寫到記憶體中, 過程較慢, 操作效能交快, 亦要注意有足夠的記憶體.

以下講解一下它的創建過程:
先到官網下載它的ISO檔案(我選用它的Standard版本): https://minios.dev/
再到Rufus下載這個影像寫入工具: https://rufus.ie/en/
再準備一支高速的USB隨身碟
材料齊備後, 便啟動Rufus>揀選隨身碟作為寫入裝置>揀選MiniOS的ISO映像檔>檔案系統選FAT32(還能保留USB隨身碟分享檔案的基本功用)>點選執行, 以ISO映模式寫入>大功告成!

現在重啟電腦於BIOS選用USB隨身碟啟動, 再選Resume Previous Session便能進入MiniOS系統, 而我會再執行以下個人化配置:
安裝中文字體:
```
sudo apt update
sudo apt install fonts-wqy-zenhei
```

設定以中文顯示(過程比較繁複, 有需要才操作吧):
從GRUB原始碼倉庫下載繁體中文語言檔zh_TW.po:
```
wget https://github.com/ParrotSec/grub2/blob/master/po/zh_TW.po
```
安裝編譯工具:
```
sudo apt install msgfmt nano
```

將zh_TW.po編譯成zh_TW.mo:
```
msgfmt zh_TW.po -o zh_TW.mo
```
將編譯好的zh_TW.mo複製到 MiniOS的locale目錄中:
```
sudo cp zh_TW.mo /minios/boot/grub/locale/
```

修改grub.cfg配置:
```
nano /minios/boot/grub/grub.cfg
```
於en_US=English前方加上zh_TW=Chinese 

安裝語言包與設定locales:
```
sudo apt install locales
```

執行locale設定指令:
```
sudo dpkg-reconfigure locales
```
在選單中選擇zh_TW.UTF-8(繁體中文 UTF-8編碼)並設為預設語言

編輯.bashrc:
```
nano ~/.bashrc
```
於檔案內容尾段加入:
```
export LANG=zh_TW.UTF-8
export LC_ALL=zh_TW.UTF-8
```

然後執行以下指令將之生效:
```
source ~/.bashrc
```

再於系統管理工具MiniOS Configurator套用以下設定:
Locales: zh_TW.UTF-8
Timezone: Asia/Hong_Kong
然後重啟MiniOS便能以中文顯示

安裝中文輸入法:
```
sudo apt install fcitx5 fcitx5-chinese-addons fcitx5-table-cangjie5 fcitx5-frontend-gtk3 fcitx5-frontend-qt5 fcitx5-config-qt
```

安裝輸入法配置具:
```
sudo apt install im-config zenity jq
```
執行配置:
```
im-config
```
提示是否選擇輸入法設定, 選Yes
選擇系統預設使用的輸入法, 選fcitx5並點擊OK
重啟MiniOS後於工具列的Fcitx Confuguration中加入Cangjie5, 便能使用Ctrl+Space方式切換成倉頡輸入法

要讓fcitx5每次進入桌面自動執行, 可執行以下指令，將fcitx5的桌面啟動檔案複製到用戶自動啟動資料夾中:
```
mkdir -p ~/.config/autostart
cp /usr/share/applications/org.fcitx.Fcitx5.desktop ~/.config/autostart/
```

修改GRUB啟動選單的預設顯示時長:
```
nano /minios/boot/grub/grub.cfg
```
修改成set timeout=0
```
nano /minios/boot/grub/main.cfg
```
修改成set timeout=3

安裝Docker:
```
sudo apt update
sudo apt install ca-certificates curl gnupg lsb-release -y
```

添加官方GPG key:
```
sudo mkdir -m 0755 -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```
配置Docker軟件源:
```
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian \
$(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

安裝Docker Engine:
```
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
```

最後上方我不是說發現了一個有趣的做法, 那就是原先Start a New Session是創建一個全新初始狀態的工作區, 但透過以下方法, 可替換成一個被修改過的工作區,
這樣做法有什麼好處? 就是能保存起一個滿意的工作區作為備份, 然後快速搭建起另一個滿意的工作區作為變更, 你都不想再多做一次上述種種的個人配置及工具安裝過程吧.
做法其實很簡單:
於桌面按Alt+F3開啟應用程式搜尋器, 尋找並啟動MiniOS Session Manager, 再按Create完成創建另一新工作區.
然後以檔案總管進入本工作區的資料夾中:/minios/changes/1將所有檔案複製到/minios/changes/2(新工作區的資料夾)並取代相同的檔案, 
最後重啟電腦, 於啟動選單中選Choose Session During Startup揀選新的工作區, 這樣你便簡單地複製了一個滿意的工作區.
