# 第九章
## 變數與可讀性

> 自其變者而觀之，則天地曾不能以一瞬；自其不變者而觀之，則物與我皆無盡也。─蘇軾《前赤璧賦》



## 問題

* 變數愈多愈難記住所有變數
* 變數存活的範圍愈大，就必須記得愈久。
* 變數愈常改變，愈難記得目前的數值



## Solution

* 消除變數
* 縮限變數範圍
* 儘量使用只寫入一次的變數



## 消除變數

消除對可讀性沒有幫助的變數


### 不必要的暫存變數
```python
now = datetime.datetime.now()
root_message.last_view_time = now
```

```python
root_message.last_view_time = datetime.datetime.now()
```


### 消除中間結果

**index_to_remove**

```javascript
var remove_one = function(array, value_to_remove) {
    var index_to_remove = null;
    for (var i = 0; i < array.length; i += 1) {
        if (array[i] === value_to_remove) {
            index_to_remove = i;
            break;
        }
    }
    if (index_to_remove !== null) {
        array.splice(index_to_remove, 1);
    }
};

```

```javascript
var remove_one = function(array, value_to_remove) {
    for (var i = 0; i < array.length; i += 1) {
        if (array[i] === value_to_remove) {
            array.splice(i, 1);
            return;
        }
    }
};
```

**儘快完成工作**是個好習慣。


### 消除控制流程變數

結構化程式設計

```
boolean done = false;
while ( /* condition */ && !done) {
    ...
    if(...) {
        done = true;
        continue;
    }
}
```

```
while ( /* condition */ ) {
    ...
    if(...) {
        break;
    }
}
```

### 小結

你想讓同事覺得一直在面試嗎？
好的面試問題必須至少包括三個變數，也許是因為同時處理三個變數會讓人必須認真思考！



## 縮限變數範圍

* 避免使用全域變數
* 對所有變數縮限範圍
* 儘可能減少可以看到變數的程式碼行數


### 退化變數範圍

```
class LargeClass {
    string str_;
    void Method1() {
        str_ = ...;
        Method2();
    }
    void Method2() {
        // 使用 str_
    }
    // 其他許多不會用到 str_ 的方法
};
```

```
class LargeClass {
    void Method1() {
        string str = ...;
        Method2(str);
    }
    void Method2(string str) {
        // 使用 str
    }
    // 現在其他方法看不到 str
};

```

### 結論
* 儘量使用靜態方法 (static method)
  * 這些程式碼與成員變數無關
* 將大類別分解為較小的類別
  * 分解大檔案成為小的檔案
  * 分解大函數為小的函數


### 變數可視範圍

javascript

```javascript
submitted = false; // 全域變數
var submit_form = function(form_name) {
    if (submitted) {
        return; // 避免重複送出表單
    }
    ...
    submitted = true;
};

```

Closure

```javascript
var submit_form = (function() {
    var submitted = false; // 只能被以下函數使用
    return function(form_name) {
        if (submitted) {
            return; // 免重複送出表單
        }
        ...
        submitted = true;
    };
}());
```
(): 立刻執行外部的匿名函數，傳回內部的函數。


### 全域範圍

```
<script>
var f = function() {
    // 危險，宣告 'i'　時沒有使用 'var'！
    for (i = 0; i < 10; i += 1) ...
};
f();
</script>
```

```
<script>
    alert(i); // Alerts '10'. 'i' 是全域變數！
</script>
```

**總是使用 var 宣告變數**


### 巢狀範圍

Python 與 Javascript 沒有巢狀範圍 (Php 也是)

```java
if (...) {
    int x = 1;
}
x++; // 編譯錯誤！ 'x' 未定義
```

example_value

```python
# 之前都沒有使用 example_value
if request:
    for value in request.values:
        if value > 0:
            example_value = value
            break

for logger in debug.loggers:
    logger.log("Example:", example_value)

```

```
example_value = None
if request:
    for value in request.values:
        if value > 0:
            example_value = value
            break

if example_value:
    for logger in debug.loggers:
        logger.log("Example:", example_value)
```

```
def LogExample(value):
    for logger in debug.loggers:
        logger.log("Example:", value)

if request:
    for value in request.values:
        if value > 0:
            LogExample(value)  # 立刻處理 'value'
            break
```


## 下移宣告

```python
def ViewFilteredReplies(original_id):
    filtered_replies = []
    root_message = Messages.objects.get(original_id)
    all_replies = Messages.objects.select(root_id = original_id)

    root_message.view_count += 1
    root_message.last_view_time = datetime.datetime.now()
    root_message.save()

    for reply in all_replies:
        if reply.spam_votes <= MAX_SPAM_VOTES:
            filtered_replies.append(reply)

    return filtered_replies
```

```python
def ViewFilteredReplies(original_id):
    root_message = Messages.objects.get(original_id)
    root_message.view_count += 1
    root_message.last_view_time = datetime.datetime.now()
    root_message.save()

    all_replies = Messages.objects.select(root_id = original_id)
    filtered_replies = []

    for reply in all_replies:
        if reply.spam_votes <= MAX_SPAM_VOTES:
        filtered_replies.append(reply)

return filtered_replies
```

Think about *all_replies* ?

```
for reply in Messages.objects.select(root_id=original_id):
```



## 偏好單次寫入的變數

* 儘量使用單次寫入變數

```
static const int NUM_THREADS = 10;
```

* 不可變一般來說比較不會造成問題
* 操作變數的地方愈多，愈難記得變數目前的數值



## 範例

```html
<input type="text" id="input1" value="Dustin">
<input type="text" id="input2" value="Trevor">
<input type="text" id="input3" value="">
<input type="text" id="input4" value="Melissa">
```

* var found
* var i
* var elem

```javascript
var setFirstEmptyInput = function(new_value) {
    var found = false;
    var i = 1;
    var elem = document.getElementById('input' + i);
    while (elem !== null) {
        if (elem.value === '') {
            found = true;
            break;
        }
        i++;
        elem = document.getElementById('input' + i);
    }
    if (found) elem.value = new_value;
    return elem;
};
```

* var found

```javascript
var setFirstEmptyInput = function(new_value) {
    var i = 1;
    var elem = document.getElementById('input' + i);
    while (elem !== null) {
        if (elem.value === '') {
            elem.value = new_value;
            return elem;
        }
        i++;
        elem = document.getElementById('input' + i);
    }
    return null;
};

```

* var i
* var elem

```javascript
var setFirstEmptyInput = function(new_value) {
    for (var i = 1; true; i++) {
        var elem = document.getElementById('input' + i);
        if (elem === null)
            return null; // Search Failed. No empty input found.
        if (elem.value === '') {
            elem.value = new_value;
            return elem;
        }
    }
};

```



## 結語

* 消除變數 (儲存**中間結果**的變數)
* 縮限變數範圍 (將變數移到最少程式碼的可視範圍之內)
* 儘量使用只寫入一次的變數 (const, final)