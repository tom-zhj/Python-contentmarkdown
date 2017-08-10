# python去除html标签

python去除html标签，自己写的，若有不足请指正：  

    
    
    #! /usr/bin/env python
    #coding=utf-8
    # blueel 2013-01-19
    from HTMLParser import HTMLParser
     
    class MLStripper(HTMLParser):
        def __init__(self):
            self.reset()
            self.fed = []
        def handle_data(self, d):
            self.fed.append(d)
        def get_data(self):
            return ''.join(self.fed)
     
    def strip_tags(html):
        s = MLStripper()
        s.feed(html)
        return s.get_data()

调用：  

html = '<em productIndex="0" class="valor-dividido"
style="display:block"><span>ou <strong><label productIndex="0"
class="skuBestInstallmentNumber">12</label>X</strong> de <strong> <label
productIndex="0" class="skuBestInstallmentValue">R$  116,58</label></strong>
sem juros</span></em>'

  

print strip_tags(html)

  

