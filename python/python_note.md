# sensoro python应用笔记

+ `''' xxxx '''`的多行注释

+ ` form x0 import x1` 的应用
```
  从x0文件（.py 模块）中导入其中的x1作用块
```

+ `import requests `导入requests模块 （cmd: pip install requests 进行安装）<br>
  向第三方发送http请求的应用场景。
  + requests模块发送post请求
  ```
  requests.post(url=url, headers=headers, data=params)
  
    url：请求url地址
    headers：请求头
    data：发送编码为表单形式的数据
  ```
  
  + requests模块上传文件
  ```
  requests.post(url=url, headers=headers, data=params, files=files)
  
  参数说明：
  url：请求url地址
  headers：请求头
  data：发送编码为表单形式的数据
  files：上传的文件,如：
  files = {'upload_img': ('report.png', open('report.png', 'rb'), 'image/png')}
  参数说明：
  1.report.png：文件名
  2.open('report.png', 'rb')：文件内容
  3.image/png：文件类型
  ```

+ `class   def: __init__(self) `的用法<br>
self只能用在python类的方法（即函数）中。举例：
   ```
   class cxnprint:
    def __init__(self,envType='caonima'):
        self.domain = envType
    def sayhello(self):
        print 'you is :' ,self.domain
  p=cxnprint()
  q=cxnprint()
  q.domain='haha'
  print p.domain
  print q.sayhello()
  
  result:
  caonima
  you is : haha
  
  理解：self是继承class对象后的继承体的成员，而非类自身的成员。假如使用的是类的方法（函数），记得后面加（）号
   ```
   

   
