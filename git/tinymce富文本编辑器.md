### 摘要

**tinymce 粘贴图片且自动上传到阿里云，引用上传后的图片链接且删除粘贴的图片base64数据流**



### 初始化编辑器实例时要执行函数

```
init_instance_callback: editor => {
// do something
},
```



### 监听粘贴事件

```
init_instance_callback: editor => {
	editor.on('paste', (evt) => {
		this.onPaste(evt)
	})
}
```



### 获取剪切板内容

    /*粘贴事件*/
    onPaste(event){
      	var _this = this;
      	const items = (event.clipboardData || window.clipboardData).items;
      	if (items[0].type.indexOf('image') !== -1) {    //获取img标签信息
        		const file = items[0].getAsFile()
          	var fileName = file.name;
          	var reader = new FileReader();
          	reader.readAsArrayBuffer(file);   //二进制缓冲区- file对象转arrayBuffer
          	// reader.readAsDataURL(file);   //获取图片的base64数据流
          	var imgBase;     //图片字节流
        		reader.onload = function(e){
          			/*删除粘贴的base64图*/
          			var editor = tinymce.activeEditor;
          			var editorPic = editor.selection.getNode();
          			var pic = editorPic.getElementsByTagName("img");
          			for (var i=0;i<pic.length;i++) {
            				if (pic[i].nodeName == 'IMG') {
              					pic[i].remove();
            				}
          			}
          			imgBase = e.target.result    //获取图片字节流
          			const buffer = new OSS.Buffer(imgBase);
          			_this.uploads(buffer,fileName).then(res => {
            				_this.setImageSize(res.default[0])
          			})
    
        		}
      	}
    },
    /* base64 转 blob */
    // toBlob(urlData,fileType) {
    //     let bytes = window.atob(urlData);
    //     let n = bytes.length;
    //     let u8arr = new Uint8Array(n);
    //     while (n--) {
    //         u8arr[n] = bytes.charCodeAt(n);
    //     }
    //     return new Blob([u8arr], { type: fileType });
    // },
上传阿里云

    async uploads(fileList,fileName) {
      const file = fileList;
      const datas = await api.resource.getAliyunConf()
      const ossClient = this.initOSSClient(datas)
      const fileType = 'image'
      const uploadUrl = `${datas.data.filePathPrefix}${fileType}/${this.creatFileUrl(fileName)}`;
      return new Promise((resolve, reject)=>{
        ossClient.put(uploadUrl, file).then(data=> {
          if (data && data.res && data.res.status === 200) {
              const arr = data.res.requestUrls.map(element => {
                  let cbUrl = ''
                  if (element.indexOf('?uploadId')>-1) {
                      cbUrl = element.split('?')[0]
                  } else {
                      cbUrl = element
                  }
                  return cbUrl
              });
          
              resolve({
                default: arr,
                name: data.name
              })
          } else {
            reject(data)
          }
        }).catch(e => {
            reject(e)
        })
      })
    },
    
    initOSSClient(datas) {
        const ossClient = new OSS({
            region: datas.data.region,
            accessKeyId: datas.data.accessKeyId,
            accessKeySecret: datas.data.accessKeySecret,
            bucket: datas.data.bucket || datas.data.bucketName, /* 装图片的桶名 */
            stsToken: datas.data.securityToken,
            endpoint: datas.data.endpoint, // 新增加
        });
        return ossClient
    },
    creatFileUrl(name) {
        console.log('name',name)
        const date = new Date();
        const year = date.getFullYear();
        const month = date.getMonth() + 1;
        const timestamp = new Date().getTime();
        const fileSuffix = name.lastIndexOf('.');
        const fileExt = name.substring(fileSuffix);// 后缀名
        const storeAs = `${timestamp}${fileExt}`;
        return `${year}/${this.add0(month)}/${storeAs}`
    },
    add0(m){
        return m < 10 ? `0${m}` : m;
    },




