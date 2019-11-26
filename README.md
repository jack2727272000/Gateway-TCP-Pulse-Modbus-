# Gateway-TCP-Pulse-Modbus-
## 1.修改歷史
| 版本  | 修改日期	| 修改內容| 
| ---- |:---------:| ------:|
|V1.0 	|2019/08/25|	

## 2.Watchdog使用說明書-硬體
![alt 文字][logo]

[logo]: https://github.com/jack2727272000/Gateway-TCP-Pulse-Modbus-/blob/master/WatchDog%20%E8%85%B3%E4%BD%8D%E5%AE%9A%E7%BE%A9.png "WatchDog 腳位定義"

系統初使化由S2指撥開關設定工作模式，工作模式更新需要RESET或斷電重開。因REALY有2組故至多可有2個工作模式。

|設備電源	|DC 12-24V，50W|
| ------ |:------------:| 
|繼電器最大承受範圍	|125VAC，60VDC，1A|
|Pulse輸入	|5/24VDC(可調)|

//表格置做參考http://www.tablesgenerator.com/html_tables
<table>
  <tr>
    <th>S2指撥開關編碼</th>
    <th>TCP</th>
    <th>Pulse</th>
    <th>Modbus</th>
  </tr>
  <tr>
    <td>100</td>
    <td>1</td>
    <td></td>
    <td></td>
  </tr>
  <tr>
    <td>010</td>
    <td></td>
    <td>1</td>
    <td></td>
  </tr>
  <tr>
    <td>001</td>
    <td></td>
    <td></td>
    <td>1</td>
  </tr>
  <tr>
    <td>110</td>
    <td>1</td>
    <td>2</td>
    <td></td>
  </tr>
  <tr>
    <td>011</td>
    <td></td>
    <td>1</td>
    <td>2</td>
  </tr>
  <tr>
    <td>101</td>
    <td>1</td>
    <td></td>
    <td>2</td>
  </tr>
  <tr>
    <td>000</td>
    <td colspan="3">修改參數模式</td>
  </tr>
  <tr>
    <td>111</td>
    <td colspan="3">恢復出廠設定</td>
  </tr>
</table>

## 3.Watchdog使用說明書-軟體
### 3.1 TCP模式
```diff
-*注意:此設備需與有DHCP功能的分享器連接。*
```
####  默認設定:
####  1.	初始開機等待3分鐘(可調)。
####  2.	檢查網路狀態，每隔1秒，去PING  4大網站(中華電信、GOOGLE、YAHOO、亞太)，每個time out為 5秒，因此網路斷線最快20秒檢出。連續3次(可調)均無法回復正常，RELAY動作。
####  3.	DO動作時，維持高準位輸出，持續1秒。RELAY連續作動3次(可調)，鎖住DO輸出功能，直至網路恢復後自動重置，回復DO功能。
### 3.2 Pulse模式
####  默認設定:
####  1.	初使開機等待1分鐘(可調)。
####  2.	若為Pulse，check alive，檢查是否有負緣輸入，連續1分鐘(可調)若無則DO作動。
####  3.	DO作動，參考TCP 模式第3點。
### 3.3 Modbus模式
```diff
-注意: (此模式不接收"修改參數模式"的資料)
```
####  默認設定:
####  1.	初使開機等待1分鐘(可調)。
####  2.	Modbus寫入功能，設備ID為0x0F，鮑率9600，寫入寄存器為0x07D0，對此裝置寫入0F 06 07 D0 00 01 49 A9，其中 00 01 為傳輸資料，不可傳00 00，check alive週期是1分鐘(可調)。每分鐘檢查一次狀態是否有變化，若無則DO作動。
####  3.	DO作動，參考TCP 模式第3點。

