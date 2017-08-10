# 用python脚本监控并发量

该脚本作用用于查询日志过去一分钟内的并发量，并发单位位1分钟，结果打印在标准输出中，可以配合一些软件实现日志的并发实时监控，比如zabbix。

    
    
    #! /usr/local/bin/python3
    import sys
    import re
    import datetime
    import os
    def generate_previous_minutes():
        format='%d/%b/%Y:%H:%M'
        return (datetime.datetime.today()-datetime.timedelta(minutes=1)).strftime(format)
    def check_logs(log_path,examine_minutes):
        regex_minutes=re.compile(examine_minutes)
        minutes_count=0
        step=10*1024*1024
        with open(log_path,encoding='Latin-1') as file:
            line=file.readline()
            while line:
                time_line=line.split(' ')[3][1:]
                if time_line>=examine_minutes:
                    file.seek(file.tell()-step)
                    file.readline()
                    break
                file.seek(file.tell()+step)
                if file.tell()>=os.path.getsize(log_path):
                    file.seek(file.tell()-step)
                    file.readline()
                    break
                file.readline()
                line=file.readline().strip()
            for line in file:
                line=line.strip()
                words=line.split(' ')
                if(regex_minutes.search(words[3])):
                    minutes_count+=1
        print(minutes_count)
    def main(log_path):
        previous_minutes=generate_previous_minutes()
        print(previous_minutes)
        check_logs(log_path,previous_minutes)
    if __name__ == '__main__':
        log_path=sys.argv[1]
        main(log_path)

  

