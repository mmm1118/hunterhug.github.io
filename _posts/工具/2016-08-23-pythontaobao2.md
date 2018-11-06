---
layout: post
title: "Python3中级玩家：淘宝天猫商品搜索爬虫自动化工具（第二篇）"
date: 2016-08-23
author: silly
categories: ["工具"]
desc: "一只淘宝爬虫，相关原理和代码解释"
tags: ["爬虫","Python","淘宝"]
permalink: "/tool/spider-taobao2.html"
---

淘宝改字段，Bugfix，查看·[Github](https://github.com/hunterhug/taobaoscrapy.git)

请见第一篇：[Python3中级玩家：淘宝天猫商品搜索爬虫自动化工具（第一篇](https://www.lenggirl.com/spider/taobao1.html)

说完核心函数，我们接下来看其他函数。

认真看完这些函数之后，我们就可以开始工作了。

# 一、开始的介绍函数

    def begin():
        sangjin = '''
            -----------------------------------------
            | 欢迎使用自动抓取手机淘宝关键字程序       |
            | 时间：2015年12月23日                  |
            -----------------------------------------
        '''
        print(sangjin)
        
一开始是自我介绍，为了展示主权嘛~~~~

# 二、符合中国人思维的运行时间函数

接下来是测试运行了多长时间，对于爬数据来说，测试时间多久是必须的。

    def timetochina(longtime,formats='{}天{}小时{}分钟{}秒'):
        day=0
        hour=0
        minutue=0
        second=0
        try:
            if longtime>60:
                second=longtime%60
                minutue=longtime//60
            else:
                second=longtime
            if minutue>60:
                hour=minutue//60
                minutue=minutue%60
            if hour>24:
                day=hour//24
                hour=hour%24
            return formats.format(day,hour,minutue,second)
        except:
            raise Exception('时间非法')
            
我们传入了一个longtime是一个时间长度，默认是秒，如10000秒，如果小于60秒，那么不用转成分钟。否则先%，取余数则是秒数，//整除则是分钟。

分钟同样如果小于60，那么不用转化为小时，以此类推。。。

     return formats.format(day,hour,minutue,second)
     
我们传入了一个formats的字符串，进行字符串格式化即可得到符合中国人的时间！！

# 三、递归创建目录函数

## 递归创建文件夹

    def createjia(path):
        try:
            os.makedirs(path)
        except:
            print('目录已经存在：'+path)
            
大家知道makedir多一个s即是递归创建目录，只是因为爬虫生成的Excel和图片，要根据今天的日期进行存储，自然要生成新的目录，

这个函数直接解决找不到目录报错的bug！！

大家请注意，一个爬虫程序，都是斗智斗勇，动歪脑筋写出来的~~

# 四、去掉非法字符函数

## 去除标题中的非法字符 (Windows)

    def validateTitle(title):
        rstr = r"[\/\\\:\*\?\"\<\>\|]"  # '/\:*?"<>|'
        new_title = re.sub(rstr, "", title)
        return new_title
        
保存的图片需要命名，直接根据店铺名和商品名直接拼接就是不错，但是保存文件的时候，你是否会莫名出错。

没错，就是因为Window文件命名不允许什么/ \ : * ?这些字符，我们直接用正则表达式re.sub给替换掉，传入title，然后根据rstr中的字符替换为空字符。

    rstr = r"[\/\\\:\*\?\"\<\>\|]"
    
正则表达式[]表示里面任意一个字符，字符串外面的r表示里面数据要是原始的，这样出来才符合正则表达式的规则。

    Python 3.4.3 (v3.4.3:9b73f1c3e601, Feb 24 2015, 22:43:06) [MSC v.1600 32 bit (Intel)] on win32
    >>> rstr = r"[\/\\\:\*\?\"\<\>\|]"
    >>> print(rstr)
    [\/\\\:\*\?\"\<\>\|]
    >>> rstr = "[\/\\\:\*\?\"\<\>\|]"
    >>> print(rstr)
    [\/\\:\*\?"\<\>\|]
    
注意到加r和不加r的区别么，正则表达式\要转义再转义哦~~

# 五、找出某文件夹下指定后缀的全部文件函数（不递归）

## 找出文件夹下所有html后缀的文件

    def listfiles(rootdir, prefix='.xml'):
        file = []
        for parent, dirnames, filenames in os.walk(rootdir):
            if parent == rootdir:
                for filename in filenames:
                    if filename.endswith(prefix):
                        file.append(rootdir + '/' + filename)
                return file
            else:
                pass
                
以上函数难点在于os.walk(rootdir)，深层遍历递归文件夹，可是我只需要某目录下的文件就可以了，不需要目录下目录的文件。

那好来讲解，将下面源码放在test.py中

    import os
    for parent, dirnames, filenames in os.walk('./'):
        print(parent,dirnames,filenames,'\n');
        
运行上述语句，遍历本文件夹夹，可得：

    ./ ['sss'] ['cookie.txt', 'help.py', 'mtaobao.py', 'setup.py', 'test.py'] 
    
    ./sss [] ['help.py'] 
     
当然目录结构如下

<img  src="/picture/taobao/taobao12.png"/>

原来parent就是遍历到的文件夹，dirnames就是遍历到的文件夹下面的文件夹，filenames就是遍历到的文件夹下面的文件，瞬间懂了！继续看原来的代码：

    if parent == rootdir:
                for filename in filenames:
                    if filename.endswith(prefix):
                        file.append(rootdir + '/' + filename)
                return file
    else:
                pass
     

如果遍历到的文件夹就是我们要找的那个文件夹，那么就开始solo该文件夹下的文件，如果endswith指定后缀，那么把它加入列表file。

    file.append(rootdir + '/' + filename)
 
列表扫描了全部文件后，返回一个列表，我们要处理的就是这个列表里面的文件。

# 六、写入Excel文件函数

    def writeexcel(path,dealcontent):
        workbook = wx.Workbook(path)
        top = workbook.add_format({'border':1,'align':'center','bg_color':'white','font_size':11,'font_name': '微软雅黑'})
        red = workbook.add_format({'font_color':'white','border':1,'align':'center','bg_color':'800000','font_size':11,'font_name': '微软雅黑','bold':True})
        image = workbook.add_format({'border':1,'align':'center','bg_color':'white','font_size':11,'font_name': '微软雅黑'})
        formatt=top
        formatt.set_align('vcenter') #设置单元格垂直对齐
        worksheet = workbook.add_worksheet()        #创建一个工作表对象
        width=len(dealcontent[0])
        worksheet.set_column(0,width,38.5)            #设定列的宽度为22像素
        for i in range(0,len(dealcontent)):
            if i==0:
                formatt=red
            else:
                formatt=top
            for j in range(0,len(dealcontent[i])):
                if i!=0 and j==len(dealcontent[i])-1:
                    if dealcontent[i][j]=='':
                        worksheet.write(i,j,' ',formatt)
                    else:
                        try:
                            worksheet.insert_image(i,j,dealcontent[i][j])
                        except:
                            worksheet.write(i,j,' ',formatt)
                else:
                    if dealcontent[i][j]:
                        worksheet.write(i,j,dealcontent[i][j].replace(' ',''),formatt)
                    else:
                        worksheet.write(i,j,'无',formatt)
        workbook.close()
 

我们一行行来讲解，其中有的语句可能没用到哦，这个库也要自己安装，参见上篇：

    def writeexcel(path,dealcontent):
    
将内容delcontent保存名为path的文件，path是路径的意思哦

    workbook = wx.Workbook(path)
    
初始化对象，该EXCEL的保存路径为path。

    top = workbook.add_format({'border':1,'align':'center','bg_color':'white','font_size':11,'font_name': '微软雅黑'})
    red = workbook.add_format({'font_color':'white','border':1,'align':'center','bg_color':'800000','font_size':11,'font_name': '微软雅黑','bold':True})
    image = workbook.add_format({'border':1,'align':'center','bg_color':'white','font_size':11,'font_name': '微软雅黑'})
    formatt=top
    formatt.set_align('vcenter') #设置单元格垂直对齐
    
格式化处理，这些变量top,red任意取名哈，仅仅格式化为Excel的样式。

    worksheet = workbook.add_worksheet()        #创建一个工作表对象
    
加一个表，Excel是有很多表的。

    width=len(dealcontent[0])
    worksheet.set_column(0,width,38.5)            #设定列的宽度为22像素
    
判断要写入多少列，每列宽度22像素.

下面逻辑十分曲折，请人脑跑一遍：

    for i in range(0,len(dealcontent)):
        if i==0:
            formatt=red
        else:
            formatt=top
        for j in range(0,len(dealcontent[i])):
            if i!=0 and j==len(dealcontent[i])-1:
                if dealcontent[i][j]=='':
                    worksheet.write(i,j,' ',formatt)
                else:
                    try:
                        worksheet.insert_image(i,j,dealcontent[i][j])
                    except:
                        worksheet.write(i,j,' ',formatt)
            else:
                if dealcontent[i][j]:
                    worksheet.write(i,j,dealcontent[i][j].replace(' ',''),formatt)
                else:
                    worksheet.write(i,j,'无',formatt)
 
主要是以下语句重要，写入字符串到一格还有插入一图到一格：

                    try:
                        worksheet.insert_image(i,j,dealcontent[i][j])
                    except:
                        worksheet.write(i,j,' ',formatt)
 
好了，接下来再讲一个高级函数就可以结束，开始第三篇了。

 
# 七、验证函数

因为你不想让所有人都可以自由访问，你想授权，那么看下面：

    def password():
        print('请输入你的账号和密码')
        user=input('账号：')
        pwd=input('密码：')
        if user=='jinhan' and pwd=='6833066':
            print('欢迎你：'+user)
            return
        try:
            mysql = pymysql.connect(host="192.168.1.177", user="dataman", passwd="123456",db='qingmu', charset="utf8")
            cur = mysql.cursor()
            isuser="SELECT * FROM mtaobao where user='{0}' and pwd='{1}'".format(user,pwd)
            cur.execute(isuser)
            mysql.commit()
            if cur.fetchall():
                print('欢迎你：'+user)
                localIP = socket.gethostbyname(socket.gethostname())#这个得到本地ip
                ipList = socket.gethostbyname_ex(socket.gethostname())
                s=''
                for i in ipList:
                    if i != localIP and i!=[]:
                        s=s+(str)(i)
                timesss=time.strftime('%Y%m%d-%H%M%S', time.localtime())
                update="UPDATE mtaobao SET `times` = `times`+1,`dates`='{0}',`ip` ='{1}' where user='{2}'".format(timesss,s.replace("'",''),user)
                #print(update)
                cur.execute(update)
                mysql.commit()
                cur.close()
                mysql.close()
                return
            else:
                raise
        except Exception as e:
            #print(e)
            mysql.rollback()
            cur.close()
            mysql.close()
            print('密码错误')
            password()
 
先埋后门，如果是后门，那么不查数据库，返回。

    print('请输入你的账号和密码')
    user=input('账号：')
    pwd=input('密码：')
    if user=='jinhan' and pwd=='6833066':
        print('欢迎你：'+user)
        return

否则，连接数据库，查找用户和密码是否匹配到。

        mysql = pymysql.connect(host="192.168.1.177", user="dataman", passwd="123456",db='qingmu', charset="utf8")
        cur = mysql.cursor()
        isuser="SELECT * FROM mtaobao where user='{0}' and pwd='{1}'".format(user,pwd)
        cur.execute(isuser)
        mysql.commit()
 
如果匹配到

        if cur.fetchall():
            print('欢迎你：'+user)
            localIP = socket.gethostbyname(socket.gethostname())#这个得到本地ip
            ipList = socket.gethostbyname_ex(socket.gethostname())
            s=''
            for i in ipList:
                if i != localIP and i!=[]:
                    s=s+(str)(i)
            timesss=time.strftime('%Y%m%d-%H%M%S', time.localtime())
            update="UPDATE mtaobao SET `times` = `times`+1,`dates`='{0}',`ip` ='{1}' where user='{2}'".format(timesss,s.replace("'",''),user)

得到IP，得到时间，更新数据库，为了方便得知哪个人哪个时间使用了这个工具。

当然数据库要关闭什么的，如果密码出错，那么异常后还使用了递归，看到password吧，错误，重来！！

    except Exception as e:
        #print(e)
        mysql.rollback()
        cur.close()
        mysql.close()
        print('密码错误')
        password()

欢迎看了这么多函数。

下一篇将把所有函数连接在一起，进行我们爬虫工具的组装！！上面只是做预热准备~

明天持续第三篇：[Python3中级玩家：淘宝天猫商品搜索爬虫自动化工具（第三篇）](https://www.lenggirl.com/spider/taobao3.html）

等不及就上github:

git clone [https://github.com/hunterhug/taobaoscrapy.git](https://github.com/hunterhug/taobaoscrapy.git)

2016/3/27

中间去了深圳两天，稍后更新。。。

读到这里心酸酸,2017.03.25
