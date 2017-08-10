# PYTHON如何在内存中生成ZIP文件

如题，代码如下：

    
    
    class MemoryZipFile(object):
       def __init__(self):
           #创建内存文件
           self._memory_zip = StringIO.StringIO()
           
       def append_content(self, filename_in_zip, file_content):
           """
           description: 写文本内容到zip
           """
           zf = zipfile.ZipFile(self._memory_zip, "a", zipfile.ZIP_DEFLATED, False)
           zf.writestr(filename_in_zip, file_content)
           for zfile in zf.filelist: zfile.create_system = 0
           return self
            
       def append_file(self, filename_in_zip, local_file_full_path):
           """

  

      description:写文件内容到zip

      注意这里的第二个参数是本地磁盘文件的全路径(windows:c/demo/1.jpg | linux:
/usr/local/test/1.jpg)

      """

      zf = zipfile.ZipFile(self._memory_zip, "a", zipfile.ZIP_DEFLATED, False)

      zf.write(local_file_full_path, filename_in_zip)

      for zfile in zf.filelist: zfile.create_system = 0

      return self



  def read(self):

      """

      description: 读取zip文件内容

      """

      self._memory_zip.seek(0)

      return self._memory_zip.read()



  def write_file(self, filename):

      """

      description:写zip文件到磁盘

      """

      f = file(filename, "wb")

      f.write(self.read())

      f.close()

  

  

  

使用方法如下：

  

    
    
            mem_zip_file = MemoryZipFile()
            mem_zip_file.append_content('mimetype', "application/epub+zip")
            mem_zip_file.append_content('META-INF/container.xml', '''<?xml version="1.0" encoding="UTF-8" ?>
    <container version="1.0" xmlns="urn:oasis:names:tc:opendocument:xmlns:container"> </container>''');
            #追加磁盘上的文件内容到内存，注意这里的第二个参数是本地磁盘文件的全路径(windows:c/demo/1.jpg | linux: /usr/local/test/1.jpg)
            mem_zip_file.append_file("1.jpg", "c:\1.jpg")
     
     
            #将内存中的zip文件写入磁盘
            mem_zip_file.write_file("c:test.zip")
     
            #获取内存zip文件内容
            data = mem_zip_file.read()
     
            #上传到fdfs
            my_fdfs_client.upload_by_buffer(data, 'zip')

  

