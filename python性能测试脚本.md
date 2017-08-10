# python性能测试脚本


    import httplib
    import urllib
    import time
    import json
     
    class Transaction(object):
             
        def __init__(self):
            self.custom_timers = {}
     
        def run(self):
            conn = httplib.HTTPConnection("localhost:8080")
            headers = {"Content-type": "application/json"} #application/x-www-form-urlencoded,"Aceept":"text/plain"
            params = ({"bindHyCardInfo":{"mobileNo":"1881026xxxx","userId":"2","hYCardno":line,"bankCardNo":"622xxxxxxxxxxxxx","ip":"127.0.0.1"},"header":{"version":"1.0.1","from":"1000","to":"2000","tid":line,"time":"12312","token":"SEW342WEER2342","ext":""}})
            start = time.time()
            conn.request("POST", "/core-oper/rest/bindHyCard", json.JSONEncoder().encode(params), headers)
            response = conn.getresponse()
            response_time = time.time()
            data = response.read()
            print data
            conn.close()
            transfer_time = time.time()
            self.custom_timers['response received'] = response_time - start
            self.custom_timers['content transferred'] = transfer_time - start
             
    if __name__ == '__main__':
         
        file = open("E://card.txt")
        while 1:
            lines = file.readlines()
            if not lines:
                break
            for line in lines:
                line = line.strip('\n')
                trans = Transaction()
                trans.run()
                for timer in ('response received', 'content transferred'):
                    print '%s: %.5f secs' % (timer, trans.custom_timers[timer])
        file.close()

  

