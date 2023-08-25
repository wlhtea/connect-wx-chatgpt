# chatpgt-
# python两分钟实现国内镜像chatgpt链接微信
#
前些日子在写剑魔题太烦了 找点东西玩玩 就想到了最近很火的chatgpt 
这个程序可改性很大 可以定时啊 很多好玩的 可以自行研究
#
效果图
![效果图](https://img-blog.csdnimg.cn/19933b562097412da7cbef56af688827.png)
**原理：**
利用wx自动化库和自动访问浏览器内镜像源对接
#
**主要调用库：**

> **wxauto：**
> 微信自动化的一个库 可以点开源码去看还是很清晰明了的 作者做了很详细注释
> Author: tikic@qq.com
Source:[https://github.com/cluic/wxauto](https://github.com/cluic/wxauto)
License: MIT License
Version: 3.3.5.3
#
> **selenium** 
> 用于浏览器自动化的一个库
> 一开始使用需要配置，配置完再搞少走十年弯路  [如何配置selenium](https://blog.csdn.net/flyskymood/article/details/124158706?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522168248050116800192229878%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=168248050116800192229878&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_positive~default-1-124158706-null-null.142%5Ev86%5Einsert_down1,239%5Ev2%5Einsert_chatgpt&utm_term=selenium%E5%AE%89%E8%A3%85&spm=1018.2226.3001.4187)

#

> 国内镜像网站：
> [https://laicj.cn/#/](https://laicj.cn/#/)

#
#
#

# selenium部分代码解读

[selenium使用方法大全](https://blog.csdn.net/qq_43965708/article/details/120658713?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522168248526916800180626565%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=168248526916800180626565&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_positive~default-1-120658713-null-null.142%5Ev86%5Einsert_down1,239%5Ev2%5Einsert_chatgpt&utm_term=selenium%E5%AE%9A%E4%BD%8D%E5%85%83%E7%B4%A0%E7%9A%84%E6%96%B9%E6%B3%95&spm=1018.2226.3001.4187)
> 本文只需要使用到
> 定位
> 点击
> 输入
> 清空

定位：

```python
wb = webdriver.Chrome()
wb.implicitly_wait(30) 
wb.get('https://laicj.cn/#/') #网址
wb.find_element(By.XPATH,'/html/body/div/section/main/div/div[1]/div[1]/div[1]/textarea')
```

> wb.implicitly_wait(30) :个人理解为每次定位点击等操作可以有30秒以内的误差时间 比如网页跳转需要时间 在这段时间内没有定位到元素会显示报错
> find_element：查找元素的意思
By.XPATH：就是按照xpath定位网页中的元素
元素：可以在控制台找到右键有一个**copy xpath/ full xpath** 如下图
![类似这样](https://img-blog.csdnimg.cn/2d56823f35b843ae84c179bcba2f079b.jpeg)


点击：
```python
wb.find_element(By.XPATH,'/html/body/div/section/main/div/div[1]/div[1]/button').click()
```
在定位元素后面直接加.click()

#

输入：
```python
wb.find_element(By.XPATH,'/html/body/div/section/main/div/div[1]/div[1]/div[1]/textarea').send_keys(f'{data_msg}')
```
在定位元素后面直接加.send_key('内容')
#
清空：

```python
wb.find_element(By.XPATH,'/html/body/div/section/main/div/div[1]/div[1]/div[1]/textarea').clear()
```
在定位元素后面直接加.clear()

最后我在源代码中加入了判断语句 为的是判断ai是否写完 如果写完则返回内容

```python
    def yes_or_no():
        a = wb.find_element(By.XPATH,'/html/body/div/section/main/div/div[1]/div[2]/div/p')
        time.sleep(10)
        b = wb.find_element(By.XPATH,'/html/body/div/section/main/div/div[1]/div[2]/div/p')
        if a == b:
            return a.text
```
#
：
#
：
#

# **wxauto部分代码解读**

```python
wx = wxauto.WeChat() #获取微信窗口
wx.GetSessionList()  #获取微信聊天的名字 存到表内 具体看源码很清晰易懂
```
#

```python
wx.Search('Bot') #输入你要找到的联系人
```

```python
data = wx.GetAllMessage #获取聊天记录
msg = data[-1][1]       #聊天记录最后一条  库里有专门获取最后一条的方法GetLastMessage
#为了以后继续改进方便 我就没有用那个方法了
```

```python
if data[-1][0] != '伟大的军事家':  #当自己发信息的时候就不会回复
    if msg not in k:               #避免重复提问ai 每次提问就加进k 避免无限循环
        k.append(msg)
        k.pop(0)#每次k里面只会有一个值 这样如果对方重复提问一个相同的问题也是能够回答的
        res = chat_give(msg)       #这是一个方法  把内容发给ai
        k.append(res)
        k.pop(0)
```
#
‘’‘’‘’‘’‘’‘’‘’‘’‘’‘’‘’‘’‘’‘’‘’‘’‘’‘’‘’‘’‘’‘’‘’‘’‘’‘’‘’‘’‘’‘’‘’‘’‘’‘’‘’‘’‘’‘’‘’‘’‘’‘’‘’‘’‘’‘’‘’‘’‘’‘’‘’‘’‘’‘’‘’‘’‘’‘’‘’‘’‘’‘’‘’‘’‘’‘’‘’‘’‘’‘’‘’‘’‘’‘’‘’‘’‘’‘’‘’‘’‘’‘’‘’‘’‘’‘’‘’‘’‘’‘’‘’‘’‘

## 完整代码：


```python
import time
from selenium import webdriver
from selenium.webdriver.common.by import By
import wxauto

wb = webdriver.Chrome()
wb.implicitly_wait(30)
wb.get('https://laicj.cn/#/')
def chat_give(data_msg):
    wb.find_element(By.XPATH,'/html/body/div/section/main/div/div[1]/div[1]/div[1]/textarea').send_keys(f'{data_msg}')
    wb.find_element(By.XPATH,'/html/body/div/section/main/div/div[1]/div[1]/button').click()
    wb.find_element(By.XPATH,'/html/body/div/section/main/div/div[1]/div[1]/div[1]/textarea').clear()
    time.sleep(2)
    def yes_or_no():
        a = wb.find_element(By.XPATH,'/html/body/div/section/main/div/div[1]/div[2]/div/p')
        time.sleep(10)
        b = wb.find_element(By.XPATH,'/html/body/div/section/main/div/div[1]/div[2]/div/p')
        if a == b:
            return a.text
    return yes_or_no()
    
wx = wxauto.WeChat()
wx.GetSessionList()

k = []
wx.Search('Bot')
while 1:
    data = wx.GetAllMessage
    msg = data[-1][1]
    if data[-1][0] != '伟大的军事家':
        if msg not in k:
            k.append(msg)
            k.pop(0)
            res = chat_give(msg)
            k.append(res)
            k.pop(0)
            wx.SendMsg(res)
 

```

简单的实现距离商业化还很远 内有许多测试还不能通过 
比如
如果网卡 会直接输出未完成的ai对话内容 
不允许太多人同时问答，只能一次回答一个
每次只能对一个联系人问答

#
再接再厉 记得点赞关注！
