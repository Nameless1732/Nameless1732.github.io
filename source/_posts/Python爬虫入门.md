title: Python爬虫入门
author: Jie
categories:
  - 数据采集
date: 2022-11-11 21:31:54
tags:
---
一些Python爬虫入门的小案例

练习使用通用爬虫框架获取静态网页的文本内容。使用Requests、urllib库，获取 http://www.ip138.com/ip 归属地查询结果，并打印输出页面的HTML文本内容。
<!-- more -->
### 完整代码
```python
import requests

headers = {
    'User-Agent': 'Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/58.0.3029.110 Safari/537.36'}
# 获取ip归属地查询结果，并打印输出页面的HTML文本内容
ip = requests.get("http://2022.ip138.com", headers=headers).text
print(ip)
```

![页面的HTML文本内容](/images/pasted-0.png)

练习使用通用爬虫框架获取多个静态网页的文本内容。使用Requests库，获取豆瓣电影TOP250 https://movie.douban.com/top250?start=0&filter= 前10页查询结果，打印输出页面的HTML文本内容。同时，将第一页内容保存到txt文件中。

### 完整代码
```python
from bs4 import BeautifulSoup
from lxml import html
import requests
import os, csv


def write_dictionary_to_csv(dict, filename):
    file_exists = os.path.isfile(filename)
    with open(filename, 'a', encoding='utf-8') as f:
        w = csv.DictWriter(f, dict.keys(), delimiter=',', quotechar='"', lineterminator='\n', quoting=csv.QUOTE_ALL,
                           skipinitialspace=True)
        if not file_exists:
            w.writeheader()
        w.writerow(dict)
    print('当前行写入csv成功！')


rank = 1


def write_one_page(url):
    headers = {
        'User-Agent': 'Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/58.0.3029.110 Safari/537.36'}

    html = requests.get(url, headers=headers).text
    # lxml：html解析库（把HTML代码转化成Python对象）
    soup = BeautifulSoup(html, 'lxml')
    global rank
    for k in soup.find('div', class_='article').find_all('div', class_='info'):
        name = k.find('div', class_='hd').find_all('span')  # 电影名字
        score = k.find('div', class_='star').find_all('span')  # 分数
        # 抓取年份、国家
        actor_infos_html = k.find(class_='bd')
        actor_infos = actor_infos_html.find('p').get_text().strip().split('\n')
        actor_infos1 = actor_infos[0].split('\xa0\xa0\xa0')
        director = actor_infos1[0][3:]
        role = actor_infos[1]
        year_area = actor_infos[1].lstrip().split('\xa0/\xa0')
        year = year_area[0]
        country = year_area[1]
        type = year_area[2]
        print(rank, name[0].string, score[1].string, year, country, type)
        data = {
            'rank': rank,
            'name': name[0].string,
            'score': score[1].string,
            'year': year,
            'country': country,
            'type': type
        }
        # print(data)
        write_dictionary_to_csv(data, 'top250.csv')


if __name__ == '__main__':
    headers = {
        'User-Agent': 'Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/58.0.3029.110 Safari/537.36'}

    # 爬取第一页HTML文本内容，并保存到txt文件中
    html = requests.get("https://movie.douban.com/top250?start=0&filter=", headers=headers).text
    print(html)
    with open('test.txt', 'w', encoding='utf-8') as f:
        f.write(html)

    # 前十页查询结果
    for i in range(10):
        a = i * 25
        url = "https://movie.douban.com/top250?start=" + str(a) + "&filter="
        write_one_page(url)
```
![](/images/pasted-1.png)
![](/images/pasted-9.png)
![csv文件](/images/pasted-10.png)

爬取豆瓣小说-随笔分类前5页内容，解析网页，将其中每一项的标题、作者、价格、评分、评价人数、简介以csv格式输出至文件中
第一页网址：https://book.douban.com/tag/%E9%9A%8F%E7%AC%94

将1中获得的第一页的图书标题、作者、价格、评分、评价人数依次存入数据库dbread的book_list表中
![](/images/pasted-12.png)

### 完整代码
```python
import csv
import pymysql
import requests
from bs4 import BeautifulSoup

connect = pymysql.Connect(
    host='localhost',
    port=3306,
    user='root',
    passwd='000000',
    db='dbread',
    charset='utf8mb4'
)
# 写入表头数据
file = open('豆瓣小说.csv', mode='a', encoding='utf-8', newline='')
csv_write = csv.DictWriter(file, fieldnames=['标题', '作者', '价格', '评分', '评价人数', '简介'])
csv_write.writeheader()
for i in range(5):
    a = i * 20
    url = 'https://book.douban.com/tag/%E9%9A%8F%E7%AC%94?start=" + str(a) + "&type=T'
    headers = {
        'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/106.0.0.0 Safari/537.36 Edg/106.0.1370.52',
    }
    html = requests.get(url=url, headers=headers).text
    soup = BeautifulSoup(html, "html.parser")

    # 查找所需要的数据
    divs = soup.find_all("div", class_="info")
    for div in divs:
        title = div.find("h2").get_text().strip("\n /").replace(" ", "").replace("\n", "")
        author = div.find("div", class_="pub").get_text().strip("\n /")
        price_ = div.find("span", class_="buy-info")
        if price_ is not None:
            price = price_.get_text().strip("\n /")
        else:
            price = ""
        score_ = div.find("span", class_="rating_nums")
        if score_ is not None:
            score = score_.get_text().strip("\n /")
        else:
            score = ""
        pl = div.find("span", class_="pl").get_text().strip("\n /").lstrip("(").rstrip(")")
        info = div.find("p").get_text().strip("\n /").replace("\n", "")
        print(title, author, price, score, pl, info)
        # 写入csv文件
        data_dict = {'标题': title, '作者': author, '价格': price, '评分': score, '评价人数': pl, '简介': info}
        csv_write.writerow(data_dict)
        # 写入数据库
        cursor = connect.cursor()
        sql = "INSERT INTO book_list(title, author, price, score, pl, info) VALUES('%s','%s','%s','%s','%s','%s')"
        data = (title, author, price, score, pl, info)
        cursor.execute(sql % data)
        connect.commit()
        print('成功插入数据')
connect.close()
```
运行截图
![运行截图](/images/pasted-11.png)

csv文件
![csv文件](/images/pasted-13.png)

保存到mysql数据库中的数据
![数据库](/images/pasted-14.png)
