# Python爬虫使用代理proxy抓取网页

代理类型（proxy）:透明代理 匿名代理 混淆代理和高匿代理. 这里写一些python爬虫使用代理的知识, 还有一个代理池的类.
方便大家应对工作中各种复杂的抓取问题。

### urllib 模块使用代理

urllib/urllib2使用代理比较麻烦, 需要先构建一个ProxyHandler的类,
随后将该类用于构建网页打开的opener的类,再在request中安装该opener.

代理格式是"http://127.0.0.1:80",如果要账号密码是"http://user:password@127.0.0.1:80".

    
    
    proxy="http://127.0.0.1:80"
    # 创建一个ProxyHandler对象
    proxy_support=urllib.request.ProxyHandler({'http':proxy})
    # 创建一个opener对象
    opener = urllib.request.build_opener(proxy_support)
    # 给request装载opener
    urllib.request.install_opener(opener)
    # 打开一个url
    r = urllib.request.urlopen('http://youtube.com',timeout = 500)

### requests 模块 使用代理

requests使用代理要比urllib简单多了…这里以单次代理为例. 多次的话可以用session一类构建.

如果需要使用代理，你可以通过为任意请求方法提供 proxies 参数来配置单个请求:

    
    
    import requests
    proxies = {
      "http": "http://127.0.0.1:3128",
      "https": "http://127.0.0.1:2080",
    }
    r=requests.get("http://youtube.com", proxies=proxies)
    print r.text

你也可以通过环境变量 HTTP_PROXY 和 HTTPS_PROXY 来配置代理。

    
    
    export HTTP_PROXY="http://127.0.0.1:3128"
    export HTTPS_PROXY="http://127.0.0.1:2080"
    python
    >>> import requests
    >>> r=requests.get("http://youtube.com")
    >>> print r.text

若你的代理需要使用HTTP Basic Auth，可以使用 http://user:password@host/ 语法:

    
    
    proxies = {
        "http": "http://user:pass@127.0.0.1:3128/",
    }

  

python的代理使用非常简单， 最重要的是要找一个网络稳定可靠的代理，有问题欢迎留言提问

