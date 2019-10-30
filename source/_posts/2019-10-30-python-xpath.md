---
title: python3 使用xpath
date: 2019-10-30 09:19:29
category: learn
toc: true
tags:
    - python3
    - xpath
---

python爬网页数据可以用xpath 进行标签定位

<!-- more -->

```python
import requests
from lxml import html

url = 'https://book.douban.com/' #需要爬数据的网址
page = requests.Session().get(url)
tree = html.fromstring(page.text)

```

> 使用 lxml 中的 xpath 高效提取文本与标签属性值

先安装 lxml

```bash
pip install lxml
```

## 定位html元素

```python
divs1 = tree.xpath('//div') #查找所有div标签

divs2 = tree.xpath('//div[@id="header"]') # 查找id为header的标签

divs3 = tree.xpath('//div[@class="foot"]') # 获取class为foot的div标签

divs4 = tree.xpath('//div[@class="foot"]//a') # 获取class为foot下的a标签
```

> 属性名称后面的一定要加引号 `@class="foot"`

### 定位多属性标签

原贴链接：[[How can I find an element by CSS class with XPath?](https://stackoverflow.com/questions/1604471/how-can-i-find-an-element-by-css-class-with-xpath)]( http://stackoverflow.com/questions/1604471/how-can-i-find-an-element-by-css-class-with-xpath )

```python
//*[contains(@class, 'Test')]
```

 但是这个表达式会把类似 class="Testvalue" 或者 class="newTest"也匹配出来。 可以使用如下版本

```python
//*[contains(concat(' ', @class, ' '), ' Test ')]
```

 如果你希望匹配的结果尽量精确，你还可以使用 normalize-space 功能来清除class名称周围的空格 

```python
//*[contains(concat(' ', normalize-space(@class), ' '), ' Test ')]
```



需要定位具有 *menu* 属性的`ul`标签

```html
<div class="header">
	<ul class="banner menu">
		<li></li>
		<li></li>
		<li></li>
	</ul>
</div>
```



```python
div5 = tree.xpath('//div[@class="header"]//ul[contains(@class, "menu")]')
```

> 建议把class换成更好识别的标识执行效率会更高 



## 取元素属性或文本

获取 *div* 标签下的 *span* 标签中的文本信息

```python
content = tree.xpath('//div//span/text()')
```



获取标签中的属性信息

```python
href = tree.xpath('//div//a/@href') #获取a标签的href图片地址

src = tree.xpath('//div//a/@alt') #获取a标签的alt值
```


如果获取的是标签，则返回的对象可以继续使用xpath方法

```python
result = tree.xpath('//ul[contains(@class, "pic-list2")]//li//a')  # 获取需要的数据

for item in result:
    src = item.xpath('@src')[0] # 获取的是list
    alt = item.xpath('@alt')[0] # 获取的是list
    print(src, alt)

```



---

## 爬取豆瓣读书的图片

```python
# 豆瓣读书
def captureDouBan():
    url = 'https://book.douban.com/' #需要爬数据的网址
    page = requests.Session().get(url)
    tree = html.fromstring(page.text)
    result = tree.xpath('//div[@class="cover"]//a//img') #获取需要的数据
    for item in result:
        src = item.xpath('@src')[0]
        alt = item.xpath('@alt')[0]
        print(src, alt)
        request.urlretrieve(src, "E://xiaotang//python3//img//"+alt+".jpg")

```



