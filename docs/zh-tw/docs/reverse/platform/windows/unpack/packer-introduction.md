# 保護殼簡介

## 認識殼是什麼

**殼** 是在一些計算機軟件裏也有一段專門負責保護軟件不被非法修改或反編譯的程序。

它們一般都是先於程序運行，拿到控制權，然後完成它們保護軟件的任務。

![](./figure/what_is_pack.png)

由於這段程序和自然界的殼在功能上有很多相同的地方，基於命名的規則，就把這樣的程序稱爲 **殼** 了。

## 殼的分類

我們通常將 **殼** 分爲兩類，一類是壓縮殼，另一類是加密殼。

### 壓縮殼

壓縮殼早在 DOS 時代就已經出現了，但是當時因爲計算能力有限，解壓開銷過大，並沒有得到廣泛的運用。

使用壓縮殼可以幫助縮減 PE 文件的大小，隱藏了 PE 文件內部代碼和資源，便於網絡傳輸和保存。

通常壓縮殼有兩類用途，一種只是單純用於壓縮普通 PE 文件的壓縮殼，而另一種則會對源文件進行較大變形，嚴重破壞 PE 文件頭，經常用於壓縮惡意程序。

常見的壓縮殼有：Upx、ASpack、PECompat

### 加密殼

加密殼或稱保護殼，應用有多種防止代碼逆向分析的技術，它最主要的功能是保護 PE 免受代碼逆向分析。

由於加密殼的主要目的不再是壓縮文件資源，所以加密殼保護的 PE 程序通常比原文件大得多。

目前加密殼大量用於對安全性要求高，對破解敏感的應用程序，同時也有惡意程序用於避免（降低）殺毒軟件的檢測查殺。

常見的加密殼有：ASProtector、Armadillo、EXECryptor、Themida、VMProtect

## 殼的加載過程

### 保存入口參數

1.  加殼程序初始化時保存各寄存器的值
2.  外殼執行完畢，恢復各寄存器值
3.  最後再跳到原程序執行

通常用 `pushad` / `popad`、`pushfd` / `popfd` 指令對來保存和恢復現場環境

### 獲取所需函數 API

1.  一般殼的輸入表中只有 `GetProcAddress`、`GetModuleHandle` 和 `LoadLibrary` 這幾個 API 函數
2.  如果需要其他 API 函數，則通過 `LoadLibraryA(W)` 或 `LoadLibraryExA(W)` 將 DLL 文件映射到調用進程的地址空間中
3.  如果 DLL 文件已被映射到調用進程的地址空間裏，就可以調用 `GetModuleHandleA(W)` 函數獲得 DLL 模塊句柄
4.  一旦 DLL 模塊被加載，就可以調用 `GetProcAddress` 函數獲取輸入函數的地址

### 解密各區塊數據

1.  處於保護源程序代碼和數據的目的，一般會加密源程序文件的各個區塊。在程序執行時外殼將這些區塊數據解密，以讓程序正常運行
2.  外殼一般按區塊加密，按區塊解密，並將解密的數據放回在合適的內存位置

### 跳轉回原程序入口點

1.  在跳轉回入口點之前，一般會恢復填寫原PE文件輸入表（IAT），並處理好重定位項（主要是 DLL 文件）
2.  因爲加殼時外殼自己構造了一個輸入表，因此在這裏需要重新對每一個 DLL 引入的所有函數重新獲取地址，並填寫到 IAT 表中
3.  做完上述工作後，會將控制權移交原程序，並繼續執行