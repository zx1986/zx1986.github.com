---
layout: post
title: "Debugging Javascript"
date: 2012-08-04 15:19
comments: true
categories: Javascript
---

讀了一個很棒的 JavaScript/jQuery Debug [投影片](http://fixingthesejquery.com)，
做個筆記。

首先，要認識各個瀏覽器的 Debugger。
Firefox，Chrome，甚至 IE，都有它們自己的 Debugger。
這些 Debugger 長得都很像，熟悉了其中一種，其他都大同小异。
各個 Debugger 預設的啟動快捷鍵大都是`F12`。

Firefox 除了非常知名的 Debug Plugin：[Firebug](https://getfirebug.com)。
還內建了個更炫的玩意：3D View。
在網頁中`點擊右鍵`，選擇`檢視元素`，右下角有一個`3D View`按鈕。
它可以讓你在三維環境中，檢視每個網頁元素（DOM Element）。
在分析多層 CSS Layer（z-index）的網頁時，非常實用！

使用 jQuery 出現`jQuery is not defined`或`$ is not defined`時，
請先檢查 jQuery 的引入路徑是否正確，
`$`別名符號是否已經被其他的 Javascrip Library 用走了？
可以用`jQuery.noConflict()`函式檢查一下。

出現`jQuery.fn.somePlugin is not defined`時，
先檢查 Plugin Library 是否有在載入 jQuery Library 之後載入。

善用 Debugger 中的 breakpoint（斷點）設定。
在想要設定斷點的 Javascript 語句後，
加入一行`debugger;`即可輕鬆設定斷點。

Debugger 中找到一個叫`watch expression`的地方，
直接輸入想要檢查的變數，它會把這個變數的身家通通翻出來。
例如輸入：`this`

使用`window.alert('錯誤訊息或變數')`不是個好主意。
改用`console.log('錯誤訊息或變數')`搭配 Debugger 的 Console 視窗是好主意！
例如`console.log($(""your selector").length)`可以檢查元素是否存在。
Console 視窗還有一大堆 [API function](http://getfirebug.com/wiki/index.php/Console_API) 。

搭配 jQuery 的 Javascript 建議寫法：

    (function($)){
        $(document).ready(function(){

                // your code ...

        });
    }(jQuery); // take $ as jQuery

以確定整個 DOM 文件載入完成後，才執行你寫的 Javascript 腳本。

> When you call $('a'), it returns all the links on the page at the time it was called, and .click(fn) adds your handler to only those elements. When new links are added, they are not affected.

    $(document).bind("click", function(e)
    {
      if ($(e.target).is("a")
      {
        // this === document
      }
    });
     
    $(document).delegate("a", "click", function(e)
    {
        // this === clicked anchor
    });
     
    $("a").live("click", function(e)
    {
        // this === clicked anchor
    });

注意，呼叫 this 的 jQuery 物件時，是用`$(this)`，不是`$('this')`。
