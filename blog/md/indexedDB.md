---
title: indexedDB数据库使用总结
date: 2018-07-10 13:15:45
categories: [开发,总结]
tags: [indexedDB]
---
## indexedDB简介
indexedDB是一个前端存储数据库，之前也没有什么了解，这次项目中需要用到，然后就去找了相关资料。数据库有两种，一种是关系型数据库，另一种是非关系型数据库。indexedDB是第二种，它是非关系型数据库，它不需要你去写一些特定的sql语句来对数据库进行操作，数据形式使用的是json。

## 与其他前端存储方式对比
>
也许熟悉前端存储的会说，不是有了LocalStorage和Cookies吗？为什么还要推出indexedDB呢？其实对于在浏览器里存储数据，你可以使用cookies或local storage，但它们都是比较简单的技术，而IndexedDB提供了类似数据库风格的数据存储和使用方式。

>首先说说Cookies，英文直接翻译过来就是小甜点，听起来很好吃，实际上并不是，每次HTTP接受和发送都会传递Cookies数据，它会占用额外的流量。例如，如果你有一个10KB的Cookies数据，发送10次请求，那么，总计就会有100KB的数据在网络上传输。Cookies只能是字符串。浏览器里存储Cookies的空间有限，很多用户禁止浏览器使用Cookies。所以，Cookies只能用来存储小量的非关键的数据。

>其次说说LocalStorage，LocalStorage是用key-value键值模式存储数据，但跟IndexedDB不一样的是，它的数据并不是按对象形式存储。它存储的数据都是字符串形式。如果你想让LocalStorage存储对象，你需要借助JSON.stringify()能将对象变成字符串形式，再用JSON.parse()将字符串还原成对象。但如果要存储大量的复杂的数据，这并不是一种很好的方案。毕竟，localstorage就是专门为小数量数据设计的，所以它的api设计为同步的。而IndexedDB很适合存储大量数据，它的API是异步调用的。IndexedDB使用索引存储数据，各种数据库操作放在事务中执行。IndexedDB甚至还支持简单的数据类型。IndexedDB比localstorage强大得多，但它的API也相对复杂。对于简单的数据，你应该继续使用localstorage，但当你希望存储大量数据时，IndexedDB会明显的更适合，IndexedDB能提供你更为复杂的查询数据的方式。

## indexedDB特性
- 对象仓库
indexedDB中没有表的概念，而是objectStore，一个数据库中可以包含多个objectStore，objectStore是一个灵活的数据结构，可以存放多种类型数据。也就是说一个objectStore相当于一张表，里面存储的每条数据和一个键相关联。我们可以使用每条记录中的某个指定字段作为键值（keyPath），也可以使用自动生成的递增数字作为键值（keyGenerator），也可以不指定。选择键的类型不同，objectStore可以存储的数据结构也有差异。
<table>
<thead>
<tr class="header">
<th>键类型</th>
<th>存储数据</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>不使用</td>
<td>任意值，但是每添加一条数据的时候，需指定键参数</td>
</tr>
<tr class="even">
<td>keyPath</td>
<td>对象，eg: {keyPath: 'id'}</td>
</tr>
<tr class="odd">
<td>keyGenerator</td>
<td>任意值 eg: {autoincrement: true}</td>
</tr>
<tr class="even">
<td>keyPath and KeyGenerator 都使用</td>
<td>对象，如果对象中有keyPath指定的属性则不生成新的键值，如果没有自动生成递增键值，填充keyPath指定的属性</td>
</tr>
</tbody>
</table>

- 事务性
在indexedDB中，每一个对数据库操作是在一个事务的上下文中执行的。事务范围一次影响一个或多个object stores，你通过传入一个object store名字的数组到创建事务范围的函数来定义。例如：db.transaction(storeName, 'readwrite')，创建事务的第二个参数是事务模式。当请求一个事务时,必须决定是按照只读还是读写模式请求访问。

- 基于请求
对indexedDB数据库的每次操作，描述为通过一个请求打开数据库,访问一个object store，再继续。IndexedDB API天生是基于请求的,这也是API异步本性指示。对于你在数据库执行的每次操作,你必须首先为这个操作创建一个请求。当请求完成,你可以响应由请求结果产生的事件和错误。

- 异步
在IndexedDB大部分操作并不是我们常用的调用方法，返回结果的模式，而是请求—响应的模式，所谓异步API是指并不是这条指令执行完毕，我们就可以使用request.result来获取indexedDB对象了，就像使用ajax一样，语句执行完并不代表已经获取到了对象，所以我们一般在其回调函数中处理。

