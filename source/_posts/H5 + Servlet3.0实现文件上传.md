---
title: H5 + Servlet3.0实现文件上传
date: 2018-03-27 13:55:00
tags:
- j2ee
categories:
- 技术
---
在之前的项目中要实现文件上传功能时，我一般使用Apache的commons-fileupload组件，最近发现Servlet3.0提供了对文件上传的原生支持，API非常简洁易懂，总结一下使用方法。

 - 后端
 
1. 使用注解@MultipartConfig将一个Servlet标识为支持文件上传，否则会报Unable to process parts as no multi-part configuration has been provided错误，这里使用了Servlet3.0的注解特性。
2. HttpServletRequest提供了getPart和getParts两个方法获取到表单的内容。

```
		//getPart方法
		Part image = null;
        InputStream imageInputStream = null;
        try {
            image = request.getPart("image");
            if (image != null && image.getSize() != 0)
                imageInputStream = image.getInputStream();
	        FileUtils.copyInputStreamToFile(imageInputStream, new File(targetFile));//common-io包提供的写文件API
        } catch (Exception e) {
            e.printStackTrace();
        }
```

getPart方法根据表单的name属性获取表单内容。

```
		//getParts方法
		ArrayList<Part> images;
        try {
            images = (ArrayList<Part>) request.getParts();
            count = images.size();
            for (Part image : images) {
                InputStream imageInputStream = null;
                imageInputStream = image.getInputStream();
				FileUtils.copyInputStreamToFile(imageInputStream, new File(targetFile));
			}
        } catch (IOException e1) {
            e1.printStackTrace();
        }
```

getParts则获取到表单中的全部内容。

 - 前端

创建一个FormData对象，由于参数为表单的DOM对象，所以用JQuery选择器获取后应转换为DOM对象。

```
			$.ajax({
                type: 'post',
                url: 'updateAvatar.action',
                data: new FormData($('#form')[0]),
                processData: false,
                contentType: false,
                success: function(data) {
                    
                }
            });
```