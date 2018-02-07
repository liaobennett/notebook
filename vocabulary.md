# 名詞

### request  

請求

### response 

回應

### prototype 

這個字沒有一定的中文翻譯，如果硬是要說，也許會說「原型」，也聽過有人說「雛型」。

因為一般都會直接說英文 Prototype 這個字，所以一定要會念，英文念法點擊這裡。

https://developer.mozilla.org/zh-TW/docs/Web/JavaScript/Inheritance_and_the_prototype_chain

### util.inspect

util.inspect(object,[showHidden],[depth],[colors])

是一個將任意物件轉換 為字串的方法，通常用於調試和錯誤輸出。它至少接受一個參數 object，即要轉換的物件。

showHidden 是一個可選參數，如果值為 true，將會輸出更多隱藏資訊。

depth 表示最大遞迴的層數，如果物件很複雜，你可以指定層數以控制輸出資訊的多 少。如果不指定depth，默認會遞迴2層，指定為 null 表示將不限遞迴層數完整遍歷物件。 如果color 值為 true，輸出格式將會以ANSI 顏色編碼，通常用於在終端顯示更漂亮 的效果。

特別要指出的是，util.inspect 並不會簡單地直接把物件轉換為字串，即使該對 象定義了toString 方法也不會調用。
