---
title: Face++与Java简单应用（新）
date: 2018-5-24 15:42:34
categories: [开发,Web]
tags: [Java,Face++,BootStrapFileInput]
---
## 开篇
- 前段时间逛了下[旷视官网](https://www.megvii.com/#jzl_kwd=41736852109&jzl_ctv=16511707555&jzl_mtt=1&jzl_adt=cl1)，真是变化好大页面好漂亮现在，依稀记得最开始上的官网哪来这么炫酷
- 然后就继续点进去看了下[Face++API](https://console.faceplusplus.com.cn/documents/4887579)，果不其然参数变了不少，识别点越来越多了
- 再接下来我又翻了翻自己很久前写的一篇[博文](https://blog.csdn.net/ffj0721/article/details/54322133)，这不行了，这之前写的啥玩意，代码也找不到了，而且现在也不能用了，故，立了个flag

## 操刀
开始操刀了，还是一样我选择了SpringBoot，没什么，就是方便。这次选择用BootStrap框架。

- 首先操刀之前，先去上面官网去注册用户，创建个应用，目的是为了获取它的api-key和api-secret![](https://raw.githubusercontent.com/Folgerjun/materials/master/blog/face/face-key.png)
- 现在不像之前那样复杂了，那时候还得下个jar包导入才能调它的方法，现在直接在线调用即可。具体代码如图位置：![](https://raw.githubusercontent.com/Folgerjun/materials/master/blog/face/face-java.png)这样的话只需要把你需要识别的图片路径，你刚获取的api-key和api-secret替换就行了，然后你就可以得到一串json格式的字符串。太长我就不放了，主要信息就在faces中。我这里主要就是简单获取faces下面attributes中的一些参数。具体可直接看它官网的API，很详细的。
- 图片上传我使用的是BootStrap-FileInput插件，这个插件确实很美观，刚开始使用的时候不太会，故研究了会。特别需要注意需要导入此插件的js和css，不然你就会有种想砸键盘的感觉了。

html：

```
<div class="modal-body" style="text-align: center;">  
    <a href="" class="form-control" style="border:none;">上传照骗</a>  
    <input type="file" name="picFile" id="picFile" class="file-loading" />  
</div>
```
js:
```
//参数1:控件id、参数2:上传地址  
init("picFile", "/sendPic");

//初始化fileinput控件（第一次初始化）  
function init(ctrlName, uploadUrl) {  
	var control = $('#' + ctrlName);  
		 //初始化上传控件的样式  
		 control.fileinput({  
		 language: 'zh',                                         //设置语言  
		 uploadUrl: uploadUrl,                                   //上传的地址  
		 allowedFileExtensions: ['jpg', 'png', 'jpeg'],          //接收的文件后缀  
		 showUpload: true,                                       //是否显示上传按钮  
		 showCaption: false,                                     //是否显示标题  
		 browseClass: "btn btn-primary",                         //按钮样式       
		 dropZoneEnabled: false,                               //是否显示拖拽区域  
		 //minImageWidth: 50,                                    //图片的最小宽度  
		 //minImageHeight: 50,                                   //图片的最小高度  
		 //maxImageWidth: 1000,                                  //图片的最大宽度  
		 //maxImageHeight: 1000,                                 //图片的最大高度  
		 maxFileSize: 2048,                                       //单位为kb，如果为0表示不限制文件大小  
		 //minFileCount: 0,  
		 //maxFileCount: 10,                                       //表示允许同时上传的最大文件个数  
		 //enctype: 'multipart/form-data',  
		 validateInitialCount:true,  
		 previewFileIcon: "<i class='glyphicon glyphicon-king'></i>",  
		 //msgFilesTooMany: "选择上传的文件数量({n}) 超过允许的最大数值{m}！",  
		 uploadExtraData:function (previewId, index) {           //传参  
		    var data = {  
		             //此处自定义传参  
		       		};  
		         return data;  
		        }  
		     });  
		  
		  //导入文件上传完成之后的事件  
		  $("#picFile").on("fileuploaded", function (event, data, previewId, index) {  
		            
		        });  
		    }

```
文件上传成功后后台还需要接收它，如何接收?且听我慢慢道来：
```
MultipartHttpServletRequest multipartRequest = (MultipartHttpServletRequest) request;
List<MultipartFile> fileList = multipartRequest.getFiles(dstFileName);
```
这两句可以接收到对应页面上传的图片，List说明它可以一次上传多张。既然能接收到，那就什么都不是事了。接下来想怎么玩就怎么玩。

- 在页面用到BootStrap模态框的时候我发现它不能拖拽，然后网上冲浪了一会，发现把jquery-ui导入就可以使用draggable()方法。可全局可单页面，我这里是全局设置。
```
// 使模态框可拖拽		    
$(document).on("show.bs.modal", ".modal", function(){
    $(this).draggable();
    $(this).css("overflow-y", "scroll");   
    // 防止出现滚动条，出现的话，你会把滚动条一起拖着走的
});
```

- 前端知识还需要恶补啊，现在页面想做点炫酷的效果是一点办法没有，布局排版还丑的一批。还需努力啊！

## Screenshots

- Index ![](https://raw.githubusercontent.com/Folgerjun/materials/master/blog/face/index.png)

- One Person Result![](https://raw.githubusercontent.com/Folgerjun/materials/master/blog/face/individual.png)

- More than One Person Result![](https://raw.githubusercontent.com/Folgerjun/materials/master/blog/face/multiplayer.png)

## Links
- [演示网址](http://47.96.88.132:8091/)
- [GitHub地址](https://github.com/Folgerjun/face.putop.top)
- [BootStrap-FileInput插件详情参考](http://www.jq22.com/jquery-info5231)
- [之前写的一篇博文](https://blog.csdn.net/ffj0721/article/details/54322133)
- [Face++API](https://console.faceplusplus.com.cn/documents/4887579)