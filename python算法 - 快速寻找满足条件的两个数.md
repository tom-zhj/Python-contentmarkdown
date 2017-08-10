# python算法 - 快速寻找满足条件的两个数

题目前提是一定存在这样两个数

解法一就不写了...一般想不到吧

一开始想到的是解法二最后的用hash表

（其实是想到创建一个跟target一样大的数组啦..存在就写入index，但是要全部找出，那得二维数组，但是后面想到target要是很大的话，是不是浪费空间
了...所以改成Dict）

后面发现题目只要求给出两个数就好了啊- -

扩展问题比较有意思

找三个应该不难，其它还不清楚，有想再补充...

1.二维数组

    
    
    def find_pair(A, target):
        B = [[] for i in range(target + 1)]
        for i in range(0, len(A)):
            if A[i] <= target:
                B[A[i]].append(i)
        for i in range(0, target / 2 + 1):
            if len(B[i]) != 0 and len(B[target - i]) != 0:
                print(i, B[i], target-i, B[target-i])
     
    if __name__ == "__main__":
        A = [0, 1, 1, 2, 11, 8, 3, 4, 5, 6, 7, 8, 9, 10]
        find_pair(A, 9)

2.字典

    
    
    def find_pair(A, target):
        B = {}
        for i in range(0, len(A)):
            if A[i] <= target:
                if not B.has_key(A[i]):
                    B[A[i]] = [i]
                else:
                    B[A[i]].append(i)
        for i in range(0, target / 2 + 1):
            if B.has_key(i) and B.has_key(target-i):
                print(i, B[i], target-i, B[target-i])
     
    if __name__ == "__main__":
        A = [0, 1, 1, 2, 11, 8, 3, 4, 5, 6, 7, 8, 9, 10]
        find_pair(A, 9)

3.这种方法都已经重新排序了，不知道书上还返回索引有什么意义...排序偷懒直接用内置的啦...  

    
    
    def find_pair(A, target):
        A.sort()
        i, j = 0, len(A) - 1
        while i < j:
            s = A[i] + A[j]
            if s == target:
                print(i, A[i], j, A[j])
                i += 1
                j -= 1
            elif s < target:
                i += 1
            else:
                j -= 1
     
    if __name__ == "__main__":
        A = [0, 1, 1, 2, 11, 8, 3, 4, 5, 6, 7, 8, 9, 10]
        find_pair(A, 9)

  

