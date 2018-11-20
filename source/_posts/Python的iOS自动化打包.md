---
title: Python的iOS自动化打包
date: 2018年11月15日16:49:58
tags: [Python, Xcode, 自动化打包]
---

<img src="https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1542278885347&di=41c8d874cfdcbc1fbd137bf1fc457cab&imgtype=0&src=http%3A%2F%2Fs2.dgtle.com%2Fforum%2F201810%2F01%2F115737mkggkkbm2o6u8tbk.jpg" width=800>

### 前言

这段时间刚刚学习了一段时间的Python,加上自己是做iOS开发的,就想着用Python来做一个自动化打包,可以自动完成打包,上传到蒲公英,并且发送邮箱给测试人员.

一是可以减少打包功夫,二来可以练练手,结合自己的工作来输出一点东西.废话不多说,直接上代码...


### 原理

就是使用xcodebuild来控制Xcode进行一系列的操作,从而完成打包的操作.

<!-- more -->
<img src="https://ws3.sinaimg.cn/large/006tNbRwly1fx8u0gobnaj30tx0lujvf.jpg" width=500>

### 为什么要做这个？

在我们日常开发的时候,特别是在内部测试的时间,有可能需要频繁的打包,打包的工作比较繁琐,需要等待点击下一步,选择之类,影响了开发的节奏.(开玩笑,我能有啥节奏...), 为什么不能直接运行,然后完成所有的操作呢?

### 思路：
从网上查找了一些关于xcodebuild来打包的资料,从而得到:

1. 找到对应的项目
2. clean项目
3. archive项目
4. export IPA
5. 上传蒲公英
6. 发送邮件
7. 收工

思路有了,动手起来.

### 运行环境
Python, Xcode

这些需要大家直接去搭建好环境...

### 准备工作
1. 下载安装pycharm(这只是我开发Python的工具而已,大家可以根据自己喜欢的来选择)
2. 注册并认证蒲公英(不认证的话,是不能上传的)
3. 邮箱开启POP3/SMTP服务(我使用的是QQ邮箱),记录下16位授权码
4. 一个ExportOptions.plist文件, 这个下面会解释为什么需要还有怎么生成!
5. 一份iOS项目代码→_→