## 使用示例
### 打开数据库
- 判断浏览器是否支持indexedDB数据库
```
var indexedDB = window.indexedDB || window.webkitIndexedDB || window.mozIndexedDB || window.msIndexedDB;
if(!indexedDB)
{
    console.log("你的浏览器不支持IndexedDB");
}
```
- 创建请求打开indexedDB,IndexedDB需要你创建一个请求来打开它。<br>
```
var request = indexedDB.open(name, version);
```
第一个参数是数据库的名称，第二个参数是数据库的版本号。版本号可以在升级数据库时用来调整数据库结构和数据。但你增加数据库版本号时，会触发onupgradeneeded事件，这时可能会出现成功、失败和阻止事件三种情况：
```
request.onerror = function(e) { // 失败
        console.log(e.currentTarget.error.message);
    };

    request.onsuccess = function(e) {   // 成功
        myDB.db = e.target.result;
        console.log('成功打开DB');
    };

    request.onupgradeneeded = function(e) {
        var db = e.target.result;
        if (!db.objectStoreNames.contains('person')) {
            console.log("我需要创建一个新的存储对象");
            //如果表格不存在，创建一个新的表格（keyPath，主键 ； autoIncrement,是否自增），会返回一个对象（objectStore）
            var objectStore = db.createObjectStore('person', {
                keyPath: "id",
                autoIncrement: true
            });

            //指定可以被索引的字段，unique字段是否唯一

            objectStore.createIndex("name", "name", {
                unique: false
            });

            objectStore.createIndex("phone", "phone", {
                unique: false
            });

        }
        console.log('数据库版本更改为： ' + version);
};
```
onupgradeneeded事件在第一次打开页面初始化数据库时会被调用，或在当有版本号变化时。所以，你应该在onupgradeneeded函数里创建你的存储数据。如果没有版本号变化，而且页面之前被打开过，你会获得一个onsuccess事件。

### 添加数据
- 创建一个事务，并要求具有读写权限
```
var transaction = db.transaction(storeName, 'readwrite');
```
- 获取objectStore，调用add方法添加数据
```
var store = transaction.objectStore(storeName); //访问事务中的objectStore
        data.forEach(function (item) {
            store.add(item);//保存数据
        });
```

### 删除数据
- 创建事务，然后调用删除接口，通过key删除对象
```
var transaction = db.transaction(storeName, 'readwrite');

var store = transaction.objectStore(storeName);

store.delete(key);
```

### 查找数据
- 按key查找 开启事务，获取objectStore，调用往get()方法，往方法里传入对象的key值，取出相应的对象
```
var transaction = db.transaction(storeName, 'readwrite');

    var store = transaction.objectStore(storeName);

    var request = store.get(key);

    request.onsuccess = function(e) {

        data = e.target.result;

        console.log(student.name);

};
```
- 使用索引查找
```
var transaction = db.transaction(storeName);

    var store = transaction.objectStore(storeName);

    var index = store.index(search_index);

    index.get(value).onsuccess = function(e) {

        data = e.target.result;

        console.log(student.id);

}
```
- 游标遍历数据
```
var transaction = db.transaction(storeName);

    var store = transaction.objectStore(storeName);

    var request = store.openCursor();//打开游标

    var dataList = new Array();

    var i = 0;

    request.onsuccess = function(e) {

        var cursor = e.target.result;

        if (cursor) {

            console.log(cursor.key);

            dataList[i] = cursor.value;

            console.log(dataList[i].name);

            i++;

            cursor.continue();

        }

        data = dataList;

};
```

### 更新对象
    更新对象，首先要把它取出来，修改，然后再放回去。

```
var transaction = db.transaction(storeName, 'readwrite');

    var store = transaction.objectStore(storeName);

    var request = store.get(key);

    request.onsuccess = function(e) {

        var data = e.target.result;

        for (a in newData) {

            //除了keypath之外

 

            data.a = newData.a;

        }

        store.put(data);

};
```

### 关闭与删除数据库
    关闭数据库可以直接调用数据库对象的close方法
```
function closeDB(db) {

    db.close();

}
```

    删除数据库使用数据库对象的deleteDatabase方法
```
function deleteDB(name) {

    indexedDB.deleteDatabase(name);

}
```

## 参考资料
- [前端存储之indexedDB](https://www.cnblogs.com/dengyulinBlog/p/6141636.html)
- [客户端持久化解决方案：indexedDB](https://www.cnblogs.com/stephenykk/p/6080720.html)