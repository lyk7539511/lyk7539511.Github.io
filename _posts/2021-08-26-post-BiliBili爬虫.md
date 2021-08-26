---
title: "使用 Python 爬取 BiliBili 视频评论及弹幕"
categories:
  - Blog
  - Tech
tags:
  - Python
  - Web Crawlers
---

## 环境
  - OS: Windows 10 21H1
  - Mem: 16G
  - CPU: i7-6820HQ
  - Software: MiniConda Python=3.8 Numpy Pandas requests json time

## 爬取视频评论
### 源码

```python
# -*- coding: utf-8 -*-
"""
Created on Thu Jun 24 17:12:24 2021
@author: Liu_YK d20091100124@cityu.mo
"""

import requests
import json
import time
import numpy as np
import pandas as pd

# --------------------------------------------------
# BV2AV AV2BV
# https://blog.csdn.net/jkddf9h8xd9j646x798t/article/details/105124465
alphabet = 'fZodR9XQDSUm21yCkr6zBqiveYah8bt4xsWpHnJE7jL5VG3guMTKNPAwcF'

def dec(x):
    r = 0
    for i, v in enumerate([11, 10, 3, 8, 4, 6]):
        r += alphabet.find(x[v]) * 58**i
    return (r - 0x2_0840_07c0) ^ 0x0a93_b324

def enc(x):
    x = (x ^ 0x0a93_b324) + 0x2_0840_07c0
    r = list('BV1**4*1*7**')
    for v in [11, 10, 3, 8, 4, 6]:
        x, d = divmod(x, 58)
        r[v] = alphabet[d]
    return ''.join(r)
# ---------------------------------------------------

def getComment(bv):
    
    # https://curl.trillworks.com/
    headers = {
        'authority': 'api.bilibili.com',
        'user-agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.114 Safari/537.36 Edg/91.0.864.54',
        'referer': 'https://www.bilibili.com',
    }
    
    now_time = int(time.time() * 1000)
    
    # set proxy ip
    proxy = 'http:\\\\' +  '42.56.239.217' + ':' + '9999'
    proxies = {'proxy':proxy}
    
    # set 'next' param
    i = 0
    
    data_comment = []
    
    with open(bv + '_comment.txt', 'w', encoding='utf-8') as fp:
    
        while True: 
            
            params = (
                ('callback', 'jQuery33105450292163562576_' + str(now_time)),
                ('jsonp', 'jsonp'),
                ('next', str(i)),
                ('type', '1'),
                ('oid', dec(bv)),
                ('mode', '3'),
                ('plat', '1'),
                ('_', str(int(time.time() * 1000))),
            )
            
            response = requests.get('https://api.bilibili.com/x/v2/reply/main', headers=headers, params=params, proxies = proxies)
        
            # del some str
            rsp_str = response.text.replace('jQuery33105450292163562576_' + str(now_time) + "(", '').strip(')')
            # load json
            data = json.loads(rsp_str)
            
            if not(data["data"]["cursor"]["is_end"]):
                for comment in data["data"]["replies"]:
                    data_comment.append(comment["content"]["message"])
                    fp.write(comment["content"]["message"])
                
                now_time += 1
                print(i)
                i += 1
                time.sleep(0.5)
            
            else:
                print("end")
                break

    return data_comment

bv = input('请输入bv号：')   #BV1bB4y1M72n
comment = getComment(bv)

comment_np = np.array(comment)
comment_pd = pd.DataFrame(comment_np)

comment_pd.to_csv(bv + "_comment" + ".csv", encoding='utf-8-sig')
```

## 爬取视频弹幕
### 源码

