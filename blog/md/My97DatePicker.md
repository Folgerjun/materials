---
title: My97DatePicker时间日期插件使用示例
date: 2018-6-25 18:17:59
categories: [开发,Web]
tags: [js]
---
## 前言

- My97DatePicker目录是一个整体,不可破坏里面的目录结构,也不可对里面的文件改名,可以改目录名
- My97DatePicker.htm是必须文件,不可删除

### 各目录及文件的用途

- WdatePicker.js配置文件,在调用的地方仅需使用该文件,可多个共存,以xx_WdatePicker.js方式命名
- config.js 语言和皮肤配置文件,无需引入
- calendar.js 日期库主文件,无需引入
- My97DatePicker.htm 临时页面文件,不可删除
- 目录lang 存放语言文件,你可以根据需要清理或添加语言文件
- 目录skin 存放皮肤的相关文件,你可以根据需要清理或添加皮肤文件包
- 当WdatePicker.js里的属性:$wdate=true时,在input里加上class="Wdate"就会在选择框右边出现日期图标,如果您不喜欢这个样式,可以把class="Wdate"去掉,另外也可以通过修改skin目录下的WdatePicker.css文件来修改样式

## 示例

### 没有对控件进行设置
`<input class="Wdate" type="text" onfocus="WdatePicker()"/>`

### 限制日期的范围是 2006-09-10到2008-12-20
```
<input id="d411" class="Wdate" type="text" onfocus="WdatePicker({skin:'whyGreen',minDate: '2006-09-10', maxDate: '2008-12-20' })"/>
```

### 限制日期的范围是 2008-3-8 11:30:00 到 2008-3-10 20:59:30
```
<input type="text" class="Wdate" id="d412"
onfocus="WdatePicker({skin:'whyGreen',dateFmt: 'yyyy-MM-dd HH:mm:ss',
minDate: '2008-03-08 11:30:00', maxDate: '2008-03-10 20:59:30' })" value="2008-03-09 11:00:00"/>
```

### 限制日期的范围是 2008年2月 到 2008年10月
```
<input type="text" class="Wdate" id="d413" onfocus="WdatePicker({dateFmt: 'yyyy年M月', minDate: '2008-2', maxDate: '2008-10' })"/>
```

### 限制日期的范围是 8:00:00 到 11:30:00
```
<input type="text" class="Wdate" id="d414" onfocus="WdatePicker({dateFmt: 'H:mm:ss', minDate: '8:00:00', maxDate: '11:30:00' })"/>
```

### 只能选择今天以前的日期(包括今天)
```
<input id="d421" class="Wdate" type="text" onfocus="WdatePicker({skin:'whyGreen',maxDate: '%y-%M-%d' })"/>
```

### 使用了运算表达式 只能选择今天以后的日期(不包括今天)
```
<input id="d422" class="Wdate" type="text" onfocus="WdatePicker({minDate: '%y-%M-#{%d+1}' })"/>
```

### 只能选择本月的日期1号至本月最后一天
```
<input id="d423" class="Wdate" type="text" onfocus="WdatePicker({minDate: '%y-%M-01', maxDate: '%y-%M-%ld' })"/>
```

### 只能选择今天7:00:00至明天21:00:00的日期
```
<input id="d424" class="Wdate" type="text" onfocus="WdatePicker({dateFmt:'yyyy-M-d H:mm:ss',minDate: '%y-%M-%d 7:00:00', maxDate: '%y-%M-#{%d+1} 21:00:00' })"/>
```

### 使用了运算表达式 只能选择 20小时前 至 30小时后 的日期
```
<input id="d425" class="Wdate" type="text"
onClick="WdatePicker({dateFmt:'yyyy-MM-dd HH:mm',minDate: '%y-%M-%d #{%H-20}:%m:%s' ,maxDate: '%y-%M-%d #{%H+30}:%m:%s' })"/>
```
### 前面的日期不能大于后面的日期且两个日期都不能大于 2020-10-01
> 合同有效期从 到 <br>
[注意: 两个日期的日期格式必须相同.
dp. 相当于 document.getElementByIdx_x 函数.
那么为什么里面的 ’ 使用 \’ 呢? 那是因为 ” 和 ’ 都被外围的函数使用了,故使用转义符 \ ,否则会提示JS语法错误.所以您在其他地方使用时注意把 \’ 改成 ” 或者 ’ 来使用。
`#F{$dp.$D(\'d4312\')||\'2020-10-01\'} 表示当 d4312 为空时, 采用 2020-10-01 的值作为最大值`]

```
<input id="d4311" class="Wdate" type="text" onFocus="WdatePicker({maxDate: '#F{$dp.$D(\'d4312\')||\'2020-10-01\'}' })"/>
<input id="d4312" class="Wdate" type="text" onFocus="WdatePicker({minDate: '#F{$dp.$D(\'d4311\')}' ,maxDate:'2020-10-01' })"/>
```

## 取值和赋值
### html:
```
<input class="p-Wdate" type="text"   onfocus="WdatePicker()"/>
<p><button class="tijiaoBtn">提交</button></p>
```
### js:
```
//赋值
$(".p-Wdate").val("2019-01-01");
//取值
$(".tijiaoBtn").on("click",function(){
  console.log($(".p-Wdate").val());
});
```

## 参考
- [WdatePicker.js时间日期插件](https://www.2cto.com/kf/201707/662183.html)
- [My97DatePicker日历控件配置](https://www.cnblogs.com/weiqt/articles/2012169.html)
- [My97官网](http://www.my97.net/)