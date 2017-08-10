# 使用Python编写MapReduce作业

mrjob 可以让用 Python 2.5+ 来编写 MapReduce 作业，并在多个不同平台上运行，你可以：

使用纯 Python 编写多步的 MapReduce 作业

在本机上进行测试

在 Hadoop 集群上运行

使用 Amazon Elastic MapReduce (EMR) 在云上运行

pip 的安装方法非常简单，无需配置，直接运行：pip install mrjob

代码实例：

    
    
    from mrjob.job import MRJob
    class MRWordCounter(MRJob):
        def mapper(self, key, line):
            for word in line.split():
                yield word, 1
        def reducer(self, word, occurrences):
            yield word, sum(occurrences)
    if __name__ == '__main__':
        MRWordCounter.run()

  