```python
# -*- coding: utf-8 -*-
"""
Created on Sat Jun 26 10:18:39 2021
@author: Liu_YK d20091100124@cityu.mo
"""

import requests
from bs4 import BeautifulSoup
import time
import json
import numpy as np
import pandas as pd

# ---------------------------------------------
# BV2AV AV2BV
# https://blog.csdn.net/jkddf9h8xd9j646x798t/article/details/105124465
alphabet = 'fZodR9XQDSUm21yCkr6zBqiveYah8bt4xsWpHnJE7jL5VG3guMTKNPAwcF'

def dec(x):
    
    r = 0
    for i, v in enumerate([11, 10, 3, 8, 4, 6]):
        r += alphabet.find(x[v]) * 58**i
    return (r - 0x2_0840_07c0) ^ 0x0a93_b324

def enc(x):
    
    x = (x ^ 0x0a93_b324) + 0x2_0840_07c0
    r = list('BV1**4*1*7**')
    for v in [11, 10, 3, 8, 4, 6]:
        x, d = divmod(x, 58)
        r[v] = alphabet[d]
    return ''.join(r)

# -----------------------------------------------

# no need!
def getMid(bv):
    
    headers = {
        'authority': 'api.bilibili.com',
        'user-agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.114 Safari/537.36 Edg/91.0.864.54',
        'referer': 'https://www.bilibili.com',
    }
    
    now_time = int(time.time() * 1000)
    params = (
        ('callback', 'jQuery33105450292163562576_' + str(now_time)),
        ('jsonp', 'jsonp'),
        ('next', '0'),
        ('type', '1'),
        ('oid', dec(bv)),#588676861  630756505
        ('mode', '3'),
        ('plat', '1'),
        ('_', str(int(time.time() * 1000))),
    )
    
    proxy = 'http:\\\\' +  '42.56.239.217' + ':' + '9999'
    proxies = {'proxy':proxy}
    response = requests.get('https://api.bilibili.com/x/v2/reply/main', headers=headers, params=params, proxies = proxies)

    rsp_str = response.text.replace('jQuery33105450292163562576_' + str(now_time) + "(", '').strip(')')
    data = json.loads(rsp_str)
    
    mid = data['data']['upper']['mid']
    print("mid:", mid)
    
    return mid

# API随时变动，此方法随时失效
def getCid(bv):
    
    headers = {
        'authority': 'api.bilibili.com',
        'sec-ch-ua': '" Not;A Brand";v="99", "Microsoft Edge";v="91", "Chromium";v="91"',
        'accept': '*/*',
        'dnt': '1',
        'sec-ch-ua-mobile': '?0',
        'user-agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.114 Safari/537.36 Edg/91.0.864.59',
        'origin': 'https://www.bilibili.com',
        'sec-fetch-site': 'same-site',
        'sec-fetch-mode': 'cors',
        'sec-fetch-dest': 'empty',
        'referer': 'https://www.bilibili.com',
        'accept-language': 'zh-CN,zh;q=0.9,en;q=0.8,en-GB;q=0.7,en-US;q=0.6,zh-TW;q=0.5',
    }
    
    params = (
        ('bvid', bv),
        ('jsonp', 'jsonp'),
    )

    response = requests.get('https://api.bilibili.com/x/player/pagelist', headers=headers, params=params)
    
    data = json.loads(response.text)
    
    cid = data["data"][0]["cid"]
    
    return cid


def getDm(bv):
    
    dm = []
    headers = {
        'authority': 'comment.bilibili.com',
        'sec-ch-ua': '" Not;A Brand";v="99", "Microsoft Edge";v="91", "Chromium";v="91"',
        'sec-ch-ua-mobile': '?0',
        'dnt': '1',
        'upgrade-insecure-requests': '1',
        'user-agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.114 Safari/537.36 Edg/91.0.864.59',
        'accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9',
        'sec-fetch-site': 'none',
        'sec-fetch-mode': 'navigate',
        'sec-fetch-user': '?1',
        'sec-fetch-dest': 'document',
        'accept-language': 'zh-CN,zh;q=0.9,en;q=0.8,en-GB;q=0.7,en-US;q=0.6,zh-TW;q=0.5',
    }

    # set proxy
    proxy = 'http:\\\\' +  '58.255.199.192' + ':' + '9999'
    proxies = {'proxy':proxy}
    
    # https://comment.bilibili.com/{5771902}.xml
    # 这里是cid，非oid(av)/mid
    response = requests.get('https://comment.bilibili.com/' + str(getCid(bv)) + '.xml', headers=headers, proxies = proxies)
    
    # !!!!很重要，必须转码
    pagetext = response.content.decode("utf-8")
    
    soup = BeautifulSoup(pagetext, 'lxml')
    
    with open(bv + '_dm.txt', 'w', encoding='utf-8') as fp:
        
        for i in soup.find_all('d'):
            print(i.text)
            fp.write(i.text + '\n')
            dm.append(i.text)
    
    return dm

bv = input('请输入bv号：')
dm = getDm(bv)

dm_np = np.array(dm)
dm_pd = pd.DataFrame(dm_np)

dm_pd.to_csv(bv + "_dm" + ".csv", encoding='utf-8-sig') 
```

## 最终结果全部保存在txt和csv文件中，与源码文件同级