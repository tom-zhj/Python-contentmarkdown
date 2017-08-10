# 如何使用Python来生成sitemap

在做网站项目时，经常会使用脚本生成sitemap， 便于爬虫爬取，有利于SEO。 那么如何使用Python来生成sitemap呢？下面我们来研究一番。

安装lxml

首先需要pip install lxml安装lxml库。

如果你在ubuntu上遇到了以下错误:

    
    
    #include "libxml/xmlversion.h"
    compilation terminated.
    error: command 'x86_64-linux-gnu-gcc' failed with exit status 1
    ----------------------------------------
    Cleaning up...
     Removing temporary dir /tmp/pip_build_root...
    Command /usr/bin/python -c "import setuptools, tokenize;__file__='/tmp/pip_build_root/lxml/setup.py';exec(compile(getattr(tokenize, 'open', open)(__file__).read().replace('\r\n', '\n'), __file__, 'exec'))" install --record /tmp/pip-O4cIn6-record/install-record.txt --single-version-externally-managed --compile failed with error code 1 in /tmp/pip_build_root/lxml
    Exception information:
    Traceback (most recent call last):
     File "/usr/lib/python2.7/dist-packages/pip/basecommand.py", line 122, in main
      status = self.run(options, args)
     File "/usr/lib/python2.7/dist-packages/pip/commands/install.py", line 283, in run
      requirement_set.install(install_options, global_options, root=options.root_path)
     File "/usr/lib/python2.7/dist-packages/pip/req.py", line 1435, in install
      requirement.install(install_options, global_options, *args, **kwargs)
     File "/usr/lib/python2.7/dist-packages/pip/req.py", line 706, in install
      cwd=self.source_dir, filter_stdout=self._filter_install, show_stdout=False)
     File "/usr/lib/python2.7/dist-packages/pip/util.py", line 697, in call_subprocess
      % (command_desc, proc.returncode, cwd))
    InstallationError: Command /usr/bin/python -c "import setuptools, tokenize;__file__='/tmp/pip_build_root/lxml/setup.py';exec(compile(getattr(tokenize, 'open', open)(__file__).read().replace('\r\n', '\n'), __file__, 'exec'))" install --record /tmp/pip-O4cIn6-record/install-record.txt --single-version-externally-managed --compile failed with error code 1 in /tmp/pip_build_root/lxml

请安装以下依赖：

    
    
    sudo apt-get install libxml2-dev libxslt1-dev

Python代码

下面是生成sitemap和sitemapindex索引的代码，可以按照需求传入需要的参数，或者增加字段：

    
    
    #!/usr/bin/env python
    # -*- coding:utf-8 -*-
    import io
    import re
    from lxml import etree
    def generate_xml(filename, url_list):
      """Generate a new xml file use url_list"""
      root = etree.Element('urlset',
                 xmlns="http://www.sitemaps.org/schemas/sitemap/0.9")
      for each in url_list:
        url = etree.Element('url')
        loc = etree.Element('loc')
        loc.text = each
        url.append(loc)
        root.append(url)
      header = u'<?xml version="1.0" encoding="UTF-8"?>\n'
      s = etree.tostring(root, encoding='utf-8', pretty_print=True)
      with io.open(filename, 'w', encoding='utf-8') as f:
        f.write(unicode(header+s))
    def update_xml(filename, url_list):
      """Add new url_list to origin xml file."""
      f = open(filename, 'r')
      lines = [i.strip() for i in f.readlines()]
      f.close()
      old_url_list = []
      for each_line in lines:
        d = re.findall('<loc>(http:\/\/.+)<\/loc>', each_line)
        old_url_list += d
      url_list += old_url_list
      generate_xml(filename, url_list)
    def generatr_xml_index(filename, sitemap_list, lastmod_list):
      """Generate sitemap index xml file."""
      root = etree.Element('sitemapindex',
                 xmlns="http://www.sitemaps.org/schemas/sitemap/0.9")
      for each_sitemap, each_lastmod in zip(sitemap_list, lastmod_list):
        sitemap = etree.Element('sitemap')
        loc = etree.Element('loc')
        loc.text = each_sitemap
        lastmod = etree.Element('lastmod')
        lastmod.text = each_lastmod
        sitemap.append(loc)
        sitemap.append(lastmod)
        root.append(sitemap)
      header = u'<?xml version="1.0" encoding="UTF-8"?>\n'
      s = etree.tostring(root, encoding='utf-8', pretty_print=True)
      with io.open(filename, 'w', encoding='utf-8') as f:
        f.write(unicode(header+s))
    if __name__ == '__main__':
      urls = ['http://www.baidu.com'] * 10
      mods = ['2004-10-01T18:23:17+00:00'] * 10
      generatr_xml_index('index.xml', urls, mods)

效果

生成的效果应该是这种格式：

sitemap格式:

    
    
    <?xml version="1.0" encoding="UTF-8"?>
    <urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
     <url>
      <loc>http://www.example.com/foo.html</loc>
     </url>
    </urlset>
    sitemapindex格式：
    <?xml version="1.0" encoding="UTF-8"?>
      <sitemapindex xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
      <sitemap>
       <loc>http://www.example.com/sitemap1.xml.gz</loc>
       <lastmod>2004-10-01T18:23:17+00:00</lastmod>
      </sitemap>
      <sitemap>
       <loc>http://www.example.com/sitemap2.xml.gz</loc>
       <lastmod>2005-01-01</lastmod>
      </sitemap>
      </sitemapindex>

lastmod时间格式的问题

格式是用ISO 8601的标准，如果是linux/unix系统，可以使用以下函数获取

    
    
    def get_lastmod_time(filename):
      time_stamp = os.path.getmtime(filename)
      t = time.localtime(time_stamp)
      # return time.strftime('%Y-%m-%dT%H:%M:%S+08:00', t)
      return time.strftime('%Y-%m-%dT%H:%M:%SZ', t)

优化

一般来说，用lxml效率低并且内存占用比较大，可以直接用文件的write方法创建。

    
    
    def generate_xml(filename, url_list):
      with gzip.open(filename,"w") as f:
        f.write("""<?xml version="1.0" encoding="utf-8"?>
    <urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">\n""")
        for i in url_list:
          f.write("""<url><loc>%s</loc></url>\n"""%i)
        f.write("""</urlset>""")
    def append_xml(filename, url_list):
      with gzip.open(filename, 'r') as f:
        for each_line in f:
          d = re.findall('<loc>(http:\/\/.+)<\/loc>', each_line)
          url_list.extend(d)
        generate_xml(filename, set(url_list))
    def modify_time(filename):
      time_stamp = os.path.getmtime(filename)
      t = time.localtime(time_stamp)
      return time.strftime('%Y-%m-%dT%H:%M:%S:%SZ', t)
    def new_xml(filename, url_list):
      generate_xml(filename, url_list)
      root = dirname(filename)
      with open(join(dirname(root), "sitemap.xml"),"w") as f:
        f.write('<?xml version="1.0" encoding="utf-8"?>\n<sitemapindex xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">\n')
        for i in glob.glob(join(root,"*.xml.gz")):
          lastmod = modify_time(i)
          i = i[len(CONFIG.SITEMAP_PATH):]
          f.write("<sitemap>\n<loc>http:/%s</loc>\n"%i)
          f.write("<lastmod>%s</lastmod>\n</sitemap>\n"%lastmod)
        f.write('</sitemapindex>')

  

