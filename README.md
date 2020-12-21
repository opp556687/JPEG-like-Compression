# Data Compression HW2
## JPEG-like Compression
### 壓縮
* jpeg壓縮是將圖片切成一塊一塊進行壓縮所以一開始要先把原圖切成許多8x8的區塊  
![](https://i.imgur.com/vOGKfvL.png)
* 之後再對切出來的區塊做DCT會把這個8x8區塊的內容轉換成左上到右下低頻到高頻分布之後再對他做量化因為人眼對高頻比較不敏感所以高頻的部分可以以較少的資訊量來儲存  
![](https://i.imgur.com/iC4emDN.png)
* 量化完之後就利用huffman encode去做無損壓縮這邊是以查表的方式來做壓縮
    * 最左上角的是DC因為相鄰區塊的差異不大所以使用DPCM的方式來壓縮
    * 其他63個是AC先做zigzag會得到一連串0和非0的數字因為越右小角是高頻的部分量化完會變成很多0所以先做zigzag可以讓壓縮比較有效率再來做run length code去處理成前面有幾個0跟後面非0的數字是多少ex. 0001->(3,1)之後再來查表
    * 做完huffman之後就完成jpeg的壓縮了
* 如果是彩色圖片要先把RGB轉換成Y Cb Cr來做壓縮Y Cb Cr分別代表的是亮度跟彩度
* 因為人眼對於亮度比較敏感所以彩度可以使用降採樣的方式來減少壓縮的資料
    * 這邊是使用4\:2\:0的方式來採樣
    * 把圖片切成2x2的區塊只取左上角的區塊來進行壓縮  
    * 8x8的區塊就只取紅色的部分來做壓縮這樣資料量就變成原本的1/4  
    ![](https://i.imgur.com/59UFd6m.png)
* 壓縮流程
    * 灰階: 切塊->DCT->量化->huffman encode
    * 彩色: 切塊->採樣->DCT->量化->huffman encode
### 解壓縮
* 解壓縮的過程就是把壓縮的流程反著做一遍所以就把壓縮後的bit stream去查表會還原成run length encode的樣子再把她轉換回去run length encode前的樣子ex.3502->00052
* 之後再按照zigzag的位置把所有的值填回去就會回到沒做zigzag前的樣子再來做反量化跟IDCT就會還原回原本8x8的區塊
* 而彩色的彩度部分因為有做降採樣而且不可還原所以就用採樣的那個值來當作2x2區塊的彩度
* 解壓縮流程
    * 灰階: huffman decode->反量化->IDCT->還原成原本8x8區塊
    * 彩色: huffman decode->反量化->IDCT->升採樣->還原成原本8x8區塊
### 結果
* 左邊原圖右邊壓縮再解壓縮後結果
* 灰階  
![](https://i.imgur.com/9Srybej.png)
* 彩色
![](https://i.imgur.com/gX83x1C.jpg)
* PSNR對比不同quality factor
![](https://i.imgur.com/jt38A3y.png)
* PSNR越高的代表壓縮後與原圖差異越小
* 從結果可以看到quality factor越高的PSNR越高以及灰階圖片的PSNR比彩色高
#### 量化的影響
* 因為在做量化的時候是拿DCT後的結果跟量化的表相除除完之後做四捨五入取整數把小數的部分捨去掉這邊會造成有損壓縮
* 而且quality factor也會影響量化的結果公式如下  
![](https://i.imgur.com/s57fZrf.png)
* Q = S * f(QF) / 100, S表示的是量化表的值
* DCT的結果再去除上Q得到量化後的結果
* 當quality factor越小的時候會使得量化造成的損失越多壓縮後的品質就越差但相對的壓縮後的檔案大小就越小
* QF = 90, 檔案大小 84247 bytes  
![](https://i.imgur.com/U6CdR7N.png)  
![](https://i.imgur.com/RvP0KQb.png)  
* QF = 5, 檔案大小 38154 bytes  
![](https://i.imgur.com/C9AIrZe.png)  
![](https://i.imgur.com/bD4T58v.png)  
#### 彩色的影響
* 因為彩色的圖片有經過採樣把彩度的部分從原本的大小降為1/4而且這個過程不可逆
* 所以在2x2區塊裡面只有左上角的部分保有原本的彩度其他3個的彩度都是由這個的去填的這邊使得壓縮過程造成損失再加上量化時的損失所以在同quality factor的情況下彩色圖片的PSNR比灰階來得低
#### 其他未知問題
* 在同樣的程式同樣的quality factor在windows上執行的結果和mac os/linux上的結果不同
* 左邊windows跑出的結果右邊linux跑出的結果
![](https://i.imgur.com/giw0P4R.jpg)



