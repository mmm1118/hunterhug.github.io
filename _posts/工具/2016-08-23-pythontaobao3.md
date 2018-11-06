---
layout: post
title: "Python3中级玩家：淘宝天猫商品搜索爬虫自动化工具（第三篇）"
date: 2016-08-23
author: silly
categories: ["工具"]
desc: "一只淘宝爬虫，相关原理和代码解释"
tags: ["爬虫","Python","淘宝"]
permalink: "/tool/spider-taobao3.html"
---

淘宝改字段，Bugfix，查看[Github](https://github.com/hunterhug/taobaoscrapy.git)

欢迎回看第一篇和第二篇。

Python3中级玩家：[Python3中级玩家：淘宝天猫商品搜索爬虫自动化工具（第一篇](https://www.lenggirl.com/spider/taobao1.html)

Python3中级玩家：[Python3中级玩家：淘宝天猫商品搜索爬虫自动化工具（第二篇](https://www.lenggirl.com/spider/taobao2.html)

这一篇是终极篇，看完这个你就知道一个爬虫，是那么地需要，灵活运用，各种各样的工具。

动动歪脑筋，现在开始main函数。

使用Python3哈，大家先看下面的代码，我再放一段如何debug http数据传输的入口。

    if __name__ == '__main__':
        begin()
        password()
        today=time.strftime('%Y%m%d', time.localtime())
        a=time.clock()
        keyword = input('请输入关键字：')
        sort = input('按销量优先请按1，按价格低到高抓取请按2，价格高到低按3，信用排序按4，综合排序按5：')
        try:
            pages =int(input('需要抓取的页数（默认100页）：'))
            if pages>100 or pages<=0:
                print('页数应该在1-100之间')
                pages=100
        except:
            pages=100
        try:
            man=int(input('请设置抓取暂停时间：默认4秒（4）：'))
            if man<=0:
                man=4
        except:
            man=4
        zp=input('抓取图片按1，不抓取按2：')
        if sort == '1':
            sortss = '_sale'
        elif sort == '2':
            sortss = 'bid'
        elif sort=='3':
            sortss='_bid'
        elif sort=='4':
            sortss='_ratesum'
        elif sort=='5':
            sortss=''
        else:
            sortss = '_sale'
        namess=time.strftime('%Y%m%d%H%S', time.localtime())
        root = '../data/'+today+'/'+namess+keyword
        roota='../excel/'+today
        mulu='../image/'+today+'/'+namess+keyword
        createjia(root)
        createjia(roota)
        for page in range(0, pages):
            time.sleep(man)
            print('暂停'+str(man)+'秒')
            if sortss=='':
                postdata = {
                    'event_submit_do_new_search_auction': 1,
                    'search': '提交查询',
                    '_input_charset': 'utf-8',
                    'topSearch': 1,
                    'atype': 'b',
                    'searchfrom': 1,
                    'action': 'home:redirect_app_action',
                    'from': 1,
                    'q': keyword,
                    'sst': 1,
                    'n': 20,
                    'buying': 'buyitnow',
                    'm': 'api4h5',
                    'abtest': 16,
                    'wlsort': 16,
                    'style': 'list',
                    'closeModues': 'nav,selecthot,onesearch',
                    'page': page
                }
            else:
                postdata = {
                    'event_submit_do_new_search_auction': 1,
                    'search': '提交查询',
                    '_input_charset': 'utf-8',
                    'topSearch': 1,
                    'atype': 'b',
                    'searchfrom': 1,
                    'action': 'home:redirect_app_action',
                    'from': 1,
                    'q': keyword,
                    'sst': 1,
                    'n': 20,
                    'buying': 'buyitnow',
                    'm': 'api4h5',
                    'abtest': 16,
                    'wlsort': 16,
                    'style': 'list',
                    'closeModues': 'nav,selecthot,onesearch',
                    'sort': sortss,
                    'page': page
                }
            postdata = urllib.parse.urlencode(postdata)
            taobao = "http://s.m.taobao.com/search?" + postdata
            print(taobao)
            try:
                content1 = getHtml(taobao)
                file = open(root + '/' + str(page) + '.json', 'wb')
                file.write(content1)
            except Exception as e:
                    if hasattr(e, 'code'):
                        print('页面不存在或时间太长.')
                        print('Error code:', e.code)
                    elif hasattr(e, 'reason'):
                            print("无法到达主机.")
                            print('Reason:  ', e.reason)
                    else:
                        print(e)
    
        # files=listfiles('201512171959','.json')
        files = listfiles(root, '.json')
        total = []
        total.append(['页数', '店名', '商品标题', '商品打折价', '发货地址', '评论数', '原价', '手机折扣', '售出件数', '政策享受', '付款人数', '金币折扣','URL地址','图像URL','图像'])
        for filename in files:
            try:
                doc = open(filename, 'rb')
                doccontent = doc.read().decode('utf-8', 'ignore')
                product = doccontent.replace(' ', '').replace('\n', '')
                product = json.loads(product)
                onefile = product['listItem']
            except:
                print('抓不到' + filename)
                continue
            for item in onefile:
                itemlist = [filename, item['nick'], item['title'], item['price'], item['location'], item['commentCount']]
                itemlist.append(item['originalPrice'])
                itemlist.append(item['mobileDiscount'])
                itemlist.append(item['sold'])
                itemlist.append(item['zkType'])
                itemlist.append(item['act'])
                itemlist.append(item['coinLimit'])
                itemlist.append(item['auctionURL'])
                picpath=item['pic_path'].replace('60x60','720x720')
                itemlist.append(picpath)
                #http://g.search2.alicdn.com/img/bao/uploaded/i4/i4/TB13O7bJVXXXXbJXpXXXXXXXXXX_%21%210-item_pic.jpg_180x180.jpg
                if zp=='1':
                    if os.path.exists(mulu):
                        pass
                    else:
                        createjia(mulu)
                    url=urllib.parse.quote(picpath).replace('%3A',':')
                    urllib.request.urlcleanup()
                    try:
                        pic=urllib.request.urlopen(url)
                        picno=time.strftime('%H%M%S', time.localtime())
                        filenamep=mulu+'/'+picno+validateTitle(item['nick']+'-'+item['title'])
                        filenamepp=filenamep+'.jpeg'
                        sfilename=filenamep+'s.jpeg'
                        filess=open(filenamepp,'wb')
                        filess.write(pic.read())
                        filess.close()
                        img = Image.open(filenamepp)
                        w, h = img.size
                        size=w/6,h/6
                        img.thumbnail(size, Image.ANTIALIAS)
                        img.save(sfilename,'jpeg')
                        itemlist.append(sfilename)
                        print('抓到图片：'+sfilename)
                    except Exception as e:
                        if hasattr(e, 'code'):
                            print('页面不存在或时间太长.')
                            print('Error code:', e.code)
                        elif hasattr(e, 'reason'):
                                print("无法到达主机.")
                                print('Reason:  ', e.reason)
                        else:
                            print(e)
                        itemlist.append('')
                else:
                    itemlist.append('')
                # print(itemlist)
                total.append(itemlist)
        if len(total) > 1:
            writeexcel(roota +'/'+namess+keyword+ '淘宝手机商品.xlsx', total)
        else:
            print('什么都抓不到')
        b=time.clock()
        print('运行时间：'+timetochina(b-a))
        input('请关闭窗口')
 
好，先打开火狐浏览器，输入

    http://s.m.taobao.com/

Shift+Ctrl+M变成手机形式，然后模拟触摸事件

<img  src="/picture/taobao/taobao13.png"/>

现在好像在PC机上不能搜索宝贝了，谁怕，按F12

看到下面的post参数没有

<img  src="/picture/taobao/taobao14.png"/>

然后看到JSON数据没有。

<img  src="/picture/taobao/taobao15.png"/>

我们从中间代码剖析。

        else:
            postdata = {
                'event_submit_do_new_search_auction': 1,
                'search': '提交查询',
                '_input_charset': 'utf-8',
                'topSearch': 1,
                'atype': 'b',
                'searchfrom': 1,
                'action': 'home:redirect_app_action',
                'from': 1,
                'q': keyword,
                'sst': 1,
                'n': 20,
                'buying': 'buyitnow',
                'm': 'api4h5',
                'abtest': 16,
                'wlsort': 16,
                'style': 'list',
                'closeModues': 'nav,selecthot,onesearch',
                'sort': sortss,
                'page': page
            }
        postdata = urllib.parse.urlencode(postdata)
        taobao = "http://s.m.taobao.com/search?" + postdata
        print(taobao)
 
keyword是搜索关键字，sort是排序方式，page是第几页，默认100页之后是没有的，要观察。

然后还支持价格索引，发货地址索引，大家自己抓包Debug哈。

urlencode是因为关键字可能是汉字或非法字符，需要先转义一下。

打印出来的效果是这样的

<img  src="/picture/taobao/taobao16.png"/>
 
好！！我们从一开始的main剖析。

    if __name__ == '__main__':
        begin()
        password()
        today=time.strftime('%Y%m%d', time.localtime())
        a=time.clock()
        b=time.clock()
        print('运行时间：'+timetochina(b-a))
        input('请关闭窗口')
 

begin开始欢迎信息

password开始验证用户

clock()开始计时，看程序运行时间

today是今天的日期，格式为年月日，建文件夹要用到

最后的input是为了防止运行后直接就结束了，你都没时间看运行时间

函数参考上篇。

接下来搜索限制：

    keyword = input('请输入关键字：')
    sort = input('按销量优先请按1，按价格低到高抓取请按2，价格高到低按3，信用排序按4，综合排序按5：')
    try:
        pages =int(input('需要抓取的页数（默认100页）：'))
        if pages>100 or pages<=0:
            print('页数应该在1-100之间')
            pages=100
    except:
        pages=100
    try:
        man=int(input('请设置抓取暂停时间：默认4秒（4）：'))
        if man<=0:
            man=4
    except:
        man=4
    zp=input('抓取图片按1，不抓取按2：')
    if sort == '1':
        sortss = '_sale'
    elif sort == '2':
        sortss = 'bid'
    elif sort=='3':
        sortss='_bid'
    elif sort=='4':
        sortss='_ratesum'
    elif sort=='5':
        sortss=''
    else:
        sortss = '_sale'
 
逐行分析：

    try:
        pages =int(input('需要抓取的页数（默认100页）：'))
        if pages>100 or pages<=0:
            print('页数应该在1-100之间')
            pages=100
    except:
        pages=100
        
页数如果输出的不是数字，异常，默认100页，

是数字但是超过100或者是负数，也是默认100页，不爽来战。

    try:
        man=int(input('请设置抓取暂停时间：默认4秒（4）：'))
        if man<=0:
            man=4
    except:
        man=4
        
同样，抓取要暂停时间，不能抓太快啊，会被反爬的！四秒是默认。

    keyword = input('请输入关键字：')
    sort = input('按销量优先请按1，按价格低到高抓取请按2，价格高到低按3，信用排序按4，综合排序按5：')
    zp=input('抓取图片按1，不抓取按2：')
    if sort == '1':
        sortss = '_sale'
    elif sort == '2':
        sortss = 'bid'
    elif sort=='3':
        sortss='_bid'
    elif sort=='4':
        sortss='_ratesum'
    elif sort=='5':
        sortss=''
    else:
        sortss = '_sale'
        
上面这个重点在于sort，debug时总结的，如果综合排序那么sortss=''，默认按销量排序。

抓图片，不抓图片，是抓还是不抓，自己决定！

下面是抓取数据的储存地

    namess=time.strftime('%Y%m%d%H%S', time.localtime())
    root = '../data/'+today+'/'+namess+keyword
    roota='../excel/'+today
    mulu='../image/'+today+'/'+namess+keyword
    createjia(root)
    createjia(roota)
 
看上面再看下面，today是今天的日期，namess+keyword存放今天哪个小时那一分钟抓的什么关键字的数据。

root变量存放原始数据

roota存放Excel

mulu存放图片

createjia是创建文件夹，不存在会报错的！！

<img  src="/picture/taobao/taobao17.png"/>

关键boss来了，看好：

    for page in range(0, pages):
            time.sleep(man)
            print('暂停'+str(man)+'秒')
            if sortss=='':
                postdata = {
                    'event_submit_do_new_search_auction': 1,
                    'search': '提交查询',
                    '_input_charset': 'utf-8',
                    'topSearch': 1,
                    'atype': 'b',
                    'searchfrom': 1,
                    'action': 'home:redirect_app_action',
                    'from': 1,
                    'q': keyword,
                    'sst': 1,
                    'n': 20,
                    'buying': 'buyitnow',
                    'm': 'api4h5',
                    'abtest': 16,
                    'wlsort': 16,
                    'style': 'list',
                    'closeModues': 'nav,selecthot,onesearch',
                    'page': page
                }
            else:
                postdata = {
                    'event_submit_do_new_search_auction': 1,
                    'search': '提交查询',
                    '_input_charset': 'utf-8',
                    'topSearch': 1,
                    'atype': 'b',
                    'searchfrom': 1,
                    'action': 'home:redirect_app_action',
                    'from': 1,
                    'q': keyword,
                    'sst': 1,
                    'n': 20,
                    'buying': 'buyitnow',
                    'm': 'api4h5',
                    'abtest': 16,
                    'wlsort': 16,
                    'style': 'list',
                    'closeModues': 'nav,selecthot,onesearch',
                    'sort': sortss,
                    'page': page
                }
            postdata = urllib.parse.urlencode(postdata)
            taobao = "http://s.m.taobao.com/search?" + postdata
            print(taobao)
            try:
                content1 = getHtml(taobao)
                file = open(root + '/' + str(page) + '.json', 'wb')
                file.write(content1)
            except Exception as e:
                    if hasattr(e, 'code'):
                        print('页面不存在或时间太长.')
                        print('Error code:', e.code)
                    elif hasattr(e, 'reason'):
                            print("无法到达主机.")
                            print('Reason:  ', e.reason)
                    else:
                        print(e)
     
先睡觉一段时间，再抓，循环是从0到pages，pages是页数，构造参数会使用到。

    for page in range(0, pages):
        time.sleep(man)
        print('暂停'+str(man)+'秒')
        if sortss=='':
            postdata = {
                'event_submit_do_new_search_auction': 1,
                'search': '提交查询',
                '_input_charset': 'utf-8',
                'topSearch': 1,
                'atype': 'b',
                'searchfrom': 1,
                'action': 'home:redirect_app_action',
                'from': 1,
                'q': keyword,
                'sst': 1,
                'n': 20,
                'buying': 'buyitnow',
                'm': 'api4h5',
                'abtest': 16,
                'wlsort': 16,
                'style': 'list',
                'closeModues': 'nav,selecthot,onesearch',
                'page': page
            }
 
因为综合排序和其他排序有差异，它没有sort这个post参数，所以弄了个if和else做区分。

        else:
            postdata = {
                'event_submit_do_new_search_auction': 1,
                'search': '提交查询',
                '_input_charset': 'utf-8',
                'topSearch': 1,
                'atype': 'b',
                'searchfrom': 1,
                'action': 'home:redirect_app_action',
                'from': 1,
                'q': keyword,
                'sst': 1,
                'n': 20,
                'buying': 'buyitnow',
                'm': 'api4h5',
                'abtest': 16,
                'wlsort': 16,
                'style': 'list',
                'closeModues': 'nav,selecthot,onesearch',
                'sort': sortss,
                'page': page
            }
        postdata = urllib.parse.urlencode(postdata)
        taobao = "http://s.m.taobao.com/search?" + postdata
 
下面开始抓这个链接，然后把抓到的数据放在data下，保存为json。

        try:
            content1 = getHtml(taobao)
            file = open(root + '/' + str(page) + '.json', 'wb')
            file.write(content1)
        except Exception as e:
                if hasattr(e, 'code'):
                    print('页面不存在或时间太长.')
                    print('Error code:', e.code)
                elif hasattr(e, 'reason'):
                        print("无法到达主机.")
                        print('Reason:  ', e.reason)
                else:
                    print(e)
                    
如果出现错误了，看看错误有没有code这个属性，有的话就证明访问服务器成功，但是会出现404,403等东西。

如果有reason则是你网络有问题，无法访问服务器。

保存的数据如下(太长不放了)：

    http://s.m.taobao.com/search?q=1&abtest=16&search=%E6%8F%90%E4%BA%A4%E6%9F%A5%E8%AF%A2&topSearch=1&style=list&sst=1&atype=b&n=20&page=0&closeModues=nav%2Cselecthot%2Conesearch&_input_charset=utf-8&sort=bid&buying=buyitnow&searchfrom=1&from=1&m=api4h5&event_submit_do_new_search_auction=1&action=home%3Aredirect_app_action&wlsort=16

稍后需要解析这些东西，拆分插到Excel。

    files = listfiles(root, '.json')
    total = []
    total.append(['页数', '店名', '商品标题', '商品打折价', '发货地址', '评论数', '原价', '手机折扣', '售出件数', '政策享受', '付款人数', '金币折扣','URL地址','图像URL','图像'])
    for filename in files:
        try:
            doc = open(filename, 'rb')
            doccontent = doc.read().decode('utf-8', 'ignore')
            product = doccontent.replace(' ', '').replace('\n', '')
            product = json.loads(product)
            onefile = product['listItem']
        except:
            print('抓不到' + filename)
            continue
        for item in onefile:
            itemlist = [filename, item['nick'], item['title'], item['price'], item['location'], item['commentCount']]
            itemlist.append(item['originalPrice'])
            itemlist.append(item['mobileDiscount'])
            itemlist.append(item['sold'])
            itemlist.append(item['zkType'])
            itemlist.append(item['act'])
            itemlist.append(item['coinLimit'])
            itemlist.append(item['auctionURL'])
            picpath=item['pic_path'].replace('60x60','720x720')
            itemlist.append(picpath)
            #http://g.search2.alicdn.com/img/bao/uploaded/i4/i4/TB13O7bJVXXXXbJXpXXXXXXXXXX_%21%210-item_pic.jpg_180x180.jpg
            if zp=='1':
                if os.path.exists(mulu):
                    pass
                else:
                    createjia(mulu)
                url=urllib.parse.quote(picpath).replace('%3A',':')
                urllib.request.urlcleanup()
                try:
                    pic=urllib.request.urlopen(url)
                    picno=time.strftime('%H%M%S', time.localtime())
                    filenamep=mulu+'/'+picno+validateTitle(item['nick']+'-'+item['title'])
                    filenamepp=filenamep+'.jpeg'
                    sfilename=filenamep+'s.jpeg'
                    filess=open(filenamepp,'wb')
                    filess.write(pic.read())
                    filess.close()
                    img = Image.open(filenamepp)
                    w, h = img.size
                    size=w/6,h/6
                    img.thumbnail(size, Image.ANTIALIAS)
                    img.save(sfilename,'jpeg')
                    itemlist.append(sfilename)
                    print('抓到图片：'+sfilename)
                except Exception as e:
                    if hasattr(e, 'code'):
                        print('页面不存在或时间太长.')
                        print('Error code:', e.code)
                    elif hasattr(e, 'reason'):
                            print("无法到达主机.")
                            print('Reason:  ', e.reason)
                    else:
                        print(e)
                    itemlist.append('')
            else:
                itemlist.append('')
            # print(itemlist)
            total.append(itemlist)
    if len(total) > 1:
        writeexcel(roota +'/'+namess+keyword+ '淘宝手机商品.xlsx', total)
    else:
        print('什么都抓不到')
 
逐行分析。

    files = listfiles(root, '.json')
    total = []
    total.append(['页数', '店名', '商品标题', '商品打折价', '发货地址', '评论数', '原价', '手机折扣', '售出件数', '政策享受', '付款人数', '金币折扣','URL地址','图像URL','图像'])
    
root变量是存放原始数据目录，从该目录找出所有格式为json的文件。

total变量存放Excel数据，待生成Excel

首行当然是解释啦，页数，店名，商品标题什么的。。。

    for filename in files:
        try:
            doc = open(filename, 'rb')
            doccontent = doc.read().decode('utf-8', 'ignore')
            product = doccontent.replace(' ', '').replace('\n', '')
            product = json.loads(product)
            onefile = product['listItem']
        except:
            print('抓不到' + filename)
            continue
 
开始循环原始数据文件，以二进制open()打开，为什么？因为不那样的话有些数据编码乱七八糟，还是二进制，然后decode转成utf-8，并且加上ignore参数，忽视可能出现的

转码出错。

            doc = open(filename, 'rb')
            doccontent = doc.read().decode('utf-8', 'ignore')
 
然后替换掉一些空格，使其更符合json数据

            product = doccontent.replace(' ', '').replace('\n', '')
            product = json.loads(product)
            onefile = product['listItem']
            
使用json.loads加载这个数据，然后就可以像对象一样操作json数据，'listItem'存放了我们需要的数据，看JSON数据格式组成：

<img  src="/picture/taobao/taobao18.png"/>
<img  src="/picture/taobao/taobao19.png"/>

好的！好多码呀，好复杂。。。。

        for item in onefile:
            itemlist = [filename, item['nick'], item['title'], item['price'], item['location'], item['commentCount']]
            itemlist.append(item['originalPrice'])
            itemlist.append(item['mobileDiscount'])
            itemlist.append(item['sold'])
            itemlist.append(item['zkType'])
            itemlist.append(item['act'])
            itemlist.append(item['coinLimit'])
            itemlist.append(item['auctionURL'])
            picpath=item['pic_path'].replace('60x60','720x720')
            itemlist.append(picpath)
            
循环出每一个商品信息，组装到itemlist列表里面，json里面还有很多隐藏的字段没有用到。

json每一个商品信息格式如下：

    {
            "pos": 0,
            "sold": "2",
            "userType": "0",
            "item_id": "521020560570",
            "nick": "wangjingli327",
            "userId": "85356923",
            "quantity": "",
            "shipping": "12.00",
            "ratesum": "",
            "isCod": "",
            "isprepay": "",
            "promotedService": "",
            "auctionTag": "",
            "clickUrl": "",
            "dsrScore": "",
            "zkType": "",
            "zkGroup": "",
            "autoPost": "",
            "commentCount": "2",
            "ordinaryPostFee": "",
            "distance": "",
            "zkRate": "",
            "zkTime": "",
            "promotions": "",
            "isInLimitPromotion": "",
            "pre_title_color": "",
            "pre_title": "",
            "h5Url": "",
            "isO2o": "",
            "recommendReason": "",
            "recommendColor": "",
            "recommendType": "",
            "location": "浙江 绍兴",
            "price": "298.00",
            "priceColor": "#000000",
            "long_title": "",
            "isP4p": "false",
            "sellerLoc": "浙江 绍兴",
            "fastPostFee": "12.00",
            "title": "性感透视奢华蕾丝深v领长袖公主新娘婚纱礼服2015冬季新款2518",
            "sameCount": "",
            "spuId": "",
            "similarCount": "",
            "priceWithRate": "",
            "pic_path": "http://g.search.alicdn.com/img/bao/uploaded/i4/i3/TB1c1E8IFXXXXbsXFXXXXXXXXXX_!!0-item_pic.jpg_60x60.jpg",
            "uprightImg": "",
            "priceWap": "298.00",
            "auctionFlag": "",
            "mobileDiscount": "",
            "coinInfo": "",
            "auctionURL": "http://a.m.taobao.com/i521020560570.htm?&abtest=16&sid=6334516654ffcd5000185f5604cc1d25",
            "type": "fixed",
            "isB2c": "0",
            "iconList": "xfbug",
            "uniqpid": "",
            "maxShopGift": "",
            "goodRate": "",
            "category": "162701",
            "newDsr": "",
            "desScore": "",
            "o2oShopId": "",
            "scoref": "",
            "sellerCount": "",
            "tagInfo": "",
            "area": "绍兴",
            "realSales": "",
            "banditScore": "",
            "totalSold": "",
            "name": "性感透视奢华蕾丝深v领长袖公主新娘婚纱礼服2015冬季新款2518",
            "img2": "//gw1.alicdn.com/bao/uploaded/i3/TB1c1E8IFXXXXbsXFXXXXXXXXXX_!!0-item_pic.jpg",
            "iswebp": "",
            "url": "//a.m.taobao.com/i521020560570.htm?sid=6334516654ffcd5000185f5604cc1d25&rn=b360d891c5ca145a452cf547fa10ef24&abtest=16",
            "previewUrl": "//a.m.taobao.com/ajax/pre_view.do?sid=6334516654ffcd5000185f5604cc1d25&itemId=521020560570&abtest=16",
            "favoriteUrl": "//fav.m.taobao.com/favorite/to_collection.htm?sid=6334516654ffcd5000185f5604cc1d25&itemNumId=521020560570&abtest=16",
            "originalPrice": "298.00",
            "freight": "12.00",
            "act": "2",
            "itemNumId": "521020560570",
            "wwimUrl": "//im.m.taobao.com/ww/ad_ww_dialog.htm?item_num_id=521020560570&amp;to_user=d2FuZ2ppbmdsaTMyNw%3D%3D",
            "isMobileEcard": "false",
            "auctionType": "b",
            "coinLimit": "100",
            "collect": "",
            "assess": "",
            "recommendGuy": "",
            "pricePerUnit": "",
            "collocation": "",
            "daySold": "",
            "inStock": "",
            "extendPid": "",
            "from": "",
            "recommendLabel": ""
        }
     
当然商品图片url

    http://g.search2.alicdn.com/img/bao/uploaded/i4/i4/TB13O7bJVXXXXbJXpXXXXXXXXXX_%21%210-item_pic.jpg_60x60.jpg

可以改成

    http://g.search2.alicdn.com/img/bao/uploaded/i4/i4/TB13O7bJVXXXXbJXpXXXXXXXXXX_%21%210-item_pic.jpg_180x180.jpg
    http://g.search2.alicdn.com/img/bao/uploaded/i4/i4/TB13O7bJVXXXXbJXpXXXXXXXXXX_%21%210-item_pic.jpg_720x720.jpg
 
哈哈哈，好了！！数据都解析好了。

现在开始抓图：

            if zp=='1':
                if os.path.exists(mulu):
                    pass
                else:
                    createjia(mulu)
                url=urllib.parse.quote(picpath).replace('%3A',':')
                urllib.request.urlcleanup()
                try:
                    pic=urllib.request.urlopen(url)
                    picno=time.strftime('%H%M%S', time.localtime())
                    filenamep=mulu+'/'+picno+validateTitle(item['nick']+'-'+item['title'])
                    filenamepp=filenamep+'.jpeg'
                    sfilename=filenamep+'s.jpeg'
                    filess=open(filenamepp,'wb')
                    filess.write(pic.read())
                    filess.close()
                    img = Image.open(filenamepp)
                    w, h = img.size
                    size=w/6,h/6
                    img.thumbnail(size, Image.ANTIALIAS)
                    img.save(sfilename,'jpeg')
                    itemlist.append(sfilename)
                    print('抓到图片：'+sfilename)
                except Exception as e:
                    if hasattr(e, 'code'):
                        print('页面不存在或时间太长.')
                        print('Error code:', e.code)
                    elif hasattr(e, 'reason'):
                            print("无法到达主机.")
                            print('Reason:  ', e.reason)
                    else:
                        print(e)
                    itemlist.append('')
            else:
                itemlist.append('')
 
如果需要抓图那么执行抓图程序，否则后面append('')表示没有图片。

            if zp=='1':
                #抓图
            else:
                itemlist.append('')
            # print(itemlist)
            total.append(itemlist)
            
抓完图后，total需要把这些商品信息拼在一起，本来是一行行的，现在拼在一起就像一个矩阵，和EXCEL里面一模一样，等着写入EXCEL。

抓图代码：

            if zp=='1':
                if os.path.exists(mulu):
                    pass
                else:
                    createjia(mulu)
                url=urllib.parse.quote(picpath).replace('%3A',':')
                urllib.request.urlcleanup()
                try:
                    pic=urllib.request.urlopen(url)
                    picno=time.strftime('%H%M%S', time.localtime())
                    filenamep=mulu+'/'+picno+validateTitle(item['nick']+'-'+item['title'])
                    filenamepp=filenamep+'.jpeg'
                    sfilename=filenamep+'s.jpeg'
                    filess=open(filenamepp,'wb')
                    filess.write(pic.read())
                    filess.close()
                    img = Image.open(filenamepp)
                    w, h = img.size
                    size=w/6,h/6
                    img.thumbnail(size, Image.ANTIALIAS)
                    img.save(sfilename,'jpeg')
                    itemlist.append(sfilename)
                    print('抓到图片：'+sfilename)
                except Exception as e:
                    if hasattr(e, 'code'):
                        print('页面不存在或时间太长.')
                        print('Error code:', e.code)
                    elif hasattr(e, 'reason'):
                            print("无法到达主机.")
                            print('Reason:  ', e.reason)
                    else:
                        print(e)
                    itemlist.append('')
 
首先创建存放图片的文件夹

                if os.path.exists(mulu):
                    pass
                else:
                    createjia(mulu)
 
然后需要将图片url变成正常规则的url，然后:被转义成%3A，需要再变回来

urlcleanup则是清除全局设置，因为淘宝天猫抓图片不需要任何附加头部，session什么的，哈哈哈，不加这个可能有问题喔。

                url=urllib.parse.quote(picpath).replace('%3A',':')
                urllib.request.urlcleanup()
                
quote函数说明：

<img  src="/picture/taobao/taobao20.png"/>

最后，创建文件夹，设置图片名字，存放图片，关键在于图片取名和浓缩图设置。

                try:
                    pic=urllib.request.urlopen(url)
                    picno=time.strftime('%H%M%S', time.localtime())
                    filenamep=mulu+'/'+picno+validateTitle(item['nick']+'-'+item['title'])
                    filenamepp=filenamep+'.jpeg'
                    sfilename=filenamep+'s.jpeg'
                    filess=open(filenamepp,'wb')
                    filess.write(pic.read())
                    filess.close()
                    img = Image.open(filenamepp)
                    w, h = img.size
                    size=w/6,h/6
                    img.thumbnail(size, Image.ANTIALIAS)
                    img.save(sfilename,'jpeg')
                    itemlist.append(sfilename)
                    print('抓到图片：'+sfilename)
                except Exception as e:
                    if hasattr(e, 'code'):
                        print('页面不存在或时间太长.')
                        print('Error code:', e.code)
                    elif hasattr(e, 'reason'):
                            print("无法到达主机.")
                            print('Reason:  ', e.reason)
                    else:
                        print(e)
                    itemlist.append('')

图片命名：

<img  src="/picture/taobao/taobao21.png"/>

                    picno=time.strftime('%H%M%S', time.localtime())
                    filenamep=mulu+'/'+picno+validateTitle(item['nick']+'-'+item['title'])
                    filenamepp=filenamep+'.jpeg'
                    sfilename=filenamep+'s.jpeg'
                    
图片路径大概是 抓取日期年月日/抓取日期年月日时分秒关键字/时分秒店铺名-商品标题，有转义哦！

如2060105/201601051545婚纱礼服/153648wangjingli327-法国蕾丝钉珠深v领公主新娘甜美修身齐地婚纱礼服2015冬季新款.jpeg

而浓缩图为2060105/201601051545婚纱礼服/153648wangjingli327-法国蕾丝钉珠深v领公主新娘甜美修身齐地婚纱礼服2015冬季新款s.jpeg

浓缩图是为了插入EXCEL而设立，使用了一个库。

大小对比：

<img  src="/picture/taobao/taobao22.png"/>

                    pic=urllib.request.urlopen(url)
                    filess=open(filenamepp,'wb')
                    filess.write(pic.read())
                    filess.close()
                    img = Image.open(filenamepp)
                    w, h = img.size
                    size=w/6,h/6
                    img.thumbnail(size, Image.ANTIALIAS)
                    img.save(sfilename,'jpeg')
                    itemlist.append(sfilename)
                    print('抓到图片：'+sfilename)    

先pic=urllib.request.urlopen(url)打开url获得二进制数据，然后filess=open(filenamepp,'wb')打开一个新文件，将数据读出来再写进去filess.write(pic.read())，这样就保存好了一张图片。

                    img = Image.open(filenamepp)
                    w, h = img.size
                    size=w/6,h/6
                    img.thumbnail(size, Image.ANTIALIAS)
                    img.save(sfilename,'jpeg')
                    itemlist.append(sfilename)
 
然后Image打开这张保存的图片，获得高度宽度w，h，然后以六分之一开始剪切，再保存！

大家来看程序最后几行代码：

    if len(total) > 1:
        writeexcel(roota +'/'+namess+keyword+ '淘宝手机商品.xlsx', total)
    else:
        print('什么都抓不到')
        
total有数据，那么写入Excel，Excel命名自己分析哈，结果如下：

<img  src="/picture/taobao/taobao23.png"/>
<img  src="/picture/taobao/taobao24.png"/>

之后这样要怎样，我想让任何人都可以直接运行，不要安装python环境啦，好啦，看好。

安装cx_Freeze，自己安装!

源代码同级目录新建setup.py

写入：

    import sys
    from cx_Freeze import setup, Executable
    
    base = None
    
    executables = [
        Executable('mtaobao.py', base=base)
    ]
    
    setup (
    name = "mtaobao",
    version = "1.0",
    description = "sangjin",
    executables=executables
    )
    
然后cmd转到该文件夹下：

<img  src="/picture/taobao/taobao25.png"/>
 
生成的文件

<img  src="/picture/taobao/taobao26.png"/>

把exe.win32-3.4移到根目录，任意改名，以下改为exe

<img  src="/picture/taobao/taobao27.png"/>

可执行exe文件夹下的后缀为exe的可执行文件，但是我们还是建一个批处理脚本run.bat吧,

run.bat里面写入

    cd exe
    mtaobao.exe
    
然后直接运行run.bat就可以执行我们的工具了！！

好了，我们的Python3中级玩家：淘宝天猫商品搜索爬虫自动化工具就结束了，好累！

Python3中级玩家：[Python3中级玩家：淘宝天猫商品搜索爬虫自动化工具（第一篇](https://www.lenggirl.com/spider/taobao1.html)

Python3中级玩家：[Python3中级玩家：淘宝天猫商品搜索爬虫自动化工具（第二篇](https://www.lenggirl.com/spider/taobao2.html)

等不及就上github:

git clone [https://github.com/hunterhug/taobaoscrapy.git](https://github.com/hunterhug/taobaoscrapy.git)
 
欢迎收看新的爬虫文章！！！还有很多。。。
 
