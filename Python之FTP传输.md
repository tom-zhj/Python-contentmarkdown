# Python之FTP传输

访问FTP，无非两件事情：upload和download，最近在项目中需要从ftp下载大量文件，然后我就试着去实验自己的ftp操作类，如下（PS：此段有问题
，别复制使用，可以参考去试验自己的ftp类！）  

    
    
    import os
    from ftplib import FTP
     
    class FTPSync():
        def __init__(self, host, usr, psw, log_file):
            self.host = host
            self.usr = usr
            self.psw = psw
            self.log_file = log_file
         
        def __ConnectServer(self):
            try:
                self.ftp = FTP(self.host)
                self.ftp.login(self.usr, self.psw)
                self.ftp.set_pasv(False)
                return True
            except Exception:
                return False
         
        def __CloseServer(self):
            try:
                self.ftp.quit()
                return True
            except Exception:
                return False
         
        def __CheckSizeEqual(self, remoteFile, localFile):
            try:
                remoteFileSize = self.ftp.size(remoteFile)
                localFileSize = os.path.getsize(localFile)
                if localFileSize == remoteFileSize:
                    return True
                else:
                    return False
            except Exception:
                return None
             
        def __DownloadFile(self, remoteFile, localFile):
            try:
                self.ftp.cwd(os.path.dirname(remoteFile))
                f = open(localFile, 'wb')
                remoteFileName = 'RETR ' + os.path.basename(remoteFile)
                self.ftp.retrbinary(remoteFileName, f.write)
                 
                if self.__CheckSizeEqual(remoteFile, localFile):
                    self.log_file.write('The File is downloaded successfully to %s' + '\n' % localFile)
                    return True
                else:
                    self.log_file.write('The localFile %s size is not same with the remoteFile' + '\n' % localFile)
                    return False
            except Exception:
                return False
         
        def __DownloadFolder(self, remoteFolder, localFolder):
            try:
                fileList = []
                self.ftp.retrlines('NLST', fileList.append)
                for remoteFile in fileList:
                    localFile = os.path.join(localFolder, remoteFile)
                    return self.__DownloadFile(remoteFile, localFile)
            except Exception:
                return False
         
        def SyncFromFTP(self, remoteFolder, localFolder):
            self.__DownloadFolder(remoteFolder, localFolder)
            self.log_file.close()
            self.__CloseServer()

  

