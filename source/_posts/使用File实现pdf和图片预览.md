---
title: 使用File实现pdf和图片预览
tags: File
categories: 前端
toc: true
date: 2017-7-22 10:08:03
---

HTML5为我们带来了File相关的API，现在我们可以在Web页面中访问本地文件并且将其内容展示在页面中。

比如我们时常有这样的需求，让用户点击某个按钮上传图片，在图片上传成功之后，显示上传图片的缩略图；或者用户上传pdf文件之后，可以在线浏览pdf的内容。

这里就使用了[File](https://developer.mozilla.org/en-US/docs/Web/API/File)、[FileReader](https://developer.mozilla.org/en-US/docs/Web/API/FileReader)、[URL](https://developer.mozilla.org/en-US/docs/Web/API/URL/createObjectURL)这几个web api来实现了图片和pdf文件的在线预览功能，其实现的效果如下：

![](http://7xvlvo.com1.z0.glb.clouddn.com/GIF.gif)

html:

{% codeblock lang:html %}
<div class="selectDemo">
    <div class="selectBox">
        <label class="selectBtn">
            请选择图片或者pdf
            <input type="file" multiple accept="image/*, .pdf" class="selectIpt"/>
        </label>
    </div>
    <div class="previewBox">
        <ul class="previewList"></ul>
    </div>
</div>
{% endcodeblock %}

由于input标签太丑了，所以我们将它通过`display:none`来隐藏，仅留下label。当然，我们也可以使用另外的一个标签比如<a>,在点击<a>标签时触发input的click事件。

javascript:

{% codeblock lang:javascript %}
const $ = (el) => document.querySelector(el);
$('.selectIpt').addEventListener('change', handleFiles);
function handleFiles () {
    const files = this.files;
    for (let i = 0, len = files.length; i < len; i ++) {
        showFilePreview(files[i]);
    }
}
function showFilePreview (file) {
    const fileType = file.type;
    if (!fileType) {
        return;
    }
    if (/^image/.test(fileType)) {
        return handleImageFile(file);
    }
    if (fileType === 'application/pdf') {
        return handlePdfFile(file);
    }
}
function handleImageFile (file) {
    // 使用FileReader来处理
    const reader = new FileReader ();
    reader.addEventListener ('load', (e) => {
        const imgUrl = e.target.result;
        let imgNode = document.createElement('img');
        imgNode.src = imgUrl;
        imgNode.setAttribute('height', '200px');
        createPreviewItem(imgNode);
    });
    reader.readAsDataURL(file);
}
function handlePdfFile (file) {
    // 使用URL来处理
    const URL = window.URL || window.webkitURL;
    const pdfUrl = URL.createObjectURL(file);
    let embedNode = document.createElement('embed');
    embedNode.src = pdfUrl;
     embedNode.setAttribute('height', '500px');
    embedNode.setAttribute('width', '100%');
    console.log(embedNode);
    createPreviewItem(embedNode);
}
function createPreviewItem (newNode) {
    let newLi = document.createElement('li');
    newLi.appendChild(newNode);
    $('.previewList').appendChild(newLi);
}
{% endcodeblock %}

下面总结下代码中使用到的几个API。

## [File](https://developer.mozilla.org/en-US/docs/Web/API/File)

File为我们提供了Web访问文件信息的接口。File对象通常来自与FileList对象，而FileList对象可以通过以下两种常用方式得到：

- <input>标签的files属性
- 用户拖放文件到web中产生的[DataTransfer](https://developer.mozilla.org/en-US/docs/Web/API/DataTransfer)对象

另外，每个File对象都是[Blob](https://developer.mozilla.org/en-US/docs/Web/API/Blob)对象的特例，因此File继承了Blob的属性，它俩都可以当作参数传入`FileReader`、`URL.createObjectURL()`、`XMLHttpRequest.send()`中。

File对象，有3个我们常用到的属性:`name`、`size`、`type`。

## [FileReader](https://developer.mozilla.org/en-US/docs/Web/API/FileReader)

FileReader允许web应用来`异步读取`文件的内容，这些文件既可以是File对象，也可以是Blob对象。

FileReader对象最常用的一个对象是`result`，它包含了我们读取到的文件内容。我们可以在其load事件中，通过`e.target.result`来获取其值 。

另外，它还有一些方法，比如`abort()`、`readAsDataURL`、`readAsText()`。这些方法可以将文件内容以不同形式返回给我们。

## [URL](https://developer.mozilla.org/en-US/docs/Web/API/URL/createObjectURL)

URL主要用来解析、构建、编码、解码url。这里主要学习它的一个静态方法，即`URL.createObjectURL()`。

我们可以将一个File对象或者Blob对象传入URL.createObjectURL()中，它将会返回一个代表该对象的url，我们便可以将这个url赋值给一些标签的src属性，以达到显示文件内容的目的。

这个新生成的url的生命周期是与创建它的document绑定在一起的，也就是说，当页面卸载的时候，这个url就会被回收。当然，我们也可以主动回收，比如在<img>的load事件中，调用`URL.revokeObjectURL()`方法。

## 参考资料

- [Using files from web applications](https://developer.mozilla.org/en-US/docs/Using_files_from_web_applications)







