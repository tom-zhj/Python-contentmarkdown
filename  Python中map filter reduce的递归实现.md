#  Python中map filter reduce的递归实现


    map2=lambda f,seq: [] if seq==[] else [f(seq[0])] + map2(f, seq[1:])
     
    filter2=lambda f, seq: [] if seq==[] else ( [seq[0]]+filter2(f, seq[1:]) if f(seq[0]) else filter2(f, seq[1:]) )
     
    reduce2=lambda f,seq,x: x if seq==[] else reduce2(f, seq[1:], f(x, seq[0]))
     
    scanl=lambda f,seq,x: [x] if seq==[] else  [x] +scanl(f, seq[1:], f(x,seq[0]))
     
     
    print map2(str, [1,2,3,5,8])
    print filter2(lambda x: x%2==0, range(10))
    print reduce2(lambda x,y: ''.join( ['(',x,'+',y,')'] ), map(str,range(1,11)), '0')
    print scanl(lambda x,y: ''.join( ['(',x,'+',y,')'] ), map(str,range(1,5)), '0')
     
     
    #Out:
    #['1', '2', '3', '5', '8']
    #[0, 2, 4, 6, 8]
    #((((((((((0+1)+2)+3)+4)+5)+6)+7)+8)+9)+10)
    #['0', '(0+1)', '((0+1)+2)', '(((0+1)+2)+3)', '((((0+1)+2)+3)+4)']

Python中map filter reduce的递归实现,注意：**要小心内存溢出**  