### 完整代码
```
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
# @Time    : 2018/11/14 11:04 AM
# @Author  : liangk
# @Site    :
# @File    : auto_archive_ios.py
# @Software: PyCharm


import os
import requests
import webbrowser
import subprocess
import time
import smtplib
from email.mime.text import MIMEText
from email import encoders
from email.header import Header
from email.utils import parseaddr, formataddr

project_name = 'TestArchive'    # 项目名称
archive_workspace_path = '/Users/用户/Desktop/TestArchive'    # 项目路径
export_directory = 'archive'    # 输出的文件夹
ipa_download_url = 'https://www.pgyer.com/XXX' #蒲公英的APP地址

# 蒲公英账号USER_KEY、API_KEY
USER_KEY = 'XXXXXXXXXXXXXXXXXXXX'
API_KEY = 'XXXXXXXXXXXXXXXXXXXX'

from_address = 'XXXXXXXXXXXXXXXXXXXX@qq.com'    # 发送人的地址
password = 'XXXXXXXXXXXXXXXXXXXX'  # 邮箱密码换成他提供的16位授权码
to_address = 'XXXXXXXXXXXXXXXXXXXX@qq.com'    # 收件人地址,可以是多个的
smtp_server = 'smtp.qq.com'    # 因为我是使用QQ邮箱..


class AutoArchive(object):
    """自动打包并上传到蒲公英,发邮件通知"""

    def __init__(self):
        pass

    def clean(self):
        print("\n\n===========开始clean操作===========")
        start = time.time()
        clean_command = 'xcodebuild clean -workspace %s/%s.xcworkspace -scheme %s -configuration Release' % (
            archive_workspace_path, project_name, project_name)
        clean_command_run = subprocess.Popen(clean_command, shell=True)
        clean_command_run.wait()
        end = time.time()
        # Code码
        clean_result_code = clean_command_run.returncode
        if clean_result_code != 0:
            print("=======clean失败,用时:%.2f秒=======" % (end - start))
        else:
            print("=======clean成功,用时:%.2f秒=======" % (end - start))
            self.archive()

    def archive(self):
        print("\n\n===========开始archive操作===========")

        # 删除之前的文件
        subprocess.call(['rm', '-rf', '%s/%s' % (archive_workspace_path, export_directory)])
        time.sleep(1)
        # 创建文件夹存放打包文件
        subprocess.call(['mkdir', '-p', '%s/%s' % (archive_workspace_path, export_directory)])
        time.sleep(1)

        start = time.time()
        archive_command = 'xcodebuild archive -workspace %s/%s.xcworkspace -scheme %s -configuration Release -archivePath %s/%s' % (
            archive_workspace_path, project_name, project_name, archive_workspace_path, export_directory)
        archive_command_run = subprocess.Popen(archive_command, shell=True)
        archive_command_run.wait()
        end = time.time()
        # Code码
        archive_result_code = archive_command_run.returncode
        if archive_result_code != 0:
            print("=======archive失败,用时:%.2f秒=======" % (end - start))
        else:
            print("=======archive成功,用时:%.2f秒=======" % (end - start))
            # 导出IPA
            self.export()

    def export(self):
        print("\n\n===========开始export操作===========")
        print("\n\n==========请你耐心等待一会~===========")
        start = time.time()
        # export_command = 'xcodebuild -exportArchive -archivePath /Users/liangk/Desktop/TestArchive/myArchivePath.xcarchive -exportPath /Users/liangk/Desktop/TestArchive/out -exportOptionsPlist /Users/liangk/Desktop/TestArchive/ExportOptions.plist'
        export_command = 'xcodebuild -exportArchive -archivePath %s/%s.xcarchive -exportPath %s/%s -exportOptionsPlist %s/ExportOptions.plist' % (
            archive_workspace_path, export_directory, archive_workspace_path, export_directory, archive_workspace_path)
        export_command_run = subprocess.Popen(export_command, shell=True)
        export_command_run.wait()
        end = time.time()
        # Code码
        export_result_code = export_command_run.returncode
        if export_result_code != 0:
            print("=======导出IPA失败,用时:%.2f秒=======" % (end - start))
        else:
            print("=======导出IPA成功,用时:%.2f秒=======" % (end - start))
            # 删除archive.xcarchive文件
            subprocess.call(['rm', '-rf', '%s/%s.xcarchive' % (archive_workspace_path, export_directory)])
            self.upload('%s/%s/%s.ipa' % (archive_workspace_path, export_directory, project_name))

    def upload(self, ipa_path):
        print("\n\n===========开始上传蒲公英操作===========")
        if ipa_path:
            # https://www.pgyer.com/doc/api 具体参数大家可以进去里面查看,
            url = 'http://www.pgyer.com/apiv1/app/upload'
            data = {
                'uKey': USER_KEY,
                '_api_key': API_KEY,
                'installType': '1',
                'updateDescription': description
            }
            files = {'file': open(ipa_path, 'rb')}
            r = requests.post(url, data=data, files=files)
            if r.status_code == 200:
            	# 是否需要打开浏览器
                # self.open_browser(self)
                self.send_email()
        else:
            print("\n\n===========没有找到对应的ipa===========")
            return

    @staticmethod
    def open_browser(self):
        webbrowser.open(ipa_download_url, new=1, autoraise=True)

    @staticmethod
    def _format_address(self, s):
        name, address = parseaddr(s)
        return formataddr((Header(name, 'utf-8').encode(), address))

    def send_email(self):
        # https://www.pgyer.com/XXX app地址
        # 只是单纯的发了一个文本邮箱,具体的发附件和图片大家可以自己去补充
        msg = MIMEText('<html><body><h1>Hello</h1>' +
                       '<p>╮(╯_╰)╭<a href="https://www.pgyer.com/XXX">应用已更新,请下载测试</a>╮(╯_╰)╭</p>' +
                       '<p>蒲公英的更新会有延迟,具体版本时间以邮件时间为准</p>' +
                       '</body></html>', 'html', 'utf-8')
        msg['From'] = self._format_address(self, 'iOS开发团队 <%s>' % from_address)
        msg['Subject'] = Header('来自iOS开发团队的问候……', 'utf-8').encode()
        server = smtplib.SMTP(smtp_server, 25)  # SMTP协议默认端口是25
        server.set_debuglevel(1)
        server.login(from_address, password)
        server.sendmail(from_address, [to_address], msg.as_string())
        server.quit()
        print("===========邮件发送成功===========")


if __name__ == '__main__':
    description = input("请输入内容:")
    archive = AutoArchive()
    archive.clean()

```
### 关于ExportOptions.plist文件
因为 Xcode 9+ 默认不允许访问钥匙串的内容，必须要设置 allowProvisioningUpdates 才会允许，Python的Xcode插件目前无法支持此项完成打包流程。