### 3.4	修改參數模式
```diff
-注意: (此模式不接收"Modbus模式"的資料)
```
####  連接RS485的A,B端，將指撥開關調整至000後重啟電源進行傳輸，相關寄存器位址請參考"Modbus寄存器"。
### 3.5 恢復出廠設定
####  將指撥開關調整至111後重啟電源，等待2秒就恢復出廠設定成功。
------------------------------------------------------------------------------------------------------------
## 4. Modbus寄存器
```diff
-注意:默認的時間單位為分鐘。
-注意:只使用06H寫保持寄存器與10H寫多個保存寄存器。
```
<table>
  <tr>
    <th>名稱</th>
    <th>位址</th>
    <th>說明</th>
  </tr>
  <tr>
    <td>設備ID</td>
    <td><br>&nbsp;&nbsp;0x1060<br>&nbsp;&nbsp;</td>
    <td><br>&nbsp;&nbsp;默認0x0F<br>&nbsp;&nbsp;</td>
  </tr>
  <tr>
    <td><br>  設備Baud Rate<br>  </td>
    <td><br>&nbsp;&nbsp;0x1061<br>&nbsp;&nbsp;</td>
    <td><br>&nbsp;&nbsp;默認0000:9600 bps<br>&nbsp;&nbsp;0000:9600 bps<br>&nbsp;&nbsp;0001:19200 bps<br>&nbsp;&nbsp;0002:38400 bps<br>&nbsp;&nbsp;0003:57600 bps<br>&nbsp;&nbsp;0004:115200 bps<br>&nbsp;&nbsp;</td>
  </tr>
  <tr>
    <td><br>&nbsp;&nbsp;儲存<br>&nbsp;&nbsp;"TCP開機等待時間"<br>&nbsp;&nbsp;</td>
    <td><br>&nbsp;&nbsp;0x1068<br>&nbsp;&nbsp;</td>
    <td><br>&nbsp;&nbsp;默認0x03<br>&nbsp;&nbsp;</td>
  </tr>
  <tr>
    <td><br>&nbsp;&nbsp;儲存<br>&nbsp;&nbsp;"檢查TCP失敗次數"<br>&nbsp;&nbsp;</td>
    <td><br>&nbsp;&nbsp;0x1062<br>&nbsp;&nbsp;</td>
    <td><br>&nbsp;&nbsp;默認0x03<br>&nbsp;&nbsp;</td>
  </tr>
  <tr>
    <td><br>&nbsp;&nbsp;儲存<br>&nbsp;&nbsp;"檢查TCP達到失敗次數所要啟動Relay的次數"<br>&nbsp;&nbsp;</td>
    <td><br>&nbsp;&nbsp;0x1063<br>&nbsp;&nbsp;</td>
    <td><br>&nbsp;&nbsp;默認0x03<br>&nbsp;&nbsp;</td>
  </tr>
  <tr>
    <td><br>&nbsp;&nbsp;儲存<br>&nbsp;&nbsp;"Pulse開機等待時間"<br>&nbsp;&nbsp;</td>
    <td><br>&nbsp;&nbsp;0x1069<br>&nbsp;&nbsp;</td>
    <td><br>&nbsp;&nbsp;默認0x01<br>&nbsp;&nbsp;</td>
  </tr>
  <tr>
    <td><br>&nbsp;&nbsp;儲存<br>&nbsp;&nbsp;"多久時間沒接收到Pulse後啟動Relay"<br>&nbsp;&nbsp;</td>
    <td><br>&nbsp;&nbsp;0x1064<br>&nbsp;&nbsp;</td>
    <td><br>&nbsp;&nbsp;默認0x01<br>&nbsp;&nbsp;</td>
  </tr>
  <tr>
    <td><br>&nbsp;&nbsp;儲存<br>&nbsp;&nbsp;"檢查Pulse達到失敗次數所要啟動Relay的次數"<br>&nbsp;&nbsp;</td>
    <td><br>&nbsp;&nbsp;0x1065<br>&nbsp;&nbsp;</td>
    <td><br>&nbsp;&nbsp;默認0x03<br>&nbsp;&nbsp;</td>
  </tr>
  <tr>
    <td><br>&nbsp;&nbsp;儲存<br>&nbsp;&nbsp;"Modbus開機等待時間"<br>&nbsp;&nbsp;</td>
    <td><br>&nbsp;&nbsp;0x106A<br>&nbsp;&nbsp;</td>
    <td><br>&nbsp;&nbsp;默認0x01<br>&nbsp;&nbsp;</td>
  </tr>
  <tr>
    <td><br>&nbsp;&nbsp;儲存<br>&nbsp;&nbsp;"多久時間沒接收到Modbus後啟動Relay"<br>&nbsp;&nbsp;</td>
    <td><br>&nbsp;&nbsp;0x1066<br>&nbsp;&nbsp;</td>
    <td><br>&nbsp;&nbsp;默認0x01<br>&nbsp;&nbsp;</td>
  </tr>
  <tr>
    <td><br>&nbsp;&nbsp;儲存<br>&nbsp;&nbsp;"檢查Modbus達到失敗次數所要啟動Relay的次數"<br>&nbsp;&nbsp;</td>
    <td><br>&nbsp;&nbsp;0x1067<br>&nbsp;&nbsp;</td>
    <td><br>&nbsp;&nbsp;默認0x03<br>&nbsp;&nbsp;</td>
  </tr>
</table>
