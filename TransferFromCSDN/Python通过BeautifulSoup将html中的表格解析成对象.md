# Python 通过 BeautifulSoup 将 html 中的表格解析成对象

--- 

## 一、前言
> 我之前是写java 的，java的面向对象的思想感觉挺好的，所以决定用到Python上来，通过几天的学习，总结了一些东西，所以决定在这里和大家一起分享一下。

## 二、准备环境
1. python3.5安装，自行百度，不做太多描述
2. 开发工具，本人用的是MyEclipse2014(可自行选择)
3. pip-8.1.2工具安装（如果安装好后显示：不是内部命令之类的，建议重启电脑） 
4. 通过安装好之后的 pip 安装 BeautifulSoup 包
5. PyDev 4.0.0.zip插件集成到MyEclipse

**如果到这一步没什么问题的话，准备工作就做完了**

## 三、准备页面：index.html
``` html
<!DOCTYPE html>
<html>

    <head>
        <meta charset="utf-8" />
        <title>test</title>
    </head>

    <body>
        <table id="user">
            <tbody>
                <tr>
                    <th>序号</th>
                    <th>名称</th>
                    <th>年龄</th>
                </tr>
                <tr class="tr">
                    <td><input type="checkbox" id="1"/></td>
                    <td>jack</td>
                    <td>21</td>
                </tr>
                <tr class="tr">
                    <td><input type="checkbox" id="2"/></td>
                    <td>Neo</td>
                    <td>22</td>
                </tr>
                <tr class="tr">
                    <td><input type="checkbox" id="3"/></td>
                    <td>andy</td>
                    <td>23</td>
                </tr>
                <tr class="tr">
                    <td><input type="checkbox" id="4"/></td>
                    <td>emily</td>
                    <td>24</td>
                </tr>
                <tr class="tr">
                    <td><input type="checkbox" id="5"/></td>
                    <td>echo</td>
                    <td>25</td>
                </tr>
            </tbody>
        </table>
    </body>

</html>

```

## 四、实体类准备：User.py
``` python
'''
Created on 2016年8月27日

@author: Administrator
'''

class User(object):

    def __init__(self):
        self.id=None
        self.name=None
        self.age=None
        
    def getId(self):
        return self.id
    
    def setId(self,value):
        self.id=value
        
    def getName(self):
        return self.name
    
    def setName(self,value):
        self.name=value
    
    def delName(self):
        del self.name
        
    def getAge(self):
        return self.age
    
    def setAge(self,value):
        self.age=value
        
    def delAge(self):
        del self.age

    def __str__(self):
        return "id:"+self.id+",name:"+self.name+",age:"+str(self.age)
    

        
```

## 五、编写代码：GetUserListByBS4.py
``` python
'''
Created on 2016年8月27日

@author: Administrator
'''
import urllib
import urllib.request
from bs4 import BeautifulSoup
from com.office.oop.User import *

class GetUserListByBS4():

    def __init__(self):
        self.user=User()
        '''
        Constructor
        '''
    def get(self,url,coding):
        req=urllib.request.Request(url)
        response=urllib.request.urlopen(req)
        htmls=response.read()
        htm=htmls.decode(coding,'ignore')
        return htm
        
if __name__=="__main__":
    get=GetUserListByBS4()
    html=get.get("http://127.0.0.1:8020/HelloWorld/index.html", "utf-8")
    soup=BeautifulSoup(html,"html.parser")
    trs=soup.find_all(name='tr', attrs={'class':"tr"})
    userList=list()
    for tr in trs:
        user=User()
        _soup=BeautifulSoup(str(tr),"html.parser")
        tds=_soup.find_all(name='td')
        _id=_soup.input['id']
        user.setId(_id)
        user.setName(str(tds[1].string))
        user.setAge(str(tds[2].string))
        userList.append(user)
    
    for i in userList:
        print(i)
        
```

> 关于BeautifulSoup可参考：http://beautifulsoup.readthedocs.io/zh_CN/latest/
> 
> 本文章迁移于CSDN:[Python 通过 BeautifulSoup 将 html 中的表格解析成对象](https://mp.csdn.net/postedit/52347160)