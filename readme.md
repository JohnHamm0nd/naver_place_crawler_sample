#naver_place_crawler_sample

### 트레블러리　크롤러　샘플
트레블러리에　사용한　크롤러　중　일부를　사용하여　샘플　크롤러를　만들었다．
Selenium을　사용하지　않아서　속도가　빠르다．
#### 라이브러리
```
import urllib
from bs4 import BeautifulSoup
import pandas as pd
import json
import csv
```
#### 필드체크
필드가　비어있는지　확인.　비어있으면　null을　채워야　필드가　당겨지는　문제가　발생하지　않음．
```
def key_check(dic, key):
    if f'{key}' in dic:
        return dic[f'{key}']
```
#### 크롤러
```
def np_scrap(menu, gu, pages):    
    out = False
    for page in range(1, pages, 5):
        if out:
            break
        else:
            url = urllib.request.Request('https://store.naver.com/restaurants/list?filterId=r09&menu={}&page={}&query={}%20{}'.format(menu, page, urllib.parse.quote_plus(gu), urllib.parse.quote_plus('맛집'))) #메뉴번호와　구를　받아서 url요청
            url.add_header('user-agent', 'Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/69.0.3497.100 Safari/537.36')　#url header설정
            html = urllib.request.urlopen(url)
            bsObj = BeautifulSoup(html.read(), 'lxml', from_encoding='utf-8') #parse
            r_list = bsObj.findAll('script')[2].text[19:] #필요한　데이터　찾기
            r_list = json.loads(r_list) #필요한　데이터　찾기
            r_list = r_list["businesses"][list(r_list["businesses"].keys())[2]]['items'] #필요한　데이터　찾기
            #비어있는　리스트에　값　append
            for i in r_list:
                if i is not None:
                    if key_check(i, 'commonAddr'):
                        if f'{gu}' in i['commonAddr']:
                            idd.append(key_check(i, 'id'))
                            name.append(key_check(i, 'name'))
                            businessCategory.append(key_check(i, 'businessCategory'))
                            category.append(key_check(i, 'category'))
                            desc.append(key_check(i, 'desc'))
                            totalReviewCount.append(key_check(i, 'totalReviewCount'))
                            phone.append(key_check(i, 'phone'))
                            microReview.append(key_check(i, "microReview"))
                            tags.append(key_check(i, 'tags'))
                            options.append(key_check(i, 'options'))
                            roadAddr.append(key_check(i, 'roadAddr'))
                            commonAddr.append(key_check(i, 'commonAddr'))
                            addr.append(key_check(i, 'addr'))
                            x.append(key_check(i, 'x'))
                            y.append(key_check(i, 'y'))
                            imageSrc.append(key_check(i, 'imageSrc'))
                        else:
                            out = True
                            print("종료 페이지:", page)
                            break
                    else:
                        non_addr.append(i)
```
#### 크롤링　샘플
```
idd = []
name = []
businessCategory = []
category = []
desc = []
totalReviewCount = []
phone = []
microReview = []
tags = []
options = []
roadAddr = []
commonAddr = []
addr = []
x = []
y = []
non_addr = []
imageSrc = []

np_scrap(1, '강남구', 10)

# 리스트들로　데이터프레임　만들기
col = ["menu", "id", "name", "businessCategory", "category", "desc", "microReview", "tags", "options", "totalReviewCount", "roadAddr",
       "commonAddr", "addr", "phone",  "x", "y", "imageSrc"]
data = pd.DataFrame(data={"menu": 1, "id": idd, "name": name, "businessCategory": businessCategory, "category": category, "desc": desc,
                      "microReview": microReview, "tags": tags, "options": options, "totalReviewCount": totalReviewCount,
                      "roadAddr": roadAddr,"commonAddr": commonAddr, "addr": addr, "phone": phone,  "x": x, "y": y,
                      "imageSrc": imageSrc }, columns=col)
```
#### 중복데이터　제거
```
data = data.drop_duplicates(subset='id')
```
#### csv파일로　저장
```
data.to_csv('naver_place_sample.csv', encoding='utf-8')
```
