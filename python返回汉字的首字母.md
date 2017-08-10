# python返回汉字的首字母

def get_first_letter(char):

   char=char.encode('GBK')



   if char<b"\xb0\xa1" or char>b"\xd7\xf9":

       return ""

   if char<b"\xb0\xc4":

       return "a"

   if char<b"\xb2\xc0":

       return "b"

   if char<b"\xb4\xed":

       return "c"

   if char<b"\xb6\xe9":

       return "d"

   if char<b"\xb7\xa1":

       return "e"

   if char<b"\xb8\xc0":

       return "f"

   if char<b"\xb9\xfd":

       return "g"

   if char<b"\xbb\xf6":

       return "h"

   if char<b"\xbf\xa5":

       return "j"

   if char<b"\xc0\xab":

       return "k"

   if char<b"\xc2\xe7":

       return "l"

   if char<b"\xc4\xc2":

       return "m"

   if char<b"\xc5\xb5":

       return "n"

   if char<b"\xc5\xbd":

       return "o"

   if char<b"\xc6\xd9":

       return "p"

   if char<b"\xc8\xba":

       return "q"

   if char<b"\xc8\xf5":

       return "r"

   if char<b"\xcb\xf9":

       return "s"

   if char<b"\xcd\xd9":

       return "t"

   if char<b"\xce\xf3":

       return "w"

   if char<b"\xd1\x88":

       return "x"

   if char<b"\xd4\xd0":

       return "y"

   if char<b"\xd7\xf9":

       return "z"

  

