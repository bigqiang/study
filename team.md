# 成熟Web开发团队开发、测试、上线的环境和流程

## 1. 需要一个可以模拟线上的开发环境。

本地反向代理线上真实环境开发即可。（apache，nginx，nodejs均可实现）

## 2. 需要一个可以模拟线上的测试环境。

模拟线上的测试环境，其实就是你需要一台有真实数据的测试机么，我建议没条件搭daily的，就直接用线上数据测好了，只不过程序部分走你们的测试环境而已，有条件搭daily当然最好咯。

## 3. 需要一个可连调的测试环境。

可连调的测试环境，分为2种。一种是你们开发测试都在一个局域网段，直接绑hosts就完了，不在一个网段，就一人给一台虚拟的测试机，放在大家都可以访问到的公司内网，代码直接往上布即可。

## 4. 需要一个自动化的上线系统。

自动化的上线系统，如果你们运维不给你们做，我猜你们都是直接ftp往线上扔？那么你可以自己做一个简易的上线系统。原理不复杂，每次上线时都抽取最新的trunk或master，做一个tag，再打一个时间戳的标记，然后分发到cdn就行了。界面里就2个功能，打tag，回滚到某tag，部署【够简易了吧，而且是全自动的】。

5. 一个开发流程适合前后端的。

开发流程就是看项目了还有所用到的工具，构建，框架了。简单来说，原则就是分散独立开发，互相不干扰，连调时有hosts可绑即可。

回答了你的问题之后，我说下我自己的项目是怎么个开发流程。灰常简单，代码管理工具是svn，起新需求就起新分支，独立开发，开发完合并到trunk，trunk不做任何开发工作，只负责merge。上线有上线系统，你可以理解为我上面说的那个简易功能的加强版。我们是自带build的功能的。自己编写build脚本，ant，grunt随便了。做好连到发布系统，一键集成，本地只关心源码开发。本地环境，我拿nodejs写了一个自带rewrite，反向代理的server，超级仿真线上，一个hosts组管理的工具，一套适合自己部门的grunt插件库【就是很多很多grunt插件。。】。完全适合开发各种独立项目了。当然如果你的测试，文档都集成在build那一步，是最棒的了。协同合作我们是每个人开发都有一台自己的测试机，linux的，我本地也有工具可以完成自动build+push的功能。方便快捷。可能全看下来挺复杂，不过前端工程化确实就是这个样子。帮你脱离之前的手忙脚乱，专注于业务的开发。各位大神轻喷。我只是分享自己的经验。

就用atlassian 那一套不好吗？Jira 拿来管理 事件 的， 还有设置Agile 项目管理的。 Git 和 Stash 拿来 code review 和管理 branch 的， 所有的branch 都按照Jira 的事件编号来命名Bamboo 是做 auto-build 的，一旦 stash 提交了pull request， 然后review 通过了， merge 了，自动build，build的过程中可以增加测试。 Mock 的unit test, selenium 这种integrated 也行。每次build 自动运行 （实际上， pull request 的时候就会运行）Confluence 用来写文档，记录工作会议，写wiki 的。这一套不是挺好的吗？manager 就负责写 confluence 和 盯着 agile 来督促 各模块 （dev，QA， staging， production）就好了；QA负责写一点 unit test和selenium， 或者给大家买饭盒也行；剩下的程序员 就狂写代码，狂提交啊。





因为branch，profile和server不一样，不同的应用有不同的功能。大致功能是：
* 线下开发 automan-dev
* 线上开发 automan-dev
* 线上测试 automan-alpha
* 线上测试 automan-beta
* 线上运营 automan
* 个人调试

automanXXXX下面是branch，profile，server的含义。



没人提及maven和lein吗。


百度出的前端解决方案 Fis

---

# CI实践_Android持续集成
需要的准备知识：gitlab、Jenkins、各种plugins、shell等；
另外，推荐一个seafiles，相当于云存储网盘，大家可以把构建的apk包，发送至，供团队内部使用；
当然，你也可以采用ftp为team共享也可以。

## 一. 总体的全局配置：
配置相关plugin，如果需要进行代码检测的话，也需要安装Sonar，部分配置如下：
```
Sonar installations：
Name ：sonar
Server URL：http://192.168.0.100:9000/
Sonar account login：admin
Sonar account password：*****
Database URL:jdbc:mysql://localhost:3306/sonar?useUnicode=true&characterEncoding=utf8&rewriteBatchedStatements=true
Database login:root
Database password：****
```