解决步骤如下：

1、手动Xcode10打包，导出ExportOptions.plist文件；

2、编辑ExportOptions.plist文件，配置 provisioningProfiles 对应填入Bundle identifier及证书关联配置文件(打包时自动匹配或手动填入证书，provisioningProfiles需配置的必填信息可自动生成)；

3、提供ExportOptions.plist文件路径供Python脚本调用（详请参看Python脚本代码）。


具体的内容
 
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>compileBitcode</key>//是否编译bitcode
    <true/>
    <key>method</key>
    <string>ad-hoc</string>/
    <key>provisioningProfiles</key>
    <dict>
        <key>文件bundle id</key>
        <string>Adhoc_ID</string>
    </dict>
    <key>signingCertificate</key>//证书签名
    <string>这里填证书签名</string>
    <key>signingStyle</key>
    <string>manual</string>
    <key>stripSwiftSymbols</key>
    <true/>
    <key>teamID</key>
    <string>AANCCUK4M3</string>//TeamID
    <key>thinning</key>
    <string>&lt;none&gt;</string>
</dict>
</plist>

```


### 分析
```
xcodebuild archive -workspace XXX.xcworkspace -scheme XXX -configuration Release -archivePath XXX CONFIGURATION_BUILD_DIR ./dir ODE_SIGN_IDENTITY=证书 PROVISIONING_PROFILE=描述文件UUID
```
| 文件         	 | 说明           
| -----------------   |:-------------
| -workspace XXX.xcworkspace      | XXX.xcworkspace需要编译工程的工作空间名称，如果工程不是.xcworkspace的，可以不需要-workspace XXX.xcworkspace这段话
| -scheme XXX        | XXX是工程名称，-scheme XXX是指定构建工程的名称     
|-configuration Release    | 填入打包的方式是Debug或Release，就跟在Xcode中编译前需要在Edit scheme的Build configuration中选择打出来的包是Debug还是Release包一样，-configuration就是配置编译的Build configuration     
| -archivePath XXX    | 配置生成.xcarchive的路径，
| ODE_SIGN_IDENTITY=证书         | 配置打包的指定证书，如果该工程的Xcode已经配置好了证书，那么不加入这段话也可以，打包出来的证书就是Xcode中配置好的。 
| PROVISIONING_PROFILE=描述文件UUID    | 配置打包的描述文件，同上，Xcode已经配置好了就不用在填入这段话了
| CONFIGURATION_BUILD_DIR    | 配置编译文件的输出路径，如果需要用到.xcarchive文件内部的dSYM等文件，可以使用改字段指定输出路径。 


### 问题一

<img src="https://ws3.sinaimg.cn/large/006tNbRwly1fx8uis0xxij30i003zq3c.jpg">

配置一下compileBicode=NO即可
<img src="https://ws4.sinaimg.cn/large/006tNbRwly1fx8ukd1di2j30du06edg8.jpg">

感谢[树下敲代码的超人](https://www.jianshu.com/p/4281908243a3)