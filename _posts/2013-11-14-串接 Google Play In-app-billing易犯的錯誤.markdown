---
layout:     post
title:      "串接 Google Play In-app-billing易犯的錯誤"
date:       2013-11-14 15:53
author:     "KC"
header-img: "img/post-bg-unix-linux.jpg"
tags:
    - Android
    - IAP
    - In-app products
    - Games
---

<font color="grey">
本文转自[http://lp43.blogspot.tw/2012/04/google-play-in-app-billing.html](http://lp43.blogspot.tw/2012/04/google-play-in-app-billing.html)。写得很详细，有不少参考价值，由于在墙外，转来。
</font>

## 一、前言 
之前提到，臺灣目前雖然無法購買付費型APP，但卻可以使用In-app-billing機制來獲利。 

Google Play的In-app-billing機制很完善，因此在機制底下的規矩也很多。新手在串接時，可能因此發生了一堆奇奇怪怪的錯誤。 這邊我把遇到的問題跟大家分享，這些都是我很寶貴的出錯經驗。 

## 二、文章開始
我在串接Google Play in-app-billing時發生過的問題及錯誤如下︰

### 1.關於應用程式產品內ID值的問題

##### (1)應用程式產品內ID沒有照規則走
雖然Google Play In-app-billing Document已經很明確的教導我們ID值的命名規則是︰

產品IDs是以唯一性的方式跨越應用程式命名空間。產品ID必須啟始字元必須為小寫或數字，而且組成的字串也都只能有小寫(a-z)、數字(0-9)、下 底線(_)和小數點(.)。以"android.test"開頭的產品ID命名被保留，任何以android.test開頭的命名方式皆不可用。附帶一 提，當您在建立產品ID後，是無法修改的，而且您也無法重覆使用同一組產品ID。

有時候為了搶時間，這個ID值沒有照規範走就定義在程式碼中並上傳成草稿APK至Google Play Publisher。直到要在Publisher後臺添加 應用程式產品內ID值 時，才發現自己沒有遵照規範。

因此程式碼內的 應用程式產品內ID值 又要再改一次，再重新上傳一個新的草稿APK。

更糟的是︰
如果你的APP有做package name控管，那麼你的package name一定會超出控管。
因為同樣的package name的APP，即使你從後臺刪除又重新上傳，Google Play都會把它們當成是同一隻APP。這使得你package name需要重新命名才能完成上傳，因而package name已經不是你原本希望的名稱了。 

##### (2)設定Publisher後臺 應用程式產品內ID值 的[受管理]與[不受管理]分類時請小心！

![](https://raw.githubusercontent.com/kimiazhu/kimiazhu.github.io/master/_posts/attachments/2013-11-14/1.png)

由於 應用程式產品內ID值 分成受管理和不受管理類，這個值如果沒有設定好就儲存或發佈，後來發現設定錯了，即使刪除，都不能再在同一個APP內設定同一個產品ID了。 這很麻煩，因為程式還要為了這個不小心的錯誤，重新改程式碼中對應的ID值、重新上傳草稿APK、重新測試… 

### 2.點擊購買流程，iap視窗彈出「這個版本的應用程式還無法用付款功能。」

這算是一個新手錯誤。

![](https://raw.githubusercontent.com/kimiazhu/kimiazhu.github.io/master/_posts/attachments/2013-11-14/2.png)

如果查看一下LOG，收到的LOG應該是 **RESULT_DEVELOPER_ERROR** 官方文件對這個LOG的定義如下︰
此回應指出您的APP試圖發送iap請求，但是APP的AnddroidManifest.xml裡卻沒有宣告 com.android.vending.BILLING權限。也可能是因為應用程式沒有正確的被簽署，或者您發送了一個非正確格式的請求，像是忘了傳 Bundle的key值或者是使用了一個無法被識別的請求類型。

官方文件曾經說，
要測試iap內容，有幾個條件需要實作︰

##### (1)手機的primary account要設成test account

如果不是，請重設為原廠設定，這是官方建議我們的。

##### (2)test account要設定在Google Play Publisher的編輯個人資料的測試帳戶底下。

因為是測試iap，因此我們上傳的草稿apk不用被發佈，只要儲存即可。</br>
但是 應用程式內產品ID 一定要被發佈。</br>
但是這樣要怎麼找的到這些沒有被發佈的應用程式內ID呢？</br>
答案就是要將手機的account設到這邊當成test account。</br>

##### (3)androidManifest.xml裡要宣告

	<uses-permission android:name="com.android.vending.BILLING" />

而且重點是，不僅要宣告這行，還要把這行 **放在<manifest>和<application>中間** 。

##### (4)請確認裝置上的版編、版號、keystore與上傳的草稿APK的版編、版號、keystore一致

因為我們不能用debug.keystore上傳app，因此，在做內部測試時，請確認線上的草稿app的keystore是和裝置上要測試的版本的keystore是一致的。

官方文件Testing In-app Billing單元有提到︰

<code><font color="green">
上傳您的草稿APP至發佈網站。您無需發佈您的APP才能執行端點測試和真實產品ID的消費。</br/>
您僅需以草稿的方式將您的APP上傳。然而， 您必須將您的APP簽署上那把你平時釋出APP時，<br/>
專用的金鑰。而且，您上傳的APP版號必須和你裝置上要執行測試的APP的版號一致。如果想知<br/>
道關於如何上傳APP至Android市集，請見 [Uploading applications](http://market.android.com/support/bin/answer.py?answer=113469)。
</font></code>

我曾經將草稿APK上傳了，但是就是一直回應我**這個版本的應用程式還無法用付款功能**。<br/>
我將這行AndroidManifest.xml裡的宣告改變位置，然後將package name和 應用程式內產品ID 全部重新定義過，也都無法解決，最後才發現我裝置上的APK的版編版號沒有和線上的『已發佈』APK一致。 這些細節如果沒仔細去看，真的會很折磨人。

<font color="grey">
註︰<br/>
2012/11/05 
有時候為了不要讓服務直接上線(還在內測階段)，會上一隻假的APK檔上架Google Play來測試裝置上DEBUG模式的真實APK。如果上傳的假APK裡Android Manifest屬性和手上實測的DEBUG模式真實APK裡的Android Manifest屬性差太多，可能也會造成即使線上和手上裝置版本號一致，但仍show出此版本無法購買的問題。此時建議直接再上傳一個新版號做測試即可。
</font>

### 3.payload很好用，但是使用它是有條件的。

我們知道在request Purchase時，可以附一個payload給Google Play， 屆時如果交易成功，傳回來的Json裡面會傳回這個payload。因此這個payload變得很好用，因為我們可以拿這個值來做虛擬幣加值的依據之類的。

但，這個payload只會在正式金流交易下回傳過來，如果你請求購買的 應用程式內產品ID是**android.test.purchased**之類的，很抱歉，傳回來的Json是不會附帶這個payload的。

### 4.不要將你的購買成功後的程式動作放在Response_OK後面。

	@Override
    public void onRequestPurchaseResponse(RequestPurchase request,
            ResponseCode responseCode) {
        if (Consts.DEBUG) {
            Log.d(TAG, "331 "+request.mProductId + ": " + responseCode);
        }
        //付費成功不是在這裡處理。這裡只是一般購買請求Google Play是否答應的接收處
        if (responseCode == ResponseCode.RESULT_OK) {
            if (Consts.DEBUG) {
                Log.d(TAG, "335 purchase was successfully sent to server");
            }

		//MainActivity.addMoney();//不是在這裡加值

		//logProductActivity(request.mProductId, "sending purchase request");
        } else if (responseCode == ResponseCode.RESULT_USER_CANCELED) {
            if (Consts.DEBUG) {
                Log.d(TAG, "340 user canceled purchase");
            }
		//logProductActivity(request.mProductId, "dismissed purchase dialog");
        } else {
            if (Consts.DEBUG) {
                Log.d(TAG, "346 purchase failed");
            }
			//logProductActivity(request.mProductId, "request purchase returned " + responseCode);
        }
    }

在官方的iap教學文檔中，我們實作了PurchaseObserver。<br/>
這個PurchaseObserver被呼叫的時間點，是在Google Play對iap購買交易有完成的成功回應時，在ResponseHandler.java呼叫purchaseResponse()， 找我們實作的PurchaseObserver底下去執行程式相關的動作。

但是，
ResponseCode.RESULT_OK只是代表我們可以執行iap購買，不代表交易成功了。還記得in-app-billing交易流程圖嗎？

![](https://raw.githubusercontent.com/kimiazhu/kimiazhu.github.io/master/_posts/attachments/2013-11-14/3.png)

**ResponseCode.RESULT_OK**只是這個交易flow的第2條而已。真正表示你交易成功會發生在第7點︰PURCHASE_STATE_CHANGED。

因此交易成功要在onPurchaseStateChange()函式裡實作，程式如下︰

	@Override
    public void onPurchaseStateChange(PurchaseState purchaseState, String itemId,
            int quantity, long purchaseTime, String developerPayload) {
        if (Consts.DEBUG) {
            Log.d(TAG, "286 onPurchaseStateChange() itemId: " + itemId + " " + purchaseState);
        }

		//if (developerPayload == null) {
		//	logProductActivity(itemId, purchaseState.toString());
		//} else {
		//	logProductActivity(itemId, purchaseState + "\n\t" + developerPayload);
		//}
         
        //購買完成會呼叫這裡
        if (purchaseState == PurchaseState.PURCHASED) {
         Log.w(TAG, "297 onPurchaseStateChange: PURCHASED");
		//Toast.makeText(IapPage.this, R.string.purchased_success, Toast.LENGTH_SHORT).show();       

         MainActivity.addMoney(purchaseTime, developerPayload);      
		// mOwnedItems.add(itemId);
	}

###5.老王不能自己買瓜

剛開始串接IAP的新手有時候會遇到「找不到項目」的錯誤，看LOG回覆的錯誤訊息，並且參照Reference通常都能找出錯誤原因，我曾遇過的錯誤是因為**RESULT_ERROR**，查了一下In-app Billing Reference後，發現問題是因為我自己是販售者，我仍然又用販售者的身份去購買in-app Billing的商品，自己跟自己買東西，這件事在Google Wallet是不被允許的。 因此，如果你現在用Test Account購買商品，記得手機登入的Primary Account不能跟販售者的account相同。 

![](https://raw.githubusercontent.com/kimiazhu/kimiazhu.github.io/master/_posts/attachments/2013-11-14/4.png)

###6.別急
許多人在設定和實作的過程中，都急著想看到可以購買的結果。但是，如果你是第1次為這隻APP發佈in-app-billing商品時，Google是需要花時間去處理的(我的實測是2個小時候才找的到該商品，因為中午吃了一個便當，今天吃7-11的義大利麵)。所以，你發佈出去不代表馬上就能找到應用程式內商品，等一下他們吧！

<font color="grey">
註︰<br/>
最近Google好像不提供不發佈APK、僅發佈iab商品測試消費了，
一定要將apk發佈，才能找的到iab商品(見下圖)。2012/08/14
</font>

![](https://raw.githubusercontent.com/kimiazhu/kimiazhu.github.io/master/_posts/attachments/2013-11-14/5.png)

## 三、結論

就是這些了。 這些經驗花了我半年 + 卡住數次得來。它們都是我在串接Google Play iap時常遇到的錯誤。

如果之後還有機會遇到任何的錯誤，我會把經驗分享上來。(當然會希望不要再有了!!) 

只希望之後大家在串接時，不要再犯我犯過的錯了。

## 四、附註

1.官方的Sample Code主程式中unregister observer時的時間點錯了
(範例文件將unregister放在onStop()去執行)，
因為我們不知道系統什麼時候會讓Activity進入onStop()狀態，
造成原本應該監聽iap後續動作的observer被系統終止了。
這會造成交易完成後，我們實作的observer有時候會無法順利被呼叫。 
最好是移到**onDestroy()**再去執行***ResponseHandler.unregister(myPurchaseObserver);***

<font color="grey">張貼者： 潘小鰻 於 9:15 PM</font>