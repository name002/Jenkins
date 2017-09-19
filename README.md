#搭建Jenkins遇到问题
##权限问题
在Mac环境下，我们需要先安装JDK，然后在Jenkins的[官网](https://jenkins.io)下载最新的war包。

![D3B7AE8B-E56B-4407-97E9-7E6E356BCD4F.png](http://upload-images.jianshu.io/upload_images/2061411-d9b2a7dc5e747c6c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/900)

![E212331B-5960-467C-AE88-7C2B8CF87191.png](http://upload-images.jianshu.io/upload_images/2061411-79c707d16a645386.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/900)

直接点击xxxx.war下载，不要点击三角选择Mac OS x，不要下载.dmg，因为dmg安装后会专门生成一个jenkins用户，导致要是不在这个用户下边就会出现权限问题，考虑到想让登录到这台mac的所有用户都可以操作jenkins，所以不要安装dmg。

#iOS构建
```
#!/bin/bash -lx
export LC_ALL="en_US.UTF-8"

cd path项目文件夹

pod update --verbose --no-repo-update

security unlock-keychain -p "密码" /Library/Keychains/System.keychain

#改签名文件设置
sed -i '' 's/ProvisioningStyle = Automatic;/ProvisioningStyle = Manual;/' xx.xcodeproj/project.pbxproj

#sed -i '' 's/"CODE_SIGN_IDENTITY[sdk=iphoneos*]" = "iPhone Developer";/"CODE_SIGN_IDENTITY[sdk=iphoneos*]" = "iPhone Developer";/' xx.xcodeproj/project.pbxproj
#sed -i '' 's/DEVELOPMENT_TEAM = xx;/DEVELOPMENT_TEAM = "iPhone Developer: xx (xx)";/' xx.xcodeproj/project.pbxproj
#sed -i "" s/'PROVISIONING_PROFILE = "";'/'PROVISIONING_PROFILE = "xx-xx-xx-xx-xx";'/g xx.xcodeproj/project.pbxproj 
#sed -i '' 's/PROVISIONING_PROFILE_SPECIFIER = "";/PROVISIONING_PROFILE_SPECIFIER = xx_DEV;/' xx.xcodeproj/project.pbxproj

mkdir ${WORKSPACE}/builds
if [ -d "${WORKSPACE}/builds/xcarchive" ]; then rm -rf ${WORKSPACE}/builds/xcarchive; fi;
if [ -d "${WORKSPACE}/builds/ipa" ]; then rm -rf ${WORKSPACE}/builds/ipa; fi;
mkdir ${WORKSPACE}/builds/xcarchive
mkdir ${WORKSPACE}/builds/ipa

nowdate=`date "+%Y-%m-%d"`
app_version=$(/usr/libexec/PlistBuddy -c "Print :CFBundleShortVersionString" "${WORKSPACE}/xx/Info.plist")
packinfo=$app_version'_'$nowdate'_'${BUILD_NUMBER}

sed -i '' '/CFBundleDevelopmentRegion/i\'$'\n<key>PackVersion</key><string>'"$packinfo"'</string>'$'\n' xx/Info.plist

# Archive the application
xcodebuild \
-workspace "xx.xcworkspace" \
-scheme xx \
-sdk iphoneos \
-archivePath "${WORKSPACE}/builds/xcarchive/xx.xcarchive" \
-configuration Debug \
PROVISIONING_PROFILE="xx-xx-xx-xx-xx" \
archive 

# Export the archive to an ipa
xcodebuild \
-exportArchive \
-archivePath "${WORKSPACE}/builds/xcarchive/xx.xcarchive" \
-exportOptionsPlist "${WORKSPACE}/builds/Enterprise.plist" \
-exportPath ${WORKSPACE}/builds/ipa/

cd builds/ipa/
cp -f xx.ipa /Users/admin/Documents/apacheforipa/xx_last.ipa
mv xx.ipa ${JOB_NAME}_$app_version'_'$nowdate'_'${BUILD_NUMBER}.ipa
cp -f /Users/admin/Documents/apacheforipa/xx.png ${WORKSPACE}/builds/ipa
```

[Enterprise.plist文件](https://github.com/name002/Jenkins/blob/master/Enterprise.plist)

##改BundleId
```
/usr/libexec/PlistBuddy -c "Set :CFBundleIdentifier com.xx.xxInhouse" ${WORKSPACE}/xxchant/xxchant/xxchant-Info.plist
sed -i "" "s/com.xx.xxchant/com.xx.xxchantInhouse/g" ${WORKSPACE}/xxchant/xxchant.xcodeproj/project.pbxproj
```

##为新改的BundleId匹配证书
```
sed -i '' 's/DevelopmentTeam = xx;/DevelopmentTeam = xx;/' ${WORKSPACE}/xxchant/xxchant.xcodeproj/project.pbxproj
sed -i '' 's/com.apple.Push = {\nenabled = 1;\n};/com.apple.Push = {\nenabled = 0;\n};/' ${WORKSPACE}/xxchant/xxchant.xcodeproj/project.pbxproj
sed -i '' 's/"CODE_SIGN_IDENTITY[sdk=iphoneos*]" = "iPhone Developer: xx xx (xx)";/"CODE_SIGN_IDENTITY[sdk=iphoneos*]" = "iPhone Distribution";/' ${WORKSPACE}/xxchant/xxchant.xcodeproj/project.pbxproj
sed -i '' 's/DEVELOPMENT_TEAM = LY7Wxxx;/DEVELOPMENT_TEAM = XTYSxxx;/' ${WORKSPACE}/xxchant/xxchant.xcodeproj/project.pbxproj
sed -i '' 's/PRODUCT_BUNDLE_IDENTIFIER = com.xx.xxchant;/PRODUCT_BUNDLE_IDENTIFIER = com.xx.xxchantInhouse;/' ${WORKSPACE}/xxchant/xxchant.xcodeproj/project.pbxproj
sed -i '' 's/PROVISIONING_PROFILE = "xx-xx-xx-xx-xx8a11";/PROVISIONING_PROFILE = "xx-xx-xx-xx-xx88f7";/' ${WORKSPACE}/xxchant/xxchant.xcodeproj/project.pbxproj
sed -i '' 's/PROVISIONING_PROFILE_SPECIFIER = xxchant_dev;/PROVISIONING_PROFILE_SPECIFIER = xxchant_inhouse;/' ${WORKSPACE}/xxchant/xxchant.xcodeproj/project.pbxproj
sed -i '' 's/com.xx.xxchant/com.elong.xxchantInhouse/' ${WORKSPACE}/xxchant/KeychainAccessGroups.plist
```

#发送带附件的邮件
构建后操作中设置Editable Email Notification中的Attachments可以带附件，把所有要带的附件都放这个路径下
![26442817-7690-410E-8657-09A08410197B.png](http://upload-images.jianshu.io/upload_images/2061411-d004edceb02d6c8f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#ftp传文件到服务器
系统全局设置插件Publish over FTP
![0D743547-8CA1-41AA-97BA-1AC16DBEEFD4.png](http://upload-images.jianshu.io/upload_images/2061411-b158cc93bb22a450.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![D9FFE6E6-CC13-4809-BBAF-1E53E63145B0.png](http://upload-images.jianshu.io/upload_images/2061411-18cfd58b931f1a60.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#Jenkins免登陆到服务器
scp /Users/admin/.ssh/id_rsa.pub xx@192.168.233.161:/home/tester/
mv id_rsa.pub ~/.ssh/authorized_keys
#Jenkins上传文件到服务器
scp /Users/xx/Downloads/xx.txt xx@192.168.233.161:/home/tester/

#Android打包关键设置
build.gradle
![F7D29CD4-A82A-4DEF-816B-16C8739EB8C0.png](http://upload-images.jianshu.io/upload_images/2061411-ddd91fa5dc0f4bb3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
buildtype设置

![2061411-68d1578ee7fcd92e.png.jpeg](http://upload-images.jianshu.io/upload_images/2061411-a790ebe06baac165.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![7362DF84-68FB-484C-85DC-C0862E740391.png](http://upload-images.jianshu.io/upload_images/2061411-f65b8cb6c083ad3a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#构建
```
#!/bin/bash -lx

export LC_ALL="en_US.UTF-8"
echo "==Clear outputs ==========${BUILD_NUMBER}"
rm -fr /Users/admin/.jenkins/workspace/xx_Android/app/build/
```
![6691A373-B7DE-46C2-947A-8BD4C3E56ACC.png](http://upload-images.jianshu.io/upload_images/2061411-d85a08e438f20ff7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
#!/bin/bash -lx

export LC_ALL="en_US.UTF-8"

cd /Users/admin/.jenkins/workspace/xx_Android/app/build/outputs/apk/

echo "QR============${BUILD_NUMBER}"

for file in `ls xx_android*.apk`; do /usr/local/bin/myqr http://192.168.14.246:8085/download/xx_app/xx_android/$file -p /Users/admin/.jenkins/workspace/xx_Android/app/src/main/res/mipmap-xxhdpi/ic_launcher.png -c -v 1 -d /Users/admin/.jenkins/workspace/xx_Android/app/build/outputs/apk/;done;
```