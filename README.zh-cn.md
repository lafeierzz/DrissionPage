# 简介

***

DrissionPage，即driver和session的合体，是个基于python的Web自动化操作集成工具。  
它实现了selenium和requests之间的无缝切换。  
因此可以兼顾selenium的便利性和requests的高效率。  
它用POM模式封装了页面元素常用的方法，很适合自动化操作功能扩展。  
更棒的是，它的使用方式非常简洁和人性化，代码量少，对新手友好。  

**项目地址：**

- https://github.com/g1879/DrissionPage
- https://gitee.com/g1879/DrissionPage

**示例地址：** [使用DrissionPage爬取常见网站](https://gitee.com/g1879/DrissionPage-demos)（完善中）

**联系邮箱：** g1879@qq.com

# 特性

***

- 允许在selenium和requests间无缝切换，共享session。  
- 使用POM模式封装常用方法，便于扩展。
- 两种模式提供统一的操作方法，使用体验一致。    
- 人性化的页面元素操作方法，减轻页面分析工作量和编码量。  
- 对某些常用功能（如点击）作了优化，更符合实际使用需要。  
- 简易的配置方法，摆脱繁琐的浏览器配置。

# 理念

***

## 简洁、易用 、可扩展

- DrissionPage以简洁的代码为第一追求，对冗长的语句做了精简，并完全保留了其功能。
- DrissionPage封装了许多常用功能，使用更便捷。
- DrissionPage的核心是个页面类，可直接派生子类页面，适应各种场景须要。
- 简易的浏览器配置方法，摆脱繁琐的设置。

以下代码实现一模一样的功能，对比两者的代码量：

1. 用显性等待方式查找所有文本包含some text的元素

```python
# selenium:
element = WebDriverWait(driver).until(ec.presence_of_all_elements_located((By.XPATH, '//*[contains(text(), "some text")]')))
# DrissionPage:
element = page.eles('some text')
```

2. 跳转到第一个标签页

```python
# selenium
driver.switch_to.window(driver.window_handles[0])
# DrissionPage
page.to_tab(0)
```

3. 拖拽一个元素

```python
# selenium
ActionChains(driver).drag_and_drop(ele1, ele2).perform()
# DrissionPage
ele1.drag_to(ele2)
```

4. 滚动窗口到底部（保持水平滚动条不变）

```python
# selenium
driver.execute_script("window.scrollTo(document.documentElement.scrollLeft,document.body.scrollHeight);")
# DrissionPage
page.scroll_to('bottom')
```

5. 设置headless模式

```python
# selenium
options = webdriver.ChromeOptions()
options.add_argument("--headless")
# DrissionPage
set_headless()
```



# 背景

***

新手学习爬虫时，面对须要登录的网站，要分析数据包、JS源码，构造复杂的请求，往往还要应付验证码、JS混淆、签名参数等反爬手段，学习门槛较高。获取数据时，有的数据是由JS计算生成的，若只拿到源数据，还须重现计算过程，体验不好，开发效率不高。

使用selenium，可以很大程度上绕过这些坑，但selenium效率不高。因此，这个库要做的，是将selenium和requests合而为一，不同须要时切换相应模式，并提供一种人性化的使用方法，提高开发和运行效率。

除了合并两者，本库还以网页为单位封装了常用功能，简化了selenium的操作和语句，在用于网页自动化操作时，减少考虑细节，专注功能实现，使用更方便。

本人学习过程中踩了很多坑，因此这个库的设计理念是一切从简，尽量提供简单直接的使用方法，对新手更友好。

# 简单演示

***

例：用selenium登录网站，然后切换到requests读取网页。

```python
page = MixPage()  # 创建页面对象，默认driver模式
page.get('https://gitee.com/profile')  # 访问个人中心页面（未登录，重定向到登录页面）

page.ele('@id:user_login').input('your_user_name')  # 使用selenium输入账号密码登录
page.ele('@id:user_password').input('your_password\n')

page.change_mode()  # 切换到session模式
print('登录后title：', page.title, '\n')  # 登录后session模式的输出
```

输出：

```
登录后title： 个人资料 - 码云 Gitee.com
```

例：获取并打印属性
```python
foot = page.ele('@id:footer-left')  # 用id查找元素
first_col = foot.ele('css:>div')  # 使用css selector在元素的下级中查找元素（第一个）
lnk = first_col.ele('text:命令学')  # 使用文本内容查找元素
text = lnk.text  # 获取元素文本
href = lnk.attr('href')  # 获取元素属性值

print(first_col)
print(text, href)
```

输出：

```
<SessionElement div class='column'>
Git 命令学习 https://oschina.gitee.io/learn-git-branching/
```

# 安装

***

```
pip install DrissionPage
```
只支持python3.6及以上版本，driver模式目前只支持chrome。  
若要使用driver模式，须下载chrome和 **对应版本** 的chromedriver。[[chromedriver下载]](https://chromedriver.chromium.org/downloads)  
目前只在Windows环境下作了测试。

# 使用方法

***

## 导入模块

```python
from DrissionPage import *
```



## 初始化

使用selenium前，必须配置chrome.exe和chromedriver.exe的路径，并确保它们版本匹配。  
如果你只使用session模式，可跳过本节。

配置路径有三种方法：
- 将两个路径写入系统变量。
- 使用时手动传入路径。
- 将路径写入本库的ini文件（推荐）。

若你选择第三种方式，请在第一次使用本库前，运行这几行代码，把这两个路径记录到ini文件中。

```python
from DrissionPage.easy_set import set_paths
driver_path = 'D:\\chrome\\chromedriver.exe'  # 你的chromedriver.exe路径，可选
chrome_path = 'D:\\chrome\\chrome.exe'  # 你的chrome.exe路径，可选
set_paths(driver_path, chrome_path)
```

该方法还会检查chrome和chromedriver版本是否匹配，显示：

```
版本匹配，可正常使用。

或

出现异常：
Message: session not created: Chrome version must be between 70 and 73
  (Driver info: chromedriver=73.0.3683.68 (47787ec04b6e38e22703e856e101e840b65afe72),platform=Windows NT 10.0.19631 x86_64)
chromedriver下载网址：https://chromedriver.chromium.org/downloads
```

检查通过后，即可正常使用driver模式。

除了上述两个路径，该方法还可以设置以下路径：

```python
debugger_address  # 调试浏览器地址，如：127.0.0.1:9222
download_path  # 下载文件路径
global_tmp_path  # 临时文件夹路径
user_data_path # 用户数据路径
cache_path # 缓存路径
```

Tips：

- 不同项目可能须要不同版本的chrome和chromedriver，你还可保存多个ini文件，按须使用。
- 推荐使用绿色版chrome，并手动设置路径，避免浏览器升级造成与chromedriver版本不匹配。
- 调试项目时推荐设置debugger_address，使用手动打开的浏览器调试，省时省力。



## 创建驱动器对象Drission

Drission对象用于管理driver和session对象。在多个页面协同工作时，Drission对象用于传递驱动器，使多个页面类可控制同一个浏览器或Session对象。  
可直接读取ini文件配置信息创建，也可以在初始化时传入配置信息。

```python
# 由默认ini文件创建
drission = Drission()  

# 由其它ini文件创建
drission = Drission(ini_path = 'D:\\settings.ini')  
```

若要手动传入配置：

```python
# 用传入的配置信息创建（忽略ini文件）
from DrissionPage.config import DriverOptions

driver_options = DriverOptions()  # 创建driver配置对象
driver_options.binary_location = 'D:\\chrome\\chrome.exe'  # chrome.exe路径
session_options = {'headers': {'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_6)'}}
driver_path = 'D:\\chrome\\chromedriver.exe'  # driver_path路径

drission = Drission(driver_options, session_options, driver_path)  # 传入配置
```



## 使用页面对象MixPage

MixPage页面对象封装了常用的网页操作，并实现driver和session模式之间的切换。  
MixPage须接收一个Drission对象并使用其中的driver或session，如没有传入，MixPage会自己创建一个Drission（使用默认ini文件的配置）。

Tips: 多页面对象协同工作时，记得手动创建Drission对象并传递给页面对象使用。否则页面对象会各自创建自己的Drission对象，使信息无法传递。

```python
# 创建MixPage对象的方法
page = MixPage()  # 自动创建Drission对象，driver模式，建议只在单页面对象情况下使用
page = MixPage('s')  # 以session模式快速创建，自动创建Drission对象

page = MixPage(drission)  # 以传入Drission对象创建
page = MixPage(drission, mode='s', timeout=5)  # session模式，等待时间5秒（默认10秒）

# 访问URL
page.get(url, **kwargs)
page.post(url, data, **kwargs)  # 只有session模式才有post方法

# 切换模式
page.change_mode()

# 操作页面
print(page.html)  # 页面源代码
page.run_script(js)  # 运行js语句
page.close_other_tabs(num)  # 关闭其它标签页
page.to_iframe(iframe)  # 切入iframe
page.screenshot(path)  # 页面截图
page.scrool_to_see(element)  # 滚动直到某元素可见
# 详见APIs...
```

Tips：调用只属于driver模式的方法，会自动切换到driver模式。



## 查找元素

ele()返回第一个符合条件的元素，eles()返回所有符合条件的元素列表。  
你可在页面对象或元素对象下使用这两个函数，以查找下级元素。  

page.eles()和element.eles()查找返回符合条件的所有元素列表。  

注：元素查找超时默认为10秒，你也可以按需要设置。

```python
# 根据属性查找
page.ele('@id:ele_id', timeout = 2)  # 查找id为ele_id的元素，设置等待时间2秒  
page.eles('@class')  # 查找所有拥有class属性的元素
page.eles('@class:class_name')  # 查找所有class含有ele_class的元素 
page.eles('@class=class_name')  # 查找所有class等于ele_class的元素 

# 根据tag name查找
page.ele('tag:li')  # 查找第一个li元素  
page.eles('tag:li')  # 查找所有li元素  

# 根据tag name及属性查找
page.ele('tag:div@class=div_class')  # 查找class为div_class的div元素
page.ele('tag:div@class:ele_class') # 查找class含有ele_class的div元素
page.ele('tag:div@class=ele_class') # 查找class等于ele_class的div元素
page.ele('tag:div@text():search_text') # 查找文本含有search_text的div元素
page.ele('tag:div@text()=search_text') # 查找文本等于search_text的div元素

# 根据文本内容查找
page.ele('search text')  # 查找包含传入文本的元素  
page.eles('text:search text')  # 如文本以@、tag:、css:、xpath:、text:开头，则在前面加上text:避免冲突  
page.eles('text=search text')  # 文本等于search_text的元素

# 根据xpath或css selector查找
page.eles('xpath://div[@class="ele_class"]')  
page.eles('css:div.ele_class')  

# 根据loc查找
loc1 = By.ID, 'ele_id'
loc2 = By.XPATH, '//div[@class="ele_class"]'
page.ele(loc1)
page.ele(loc2)

# 查找下级元素
element = page.ele('@id:ele_id')
element.ele('@class:class_name')  # 在element下级查找第一个class为ele_class的元素
element.eles('tag:li')  # 在ele_id下级查找所有li元素

# 根据位置查找
element.parent  # 父元素  
element.next  # 下一个兄弟元素  
element.prev  # 上一个兄弟元素  

# 串连查找
page.ele('@id:ele_id').ele('tag:div').next.ele('some text').eles('tag:a')
```



## 元素操作

```python
# 获取元素信息
element = page.ele('@id:ele_id')
element.html  # 返回元素内html
element.text  # 返回元素内去除html标签后的text值
element.tag  # 返回元素tag name
element.attrs  # 返回元素所有属性的字典
element.attr('class')  # 返回元素的class属性
element.is_valid  # driver模式独有，用于判断元素是否还可用

# 操作元素
element.click()  # 点击元素
element.input(text)  # 输入文字
element.run_script(js)  # 运行js
element.submit()  # 提交表单
element.clear()  # 清空元素
element.is_selected()  # 是否被选中
element.is_enabled()  # 是否可用
element.is_displayed()  # 是否可见
element.is_valid()  # 是否有效，用于判断页面跳转导致元素失效的情况
element.select(text)  # 选中下拉列表选项
element.set_attr(attr,value)  # 设置元素属性
element.size  # 元素大小
element.location  # 元素位置
```



## Chrome快捷设置

chrome的配置很繁琐，为简化使用，本库提供了常用配置的设置方法。

### DriverOptions对象

DriverOptions对象继承自selenium.webdriver.chrome.options的Options对象，在其基础上增加了以下方法：

```python
remove_argument(value)  # 删除某argument值
remove_experimental_option(key)  # 删除某experimental_option设置
remove_all_extensions()  # 删除全部插件
save()  # 保存配置到默认ini文件
save('D:\\settings.ini')  # 保存到其它路径
set_argument(arg, value)  # 设置argument属性
set_headless(on_off)  # 设置是否使用无界面模式
set_no_imgs(on_off)  # 设置是否加载图片
set_no_js(on_off)  # 设置是否禁用js
set_mute(on_off)  # 设置是否静音
set_user_agent(user_agent)  # 设置user agent
set_proxy(proxy)  # 设置代理地址
set_paths(driver_path, chrome_path, debugger_address, download_path, user_data_path, cache_path)  # 设置浏览器相关的路径
```

### 使用方法

```python
do = DriverOptions(read_file=False)  # 创建chrome配置对象，不从ini文件读取
do.set_headless(False)  # 显示浏览器界面
do.set_no_imgs(True)  # 不加载图片
do.set_paths(driver_path='D:\\chromedriver.exe', chrome_path='D:\\chrome.exe')  # 设置路径
do.set_headless(False).set_no_imgs(True)  # 支持链式操作

drission = Drission(driver_options=do)  # 用配置对象创建Drission对象
page = MixPage(drission)  # 用Drission对象创建MixPage对象

do.save()  # 保存配置到默认ini文件
```



## 保存配置

因chrome和headers配置繁多，故设置一个ini文件专门用于保存常用配置，你可使用OptionsManager对象获取和保存配置，用DriverOptions对象修改chrome配置。你也可以保存多个ini文件，按不同项目须要调用。

Tips：建议把常用配置文件保存到别的路径，以防本库升级时配置被重置。

### ini文件内容

ini文件默认拥有三部分配置：paths、chrome_options、session_options，初始内容如下。

```ini
[paths]
; chromedriver.exe路径
chromedriver_path =
; 临时文件夹路径，用于保存截图、文件下载等
global_tmp_path =

[chrome_options]
; 已打开的浏览器地址和端口，如127.0.0.1:9222
debugger_address =
; chrome.exe路径
binary_location =
; 配置信息
arguments = [
            ; 隐藏浏览器窗口
            '--headless',
            ; 静音
            '--mute-audio',
            ; 不使用沙盒
            '--no-sandbox',
            ; 谷歌文档提到需要加上这个属性来规避bug
            '--disable-gpu',
            ; 忽略警告
            'ignore-certificate-errors',
            ; 不显示信息栏
            '--disable-infobars'
            ]
; 插件
extensions = []
; 实验性配置
experimental_options = {
                       'prefs': {
                       ; 下载不弹出窗口
                       'profile.default_content_settings.popups': 0,
                       ; 无弹窗
                       'profile.default_content_setting_values': {'notifications': 2},
                       ; 禁用PDF插件
                       'plugins.plugins_list': [{"enabled": False, "name": "Chrome PDF Viewer"}]
                       },
                       ; 设置为开发者模式，防反爬虫
                       'excludeSwitches': ["enable-automation"],
                       'useAutomationExtension': False
                       }

[session_options]
headers = {
          "User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_6) AppleWebKit/603.3.8 (KHTML, like Gecko) Version/10.1.2 Safari/603.3.8",
          "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8",
          "Connection": "keep-alive",
          "Accept-Charset": "utf-8;q=0.7,*;q=0.7"
          }
```

### OptionsManager对象

OptionsManager对象用于读取、设置和保存配置。

```python
get_value(section, item) -> str  # 获取某个配置的值
get_option(section) -> dict  # 以字典格式返回配置全部属性
set_item(section, item, value)  # 设置配置属性
save()  # 保存配置到默认ini文件
save('D:\\settings.ini')  # 保存到其它路径
```

### 使用示例

```python
from DrissionPage.configs import *

options_manager = OptionsManager()  # 从默认ini文件创建OptionsManager对象
options_manager = OptionsManager('D:\\settings.ini')  # 从其它ini文件创建OptionsManager对象
driver_path = options_manager.get_value('paths', 'chromedriver_path')  # 读取路径信息
options_manager.save()  # 保存到默认ini文件
options_manager.save('D:\\settings.ini')  # 保存到其它路径

drission = Drission(ini_path = 'D:\\settings.ini')  # 使用其它ini文件创建对象
```

**注意**：保存时若不传入路径，会保存到模块目录下的ini文件，即使读取的不是默认ini文件也一样。



## easy_set方法

​	调用easy_set方法会修改默认ini文件相关内容。

```python
set_headless(True)  # 开启headless模式
set_no_imgs(True)  # 开启无图模式
set_no_js(True)  # 禁用JS
set_mute(True)  # 开启静音模式
set_user_agent('Mozilla/5.0 (Macintosh; Int......')  # 设置user agent
set_proxy('127.0.0.1:8888')  # 设置代理
set_paths(paths)  # 见 [初始化] 一节
set_argument(arg, value)  # 设置属性，若属性无值（如'zh_CN.UTF-8'），value为bool表示开关；否则value为str，当value为''或False，删除该属性项
```

# POM模式

***

MixPage封装了常用的页面操作，可方便地用于扩展。  

例：扩展一个列表页面读取类

```python
import re
from time import sleep
from DrissionPage import *

class ListPage(MixPage):
    """本类封装读取列表页面的方法，根据必须的4个元素，可读取同构的列表页面
    （中文变量真香）"""
    def __init__(self, drission: Drission, url: str = None, **xpaths):
        super().__init__(drission)
        self._url = url
        self.xpath_栏目名 = xpaths['栏目名']  # [xpath字符串, 正则表达式]
        self.xpath_下一页 = xpaths['下一页']
        self.xpath_行s = xpaths['行']
        self.xpath_页数 = xpaths['页数']  # [xpath字符串, 正则表达式]
        self.总页数 = self.get_总页数()
        if url:
            self.get(url)

    def get_栏目名称(self) -> str:
        if self.xpath_栏目名[1]:
            s = self.ele(f'xpath:{self.xpath_栏目名[0]}').text
            r = re.search(self.xpath_栏目名[1], s)
            return r.group(1)
        else:
            return self.ele(f'xpath:{self.xpath_栏目名[0]}').text

    def get_总页数(self) -> int:
        if self.xpath_页数[1]:
            s = self.ele(f'xpath:{self.xpath_页数[0]}').text
            r = re.search(self.xpath_页数[1], s)
            return int(r.group(1))
        else:
            return int(self.ele(f'xpath:{self.xpath_页数[0]}').text)

    def click_下一页(self, wait: float = None):
        self.ele(f'xpath:{self.xpath_下一页}').click()
        if wait:
            sleep(wait)

    def get_当前页列表(self, 待爬内容: list) -> list:
        """
        待爬内容格式：[[xpath1,参数1],[xpath2,参数2]...]
        返回列表格式：[[参数1,参数2...],[参数1,参数2...]...]
        """
        结果列表 = []
        行s = self.eles(f'xpath:{self.xpath_行s}')
        for 行 in 行s:
            行结果 = []
            for j in 待爬内容:
                行结果.append(行.ele(f'xpath:{j[0]}').attr(j[1]))
            结果列表.append(行结果)
            print(行结果)
        return 结果列表

    def get_列表(self, 待爬内容: list,  wait: float = None) -> list:
        列表 = self.get_当前页列表(待爬内容)
        for _ in range(self.总页数 - 1):
            self.click_下一页(wait)
            列表.extend(self.get_当前页列表(待爬内容))
        return 列表
```

# 其它

***

## DriverPage和SessionPage

如果无须切换模式，可根据需要只使用DriverPage或SessionPage，用法和MixPage一致。  

```python
from DrissionPage.session_page import SessionPage
from DrissionPage.drission import Drission

session = Drission().session
page = SessionPage(session)  # 传入Session对象
page.get('http://www.baidu.com')
print(page.ele('@id:su').text)  # 输出：百度一下

driver = Drission().driver
page = DriverPage(driver)  # 传入Driver对象
page.get('http://www.baidu.com')
print(page.ele('@id:su').text)  # 输出：百度一下
```

# APIs

***

## Drission类

class **Drission**(driver_options: Union[dict, Options] = None, session_options: dict = None, ini_path: str = None, proxy: dict = None)

用于管理driver和session对象。

​	参数说明：

- driver_options     - chrome配置参数，可接收Options对象或字典
- session_options  - session配置参数，接收字典
- ini_path               - ini文件路径，默认为DrissionPage文件夹下的ini文件

### session

​	返回HTMLSession对象，调用时自动创建。

### driver

​	返回WebDriver对象，调用时自动创建，按传入配置或ini文件配置初始化。

### driver_options

​	以字典格式返回或设置driver配置。

### session_options

​	以字典格式返回或设置session配置。

### proxy

​	以字典格式返回代理配置。

### cookies_to_session()

​	cookies_to_session(copy_user_agent: bool = False, driver: WebDriver = None, session: Session = None) -> None

​	把cookies从driver复制到session。默认复制self.driver到self.session，也可以接收driver和session进行操作。

​	参数说明：

- copy_user_agent  - 是否复制user_agent到session
- driver     - WebDriver对象，复制cookies
- session  - Session对象，接收cookies

### cookies_to_driver()

​	cookies_to_driver(url: str, driver: WebDriver = None, session: Session = None) -> None

​	把cookies从session复制到driver。默认复制self.session到self.driver，也可以接收driver和session进行操作。须要指定url或域名。

​	参数说明：

- url          - cookies的域
- driver     - WebDriver对象，接收cookies
- session  - Session对象，复制cookies

### user_agent_to_session()

​	user_agent_to_session(driver: WebDriver = None, session: Session = None) -> None

​	把user agent从driver复制到session。默认复制self.driver到self.session，也可以接收driver和session进行操作。

​	参数说明：

- driver     - WebDriver对象，复制user agent
- session  - Session对象，接收user agent

### close_driver()

​	close_driver() -> None

​	关闭浏览器，driver置为None。

### close_session()

​	close_session() -> None

​	关闭session并置为None。

### close()

​	close() -> None

​	关闭driver和session。



## MixPage类

class **MixPage**(drission: Union[Drission, str] = None, mode:str = 'd', timeout: float = 10)

MixPage封装了页面操作的常用功能，可在driver和session模式间无缝切换。切换的时候会自动同步cookies。  
获取信息功能为两种模式共有，操作页面元素功能只有d模式有。调用某种模式独有的功能，会自动切换到该模式。  
它继承自DriverPage和SessionPage类，这些功能由这两个类实现，MixPage作为调度的角色存在。

参数说明：

- drission  - Drission对象，如没传入则创建一个。传入's'或'd'时快速配置相应模式
- mode      - 模式，可选'd'或's'，默认为'd'
- timeout    - 超时时间，driver模式查找元素时间及session模式连接时间

### url  

​	返回当前访问的url。

### mode

​	返回当前模式（'s'或'd'）。

### drission

​	返回当前使用的Dirssion对象。

### driver

​	返回driver对象，如没有则创建，调用时会切换到driver模式。

### session

​	返回session对象，如没有则创建。

### response

​	返回Response对象，调用时会切换到session模式。

### cookies

​	返回cookies，从当前模式获取。

### html

​	返回页面html文本。

### title

​	返回页面title文本。

### change_mode()

​	change_mode(mode: str = None, go: bool = True) -> None

​	切换模式，可指定目标模式，若目标模式与当前模式一致，则直接返回。

​	参数说明：

- mode  - 指定目标模式，'d'或's'。
- go       - 切换模式后是否跳转到当前url

### get()

​	get(url: str, go_anyway=False, **kwargs) -> Union[bool, None]

​	跳转到一个url，跳转前先同步cookies，跳转后返回目标url是否可用。

​	参数说明：

- url                - 目标url
- go_anyway  - 是否强制跳转。若目标url和当前url一致，默认不跳转。
- kwargs         - 用于session模式时访问参数。

### ele()

​	ele(loc_or_ele: Union[tuple, str, DriverElement, SessionElement], mode: str = None, timeout: float = None, show_errmsg: bool = False) -> Union[DriverElement, SessionElement]

​	返回页面中符合条件的元素，默认返回第一个。  
​	如查询参数是字符串，可选'@属性名:'、'tag:'、'text:'、'css:'、'xpath:'方式。无控制方式时默认用text方式查找。  
​	如是loc，直接按照内容查询。

​	参数说明：

- loc_or_str        - 元素的定位信息，可以是元素对象，loc元组，或查询字符串
- mode               - 'single' 或 'all‘，对应查找一个或全部
- timeout            - 查找元素超时时间，driver模式下有效
- show_errmsg  - 出现异常时是否抛出及显示

​	示例：

- 接收到元素对象时：返回元素对象对象

- 用loc元组查找：

  - ele.ele((By.CLASS_NAME, 'ele_class')) - 返回第一个class为ele_class的子元素

- 用查询字符串查找：

  属性、tag name和属性、文本、xpath、css selector。

  其中，@表示属性，=表示精确匹配，:表示模糊匹配，无控制字符串时默认搜索该字符串。

  - page.ele('@class:ele_class')                      - 返回第一个class含有ele_class的元素
  - page.ele('@name=ele_name')                    - 返回第一个name等于ele_name的元素
  - page.ele('@placeholder')                            - 返回第一个带placeholder属性的元素
  - page.ele('tag:p')                                          - 返回第一个<p>元素
  - page.ele('tag:div@class:ele_class')            - 返回第一个class含有ele_class的div元素
  - page.ele('tag:div@class=ele_class')           - 返回第一个class等于ele_class的div元素
  - page.ele('tag:div@text():some_text')           - 返回第一个文本含有some_text的div元素
  - page.ele('tag:div@text()=some_text')          - 返回第一个文本等于some_text的div元素
  - page.ele('text:some_text')                            - 返回第一个文本含有some_text的元素
  - page.ele('some_text')                                   - 返回第一个文本含有some_text的元素（等价于上一行）
  - page.ele('text=some_text')                           - 返回第一个文本等于some_text的元素
  - page.ele('xpath://div[@class="ele_class"]')  - 返回第一个符合xpath的元素
  - page.ele('css:div.ele_class')                         - 返回第一个符合css selector的元素

### eles()

​	eles(loc_or_str: Union[tuple, str], timeout: float = None, show_errmsg: bool = False) -> List[DriverElement]

​	根据查询参数获取符合条件的元素列表。查询参数使用方法和ele方法一致。

​	参数说明：

- loc_or_str        - 查询条件参数
- timeout            - 查找元素超时时间，driver模式下有效
- show_errmsg  - 出现异常时是否抛出及显示

### cookies_to_session()

​	cookies_to_session(copy_user_agent: bool = False) -> None

​	手动把cookies从driver复制到session。

​	参数说明：

- copy_user_agent  - 是否同时复制user agent

### cookies_to_driver()

​	cookies_to_driver(url=None) -> None

​	手动把cookies从session复制到driver。

​	参数说明：

- url  - cookies的域或url

### post()

​	post(url: str, params: dict = None, data: dict = None, go_anyway: bool = False, **kwargs) -> Union[bool, None]

​	以post方式跳转，调用时自动切换到session模式。

​	参数说明：

- url - 目标url
- parame        - url参数
- data             - 提交的数据
- go_anyway  - 是否强制跳转。若目标url和当前url一致，默认不跳转。
- kwargs         - headers等访问参数

### download()

​	download(file_url: str, goal_path: str = None, rename: str = None, file_exists: str = 'rename', show_msg: bool = False, **kwargs) -> tuple

​	下载一个文件，返回是否成功和下载信息字符串。改方法会自动避免和目标路径现有文件重名。

​	参数说明：

- file_url - 文件URL
- goal_path   - 存放路径，默认为ini文件中指定的临时文件夹
- rename       - 重命名文件，不改变扩展名
- file_exists   - 若存在同名文件，可选择'rename', 'overwrite', 'skip'方式处理
- show_msg  - 是否显示下载信息
- kwargs        - 用于requests的连接参数



以下方法和属性只有driver模式下生效，调用时会自动切换到driver模式

***

### tabs_count

​	返回标签页数量。

### tab_handles

​	返回所有标签页handle列表。

### current_tab_num

​	返回当前标签页序号。

### current_tab_handle

​	返回当前标签页handle。

### check_page()

​	check_page() -> bool

​	派生子类后用于检查域名是否符合预期，功能由子类实现。

### run_script()

​	run_script(script: str) -> Any

​	执行JavaScript代码。

​	参数说明：

- script - JavaScript代码文本

### create_tab()

​	create_tab(url: str = '') -> None

​	新建并定位到一个标签页,该标签页在最后面。

​	参数说明：

- url  - 新标签页跳转到的网址

### close_current_tab()

​	close_current_tab() -> None

​	关闭当前标签页。

### close_other_tabs()

​	close_other_tabs(num_or_handle: Union[int, str, None] = None) -> None

​	关闭传入的标签页以外标签页，默认保留当前页。

​	参数说明：

- num_or_handle  - 要保留的标签页序号或handle，序号第一个为0，最后为-1

### to_tab()

​	to_tab(num_or_handle: Union[int, str] = 0) -> None

​	跳转到标签页。

参数说明：

- num_or_handle  - 标签页序号或handle字符串，序号第一个为0，最后为-1

### to_iframe()

​	to_iframe(self, loc_or_ele: Union[int, str, tuple, WebElement, DriverElement] = 'main') -> None

​	跳转到iframe，默认跳转到最高层级，兼容selenium原生参数。

​	参数说明：

- loc_or_ele - 查找iframe元素的条件，可接收iframe序号(0开始)、id或name、查询字符串、loc参数、WebElement对象、DriverElement对象，传入'main'跳到最高层，传入'parent'跳到上一层

​	示例：
- to_iframe('tag:iframe')          - 通过传入iframe的查询字符串定位
- to_iframe('iframe_id')           - 通过iframe的id属性定位
- to_iframe('iframe_name')     - 通过iframe的name属性定位
- to_iframe(iframe_element)  - 通过传入元素对象定位
- to_iframe(0)                         - 通过iframe的序号定位
- to_iframe('main')                  - 跳到最高层
- to_iframe('parent')                - 跳到上一层

### scroll_to_see()

​	scroll_to_see(loc_or_ele: Union[str, tuple, WebElement, DriverElement]) -> None

​	滚动直到元素可见。

​	参数说明：

- loc_or_ele  - 查找iframe元素的条件，和ele()方法的查找条件一致。

### scroll_to()

​	scroll_to(mode: str = 'bottom', pixel: int = 300) -> None

​	滚动页面，按照参数决定如何滚动。

​	参数说明：

- mode  - 滚动的方向，top、bottom、rightmost、leftmost、up、down、left、right
- pixel    - 滚动的像素

### refresh()

​	refresh() -> None

​	刷新页面。

### back()

​	back() -> None

​	页面后退。

### set_window_size()

​	set_window_size(x: int = None, y: int = None) -> None

​	设置窗口大小，默认最大化。

​	参数说明：

- x  - 目标宽度
- y  - 目标高度

### screenshot()

​	screenshot(path: str, filename: str = None) -> str

​	网页截图，返回截图文件路径。

​	参数说明：

- path         - 截图保存路径，默认为ini文件中指定的临时文件夹
- filename  - 截图文件名，默认为页面title为文件名

### chrome_downloading()

​	chrome_downloading(download_path: str = None) -> list

​	查看浏览器下载情况。

​	参数说明：

- download_path  - 下载路径，默认为chrome options配置中的下载路径

### process_alert()

​	process_alert(mode: str = 'ok', text: str = None) -> Union[str, None]

​	处理提示框。

​	参数说明：

- mode  - 'ok' 或 'cancel'，若输入其它值，不会按按钮但依然返回文本值
- text     - 处理prompt提示框时可输入文本

### close_driver()

​	close_driver() -> None

​	关闭driver及浏览器，切换到s模式。

### close_session()

​	close_session() -> None

​	关闭session，切换到d模式。

## DriverElement类

class DriverElement(ele: WebElement, timeout: float = 10)

driver模式的元素对象，包装了一个WebElement对象，并封装了常用功能。

参数说明：

- ele - WebElement对象
- timeout - 查找元素超时时间（每次查找元素时还可单独设置）

### inner_ele

​	被包装的WebElement对象。

### driver

​	操作元素的WebDriver对象。

### attrs

​	以字典方式返回元素所有属性及值。

### text

​	返回元素内的文本。

### html

​	返回元素内html文本。

### tag

​	返回元素标签名文本。

### xpath

​	返回元素xpath路径。

### parent

​	返回父级元素对象。

### next

​	返回下一个兄弟元素对象。

### prev

​	返回上一个兄弟元素对象。

### parents()

​	parents(num: int = 1) -> Union[DriverElement, None]

​	返回第N层父级元素对象。

​	参数说明：

- 第几层父元素

### nexts()

​	nexts(num: int = 1) -> Union[DriverElement, None]

​	返回后面第N个兄弟元素对象。

​	参数说明：

- 后面第几个兄弟元素

### prevs()

​	prevs(num: int = 1) -> Union[DriverElement, None]

​	返回前面第N个兄弟元素对象。

​	参数说明：

- 前面第几个兄弟元素

### size

​	以字典方式返回元素大小。

### location

​	以字典方式放回元素坐标。

### ele()

​	ele(loc_or_str: Union[tuple, str], mode: str = None, show_errmsg: bool = False, timeout: float = None) -> Union[DriverElement, List[DriverElement], None]

​	根据查询参数获取元素。  
​	如查询参数是字符串，可选'@属性名:'、'tag:'、'text:'、'css:'、'xpath:'方式。无控制方式时默认用text方式查找。  
​	如是loc，直接按照内容查询。

​	参数说明：

- loc_or_str         - 查询条件参数
- mode               - 查找一个或多个，传入'single'或'all'
- show_errmsg  - 出现异常时是否抛出及显示
- timeout            - 查找元素超时时间

​	示例：

- 用loc元组查找：

  - ele.ele((By.CLASS_NAME, 'ele_class')) - 返回第一个class为ele_class的子元素

- 用查询字符串查找：

  属性、tag name和属性、文本、xpath、css selector。

  其中，@表示属性，=表示精确匹配，:表示模糊匹配，无控制字符串时默认搜索该字符串。

  - ele.ele('@class:ele_class')                      - 返回第一个class含有ele_class的元素
  - ele.ele('@name=ele_name')                    - 返回第一个name等于ele_name的元素
  - ele.ele('@placeholder')                            - 返回第一个带placeholder属性的元素
  - ele.ele('tag:p')                                          - 返回第一个<p>元素
  - ele.ele('tag:div@class:ele_class')            - 返回第一个class含有ele_class的div元素
  - ele.ele('tag:div@class=ele_class')           - 返回第一个class等于ele_class的div元素
  - ele.ele('tag:div@text():some_text')           - 返回第一个文本含有some_text的div元素
  - ele.ele('tag:div@text()=some_text')          - 返回第一个文本等于some_text的div元素
  - ele.ele('text:some_text')                            - 返回第一个文本含有some_text的元素
  - ele.ele('some_text')                                   - 返回第一个文本含有some_text的元素（等价于上一行）
  - ele.ele('text=some_text')                           - 返回第一个文本等于some_text的元素
  - ele.ele('xpath://div[@class="ele_class"]')  - 返回第一个符合xpath的元素
  - ele.ele('css:div.ele_class')                         - 返回第一个符合css selector的元素

### eles()

​	eles(loc_or_str: Union[tuple, str], show_errmsg: bool = False, timeout: float = None) ->  List[DriverElement]

​	根据查询参数获取符合条件的元素列表。查询参数使用方法和ele方法一致。

​	参数说明：

- loc_or_str        - 查询条件参数
- show_errmsg  - 出现异常时是否抛出及显示
- timeout            - 查找元素超时时间

### attr()

​	attr(attr: str) -> str

​	获取元素某个属性的值。

​	参数说明：

- attr  - 属性名称

### click()

​	click(by_js=None) -> bool

​	点击元素，如不成功则用js方式点击，可指定是否用js方式点击。

​	参数说明：

- by_js  - 是否用js方式点击

### input()

​	input(value, clear: bool = True) -> bool

​	输入文本。

​	参数说明：

- value  - 文本值
- clear  - 输入前是否清除文本框

### run_script()

​	run_script(script: str) -> Any

​	在元素上运行JavaScript。

​	参数说明：

- script  - JavaScript文本

### submit()

​	submit() -> None

​	提交表单。

### clear()

​	clear() -> None

​	清空文本框。

### is_selected()

​	is_selected() -> bool

​	元素是否被选中。

### is_enabled()

​	is_enabled() -> bool

​	元素在页面中是否可用。

### is_displayed()

​	is_displayed() -> bool

​	元素是否可见。

### is_valid()

​	is_valid() -> bool

​	元素是否有效。该方法用于判断页面跳转元素不能用的情况

### screenshot()

​	screenshot(path: str, filename: str = None) -> str

​	网页截图，返回截图文件路径。

​	参数说明：

- path         - 截图保存路径，默认为ini文件中指定的临时文件夹
- filename  - 截图文件名，默认为页面title为文件名

### select()

​	select(text: str) -> bool

​	在下拉列表中选择。

​	参数说明：

- text  - 选项文本

### set_attr()

​	set_attr(attr: str, value: str) -> bool

​	设置元素属性。

​	参数说明：

- attr     - 参数名
- value  - 参数值

### drag()

​	drag(x: int, y: int, speed: int = 40, shake: bool = True) -> bool

​	拖拽当前元素一段距离，返回是否拖拽成功。

​	参数说明：

- x          - 拖拽x方向距离
- y          - 拖拽y方向距离
- speed  - 拖拽速度
- shake  - 是否随机抖动

### drag_to()

​	drag_to(ele_or_loc: Union[tuple, WebElement, DrissionElement], speed: int = 40, shake: bool = True) -> bool:

​	拖拽当前元素，目标为另一个元素或坐标元组，返回是否拖拽成功。

​	参数说明：

- ele_or_loc  - 另一个元素或相对当前位置，坐标为元素中点坐标。
- speed         - 拖拽速度
- shake         - 是否随机抖动

### hover()

​	hover()

​	在元素上悬停鼠标



## SessionElement类

class SessionElement(ele: Element)

session模式的元素对象，包装了一个Element对象，并封装了常用功能。

参数说明：

- ele  - requests_html库的Element对象

### inner_ele

​	被包装的Element对象。

### attrs

​	以字典格式返回元素所有属性的名称和值。

### text

​	返回元素内的文本。

### html

​	返回元素内html文本。

### tag

​	返回元素标签名文本。

### xpath

​	返回元素xpath路径。

### parent

​	返回父级元素对象。

### next

​	返回下一个兄弟元素对象。

### prev 

​	返回上一个兄弟元素对象。

### parents()

​	parents(num: int = 1) -> Union[SessionElement, None]

​	返回第N层父级元素对象。

​	参数说明：

- num  - 第几层父元素

### nexts()

​	nexts(num: int = 1) -> Union[SessionElement, None]

​	返回后N个兄弟元素对象。

​	参数说明：

- num  - 后面第几个兄弟元素

### prevs()

​	prevs(num: int = 1) -> Union[SessionElement, None]

​	返回前N个兄弟元素对象。

​	参数说明：

- num  - 前面第几个兄弟元素

### ele()

​	ele(loc_or_str: Union[tuple, str], mode: str = None, show_errmsg: bool = False) -> Union[SessionElement, List[SessionElement], None]

​	根据查询参数获取元素。  
​	如查询参数是字符串，可选'@属性名:'、'tag:'、'text:'、'css:'、'xpath:'方式。无控制方式时默认用text方式查找。  
​	如是loc，直接按照内容查询。

​	参数说明：

- loc_or_str        - 查询条件参数

- mode               - 查找一个或多个，传入'single'或'all'

- show_errmsg  - 出现异常时是否抛出及显示

​	示例：

- 用loc元组查找：

  - ele.ele((By.CLASS_NAME, 'ele_class')) - 返回第一个class为ele_class的子元素

- 用查询字符串查找：

  属性、tag name和属性、文本、xpath、css selector。

  其中，@表示属性，=表示精确匹配，:表示模糊匹配，无控制字符串时默认搜索该字符串。

  - ele.ele('@class:ele_class')                      - 返回第一个class含有ele_class的元素
  - ele.ele('@name=ele_name')                    - 返回第一个name等于ele_name的元素
  - ele.ele('@placeholder')                            - 返回第一个带placeholder属性的元素
  - ele.ele('tag:p')                                          - 返回第一个<p>元素
  - ele.ele('tag:div@class:ele_class')            - 返回第一个class含有ele_class的div元素
  - ele.ele('tag:div@class=ele_class')           - 返回第一个class等于ele_class的div元素
  - ele.ele('tag:div@text():some_text')           - 返回第一个文本含有some_text的div元素
  - ele.ele('tag:div@text()=some_text')          - 返回第一个文本等于some_text的div元素
  - ele.ele('text:some_text')                            - 返回第一个文本含有some_text的元素
  - ele.ele('some_text')                                   - 返回第一个文本含有some_text的元素（等价于上一行）
  - ele.ele('text=some_text')                           - 返回第一个文本等于some_text的元素
  - ele.ele('xpath://div[@class="ele_class"]')  - 返回第一个符合xpath的元素
  - ele.ele('css:div.ele_class')                         - 返回第一个符合css selector的元素

### eles()

​	eles(loc_or_str: Union[tuple, str], show_errmsg: bool = False) ->  List[SessionElement]

​	根据查询参数获取符合条件的元素列表。查询参数使用方法和ele方法一致。

​	参数说明：

- loc_or_str        - 查询条件参数
- show_errmsg  - 出现异常时是否抛出及显示

### attr()

​	attr(attr: str) -> str

​	获取元素某个属性的值。

​	参数说明：

- attr  - 属性名称



## OptionsManager类

​	class OptionsManager(path: str = None)

​	管理配置文件内容的类。

​	参数说明：

- path  - ini文件路径，不传入则默认读取当前文件夹下的configs.ini文件

### get_value()

​	get_value(section: str, item: str) -> Any

​	获取配置的值。

​	参数说明：

- section  - 段落名称
- item       - 配置项名称

### get_option()

​	get_option(section: str) -> dict

​	以字典的格式返回整个段落的配置信息。

​	参数说明：

- section  - 段落名称

### set_item()

​	set_item(section: str, item: str, value: str) -> OptionsManager

​	设置配置值。

​	参数说明：

- section  - 段落名称
- item      - 配置项名称
- value     - 值内容

### save()

​	save(path: str = None) -> OptionsManager

​	保存设置到文件。

​	参数说明：

- path  - ini文件的路径，默认保存到模块文件夹下的



## DriverOptions类

​	class DriverOptions(read_file=True)

​	chrome浏览器配置类，继承自selenium.webdriver.chrome.options的Options类，增加了删除配置和保存到文件方法。

​	参数说明：

- read_file  - 布尔型，指定创建时是否从ini文件读取配置信息

### driver_path

​	chromedriver.exe的路径。

### chrome_path

​	chrome.exe的路径

### remove_argument()

​	remove_argument(value: str) -> DriverOptions

​	移除一个设置。

​	参数说明：

- value  - 要移除的属性值

### remove_experimental_option()

​	remove_experimental_option(key: str) -> DriverOptions

​	移除一个实验设置，传入key值删除。

​	参数说明：

- key  - 要移除的实验设置key值

### remove_argument()

​	remove_argument() -> DriverOptions

​	移除所有插件，因插件是以整个文件储存，难以移除其中一个，故如须设置则全部移除再重设。

### save()

​	save(path: str = None) -> DriverOptions

​	保存设置到文件。

​	参数说明：

- path  - ini文件的路径，默认保存到模块文件夹下的

### set_argument()

​	set_argument(arg: str, value: Union[bool, str]) -> DriverOptions

​	设置chrome属性，无值的属性可设置开关，有值的属性可设置属性的值。

​	参数说明：

- arg     - 属性名
- value  - 属性值，有值的属性传入值，没有的传入bool

### set_headless()

​	set_headless(on_off: bool = True) -> DriverOptions

​	打开或关闭无界面模式。

​	参数说明：

​	on_off  - 打开或关闭，bool

### set_no_imgs()

​	set_no_imgs(on_off: bool = True) -> DriverOptions

​	是否加载图片。

​	参数说明：

​	on_off  - 打开或关闭，bool

### set_no_js()

​	set_no_js(on_off: bool = True) -> DriverOptions

​	是否禁用js。

​	参数说明：

​	on_off  - 打开或关闭，bool

### set_mute()

​	set_mute(on_off: bool = True) -> DriverOptions

​	是否静音。

​	参数说明：

​	on_off  - 打开或关闭，bool

### set_user_agent()

​	set_user_agent(user_agent: str) -> DriverOptions

​	设置浏览器user agent。

​	参数说明：

- user_agent  - user agent字符串

### set_proxy()

​	set_proxy(proxy: str) -> DriverOptions

​	设置代理。

​	参数说明：

- proxy  - 代理地址

### set_paths()

​	set_paths(driver_path: str = None, chrome_path: str = None, debugger_address: str = None, download_path: str = None, user_data_path: str = None, cache_path: str = None) -> DriverOptions

​	设置浏览器相关的路径。

​	参数说明：

- driver_path              - chromedriver.exe的路径
- chrome_path           - chrome.exe的路径
- debugger_address  - 调试浏览器地址，例：127.0.0.1:9222
- download_path        - 下载文件路径
- user_data_path        - 用户数据路径
- cache_path              - 缓存路径



## easy_set方法

chrome的配置太难记，所以把常用的配置写成简单的方法，调用会修改ini文件相关内容。

### set_paths()

​	set_paths(driver_path: str = None, chrome_path: str = None, debugger_address: str = None, global_tmp_path: str = None, download_path: str = None, user_data_path: str = None, cache_path: str = None, check_version: bool = True) -> None

​	便捷的设置路径方法，把传入的路径保存到默认ini文件，并检查chrome和chromedriver版本是否匹配。

​	参数说明：

- driver_path              - chromedriver.exe路径
- chrome_path           - chrome.exe路径
- debugger_address  - 调试浏览器地址，例：127.0.0.1:9222
- download_path        - 下载文件路径
- global_tmp_path      - 临时文件夹路径
- user_data_path        - 用户数据路径
- cache_path               - 缓存路径
- check_version           - 是否检查chromedriver和chrome是否匹配

### set_argument()

​	set_argument(arg: str, value: Union[bool, str]) -> None

​	设置属性。若属性无值（如'zh_CN.UTF-8'），value传入bool表示开关；否则value传入str，当value为''或False，删除该属性项。

​	参数说明：

- arg     - 属性名
- value  - 属性值，有值的属性传入值，没有的传入bool

### set_headless()

​	set_headless(on_off: bool) -> None

​	开启或关闭headless模式。

​	参数说明：

- on_off  - 是否开启headless模式

### set_no_imgs()

​	set_no_imgs(on_off: bool) -> None

​	开启或关闭图片显示。

​	参数说明：

- on_off  - 是否开启无图模式

### set_no_js()

​	set_no_js(on_off: bool) -> None

​	开启或关闭禁用JS模式。

​	参数说明：

- on_off  - 是否开启禁用JS模式

### set_mute()

​	set_mute(on_off: bool) -> None

​	开启或关闭静音模式。

​	参数说明：

- on_off  - 是否开启静音模式

### set_user_agent()

​	set_user_agent(user_agent: str) -> None:

​	设置user_agent。

​	参数说明：

- user_agent  - user_agent值

### set_proxy()

​	set_proxy(proxy: str) -> None

​	设置代理。

​	参数说明：

- proxy  - 代理值

### check_driver_version()

​	check_driver_version(driver_path: str = None, chrome_path: str = None) -> bool

​	检查chrome与chromedriver版本是否匹配。

​	参数说明：

- driver_path     - chromedriver.exe路径
- chrome_path  - chrome.exe路径