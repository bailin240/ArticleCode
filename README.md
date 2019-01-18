> 最近做一个后台系统用的功能 总结下遇见的问题

功能需求点
- 上传选择文件 判断文件格式
- 保存需要上传的文件，展示文件名
- 提供删除文件功能（未上传前）
- 上传文件

iView 提供的上传组件

```
<Upload
    multiple
    ref="upload"
    :before-upload="handleUpload"
    :show-upload-list="false"
    :on-success="uploadSuccess"
    action="//jsonplaceholder.typicode.com/posts/">
    <Button type="ghost" icon="ios-cloud-upload-outline">浏览</Button>
</Upload>
<div v-for="(item, index) in file">Upload file: {{ item.name }} 
    <a href="javascript:;"  @click="delectFile(item.name)">X</a>
    <Button style="margin-left:30px;"
        size="small"
        v-if="index === 0"
![](https://user-gold-cdn.xitu.io/2018/3/14/16223a8c12c38be9?w=649&h=743&f=png&s=55740)
        type="primary"
        @click="upload"
        :loading="loadingStatus">上传</Button>
</div>
```
用到的API参数 / 方法
- multiple ： 是否支持多选文件 默认 false
- before-upload：上传文件这前的事件钩子，若返回 false 或者 Promise 则停止自动上传
- show-upload-list： 是否显示已上传文件列表 默认为true
- on-success: 上传文件成功后的事件钩子，返回  res(接口方返回的信息), file(上传文件), fileList(上传文件List)
- action： 文件上传地址

#### 上传选择文件 判断文件格式 保存文件
选择文件后会调用方法，在里面做的事情有 判断文件类型是否满足需求，如果满足就保存到需要上传的文件List里，这里我们需要自己定义一个keyID,应为是手动上传需要展示，删除功能，如果没有唯一ID不知道删除那个

这里如果允许多文件上传也不用但心什么，此钩子会把选择的文件当数组一样调用上传件事件前的钩子事件 handleUpload ，所以也不用但心多文件选择只会生成一个KeyID
```
handleUpload (file) { // 上传文件前的事件钩子
    // 选择文件后 这里判断文件类型 保存文件 自定义一个keyid 值 方便后面删除操作
    let keyID = Math.random().toString().substr(2);
    file['keyID'] = keyID;
    // 保存文件到总展示文件数据里
    this.file.push(file);
    // 保存文件到需要上传的文件数组里
    this.uploadFile.push(file)
    // 返回 falsa 停止自动上传 我们需要手动上传
    return false;
}
```
#### 删除功能
```
 delectFile (keyID) { // 删除文件
    // 删除总展示文件里的当前文件
    this.file = this.file.filter(item => {
        return item.name != name
    })
    // 删除需要上传文件数据里的当前文件
    this.uploadFile = this.uploadFile.filter(item => {
        return item.KeyID != keyID
    })
}
```
#### 上传文件
```
 upload () { // 上传文件
    for (let i = 0; i < this.uploadFile.length; i++) {
        let item = this.file[i]
        this.$refs.upload.post(item);
    }
},
```
这里如果是多文件的时候需要循环上传一个一个来，如果一次上传多个组件会报错，只支持一次上传一个文件，希望iView 以后会支持一次上传多个吧，这个手动上传开始一直没找到，不知道手动如何上传调用事件，找功能找半天，在官方文档里也没有写，官方到是有一个例子手动上传的但：

![](https://user-gold-cdn.xitu.io/2018/3/14/16223a94233d482a?w=649&h=743&f=png&s=55740)

并没有事实上传的操作 这里也只是模拟啦 上传方法是在源码里找到的

### 上传成功后
应为我们的上传文件功能和提交整个页面的数据是分开的， 所以提交数据的时候需要验证选择文件是否以上传。在上传成功事件回调里让后台把我们传过去的数据返出来做清空待上传文件数组里的数据（注：主要是拿到文件的keyID 做删除），提交数据时候只需要判断待上传文件数组是否为空就可以了

文件上传回调返回三个参数 
- res 上传结果 成功与失败 上传之后的地址
- file 此次上传的文件
- fileList 文件需要上传的数组数据
```
uploadSuccess (res, file,fileList) { // 文件上传回调 上传成功后删除待上传文件
    console.log(response)
    console.log(file)
    console.log(fileList)  
},
```
这里有个小问题，应为上传的时候是循环调用上传的，如果多个文件上传这里会有多个回调结果不能成功一个文件提示用户一次，所以需要处理一下，这里自定义一个数每次回调回来作自增处理，当值与上待上传文件的length 相等时才提示上传结果

[详细代码地址](https://github.com/bailin240/ArticleCode/tree/master/view/iViewUpload)