邮件通知
```
SMTP服务器        smtp.exmail.qq.com
用户默认邮件后缀    @aituyou.com
```

## 二.单个Project中的配置：
触发构建器：
```
Build after other projects are built
Projects to watch：TSD-Common
```

源码管理：

Git Repositories： http://121.201.13.32:9000/xiaobao/tsd-navigation.git


Build periodically:日程表

\# H 9-17/2 * * 1-5

Pull SCM:

\# H 9-16/2 * * 1-5
```bash
Command Shell:
echo `pwd`
echo "buld start time：`date`"
echo '******************* Start build *************************'
##cd /var/lib/jenkins/;./env.sh
cd /var/lib/jenkins/jobs/TSD-Nav/workspace/
#### dos2unix files
dos2unix `find . -name "*.java"`

/opt/Android/adt-bundle-linux-x86_64-20140624/sdk/tools/android update project -t 3 -p . -n TSD-Nav
sed -i '$d' project.properties
sed -i '$d' project.properties
sed -i '$a target=android-19' project.properties
sed -i '$a android.library.reference.1=../../TSD-Common/workspace/' project.properties
#sed -i '$a android.library.reference.1=../../TSD-Thirdparty/workspace/qiniu-sdk/' project.properties

# change key
sed -i 's/r1Em7hxaGvUmbu3Te5Mne508/0TC6r2T1uTX9xgdUSyQuV5Lo/g' /var/lib/jenkins/jobs/TSD-Nav/workspace/AndroidManifest.xml

/opt/ANT/apache-ant-1.9.4/bin/ant clean
/opt/ANT/apache-ant-1.9.4/bin/ant debug
/opt/ANT/apache-ant-1.9.4/bin/ant release

# signer
#jarsigner -verbose -keystore /home/tuyou/HAO/TuYouDemoKeyStore -signedjar /var/lib/jenkins/jobs/TSD-Nav/workspace/bin/TSD-Nav-release-signed.apk /var/lib/jenkins/jobs/TSD-Nav/workspace/bin/TSD-Nav-release-unsigned.apk 'tuxiaobao' -storepass 'tuxiaobao'
jarsigner -verbose -sigalg MD5withRSA -digestalg SHA1 -keystore /home/tuyou/HAO/tuyou_android.keystore -signedjar /var/lib/jenkins/jobs/TSD-Nav/workspace/bin/TSD-Nav-release-signed.apk /var/lib/jenkins/jobs/TSD-Nav/workspace/bin/TSD-Nav-release-unsigned.apk tuyou_android.keystore -storepass 'tuxiaobao'

# get version number
#HAO = `cat AndroidManifest.xml | grep versionName= | awk -F "=" '{print $2}' |awk -F "\"" '{print $2}'`
#echo ${HAO}
#cd /var/lib/jenkins/jobs/TSD-Nav/workspace/bin
#mv TSD-Nav-debug.apk TSD-Nav-+${HAO}.apk
#ls -la
#cd /var/lib/jenkins/jobs/TSD-Nav/workspace/

#get date
# date -d now +%Y%m%d%H%M%S


cd /var/lib/jenkins/jobs/TSD-Nav/workspace/bin
#mv TSD-Nav-debug.apk TSD-Nav-`cat /var/lib/jenkins/jobs/TSD-Nav/workspace/AndroidManifest.xml | grep versionName= | awk -F "=" '{print $2}' |awk -F "\"" '{print $2}'`.apk
mv TSD-Nav-release-signed.apk TSD-Nav-Release-`cat /var/lib/jenkins/jobs/TSD-Nav/workspace/AndroidManifest.xml | grep versionName= | awk -F "=" '{print $2}' |awk -F "\"" '{print $2}'`-`date -d now +%Y%m%d%H%M%S`.apk
cd /var/lib/jenkins/jobs/TSD-Nav/workspace/


#FTP directory ：/var/ftp/anonymous/upload
cp /var/lib/jenkins/jobs/TSD-Nav/workspace/bin/TSD-Nav-Re*.apk /var/ftp/anonymous/upload/tsd-app-Release/Nav/

#copy to seafiles
cp /var/lib/jenkins/jobs/TSD-Nav/workspace/bin/TSD-Nav-Re*.apk /home/tuyou/Seafile/Seafile/发布/AutoBuild
echo '******************* Finish build *************************'
echo "buld end time：`date`"
```

其实没有过多解释，如果需要了解细节的朋友，可以联系我。


