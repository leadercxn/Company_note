# sensoro python应用笔记

+ `''' xxxx '''`的多行注释

+ ` form x0 import x1` 的应用
```
  从x0文件（.py 模块）中导入其中的x1作用块
```

+ `import requests `导入requests模块 （cmd: pip install requests 进行安装）<br>
  向第三方发送http请求的应用场景。
  + requests模块发送post请求,即向服务器发送数据
  ```
  requests.post(url=url, headers=headers, data=params)
  
    url：请求url地址
    headers：请求头
    data：发送编码为表单形式的数据
    
    举例：(向服务器发送数据)
    payload = {"key1":"cxn","keycxn":"diuleiloumou"}
    r1=requests.post("http://httpbin.org/post",data=payload)#因为返回的报文类型是 json 格式的类型，所以我们可以用 .json 来接收
    r1.json()
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
 
+ `if __name__ == '__main__' `的用法和理解
  ```
  __name__是一个变量。前后加了爽下划线是因为是因为这是系统定义的名字。普通变量不要使用此方式命名变量。
  Python有很多模块，而这些模块是可以独立运行的！这点不像C++和C的头文件。
  import的时候是要执行所import的模块的。
  __name__就是标识模块的名字的一个系统变量。这里分两种情况：假如当前模块是主模块（也就是调用其他模块的模块），那么此模块名字就是__main__，通过if判断这样就可以执行“__mian__:”后面的主函数内容；假如此模块是被import的，则此模块名字为文件名字（不加后面的.py），通过if判断这样就会跳过“__mian__:”后面的内容。
  跟C区别的在于，C的include .h的时候，不执行.h里面的内容 ，python 的import module时候，会执行module里面相关的函数

  举例：
  在main.py里面 有import module.py ,也有 if __name__ == '__main__'，
  那么在命令行执行 python main.py时， 主模块默认是main.py,那么会执行main.py里面相关的输出函数，而会屏蔽module里面相关的函数
  
  macprastic.py
  def hello():
    print "hello.cxn"
    
  if __name__ == "__main__":
    hello()
    
  macmain.py
  import mac_prastic

  if __name__ == "__main__":
    mac_prastic.hello()
  终端命令行：python macmain.py
  result： hello.cxn
  
  假如两个文件都没有if __name__ == "__main__":修饰，则会有两个打印
  ```
