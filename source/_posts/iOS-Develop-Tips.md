---
layout: post
title: iOS Develop Tips
draft: false
date: 2018-05-16 22:57:20
categories: iOS
tags: iOS,tips
permalink:
description:
cover_img: 
toc-disable:
comments:
---

![こよりん](http://assets.chaojita.cn/68759513_p0_master1200.png)

<center>配图来自 [こよりん](https://www.pixiv.net/member_illust.php?mode=manga&illust_id=68759513) ，已取得作者授权 ~</center>

记录工作生产过程中一些 iOS 相关基础知识点，本文将不断更新

<!--more-->

* 私有 **api** 混淆

```objective-c
/** Objective-C 版 */
//base64编码
- (NSString *)encodeString:(NSString *)string
{
    NSData *data = [string dataUsingEncoding:NSUTF8StringEncoding];
    NSString *encodedStr = [data base64EncodedStringWithOptions:0];
    return encodedStr;
}
//base64解码
- (NSString *)decodeString:(NSString *)string
{
    NSData *data = [[NSData alloc] initWithBase64EncodedString:string options:0];
    NSString *decodedStr = [[NSString alloc] initWithData:data encoding:NSUTF8StringEncoding];
    return decodedStr;
}
//调用私有api
- (void)testPrivateApi
{
    NSString *path = [self decodeString:@"L1N5c3RlbS9MaWJyYXJ5L1ByaXZhdGVGcmFtZXdvcmtzL01vYmlsZUNvbnRhaW5lck1hbmFnZXIuZnJhbWV3b3Jr"];
    NSBundle *container = [NSBundle bundleWithPath:path];
    if ([container load]) {
        Class appContainer = NSClassFromString([self decodeString:@"TUNNQXBwQ29udGFpbmVy"]);
        NSString *sel = [self decodeString:@"Y29udGFpbmVyV2l0aElkZW50aWZpZXI6ZXJyb3I6"];
        id test = [appContainer performSelector:NSSelectorFromString(sel) withObject:@"com.tencent.xin" withObject:nil];
        if (test) {
            NSLog(@"存在该应用");
        }
    } 
}
```

```swift
/** Swift 版 */
//base64编码
    func encode(_ string: String) -> String {
        let data: Data? = string.data(using: .utf8)
        let encodedStr = data?.base64EncodedString(options: [])
        return encodedStr ?? ""
    }
    
    //base64解码
    func decode(_ string: String) -> String {
        let data = Data(base64Encoded: string, options: [])
        let decodedStr = String(data: data ?? Data(), encoding: .utf8)
        return decodedStr ?? ""
    }
    
    //调用私有api
func testPrivateApi() {
    let path = decodeString("L1N5c3RlbS9MaWJyYXJ5L1ByaXZhdGVGcmFtZXdvcmtzL01vYmlsZUNvbnRhaW5lck1hbmFnZXIuZnJhbWV3b3Jr")
    let container = Bundle(path: path)
    if container?.load() ?? false {
        let appContainer: AnyClass = NSClassFromString(decodeString("TUNNQXBwQ29udGFpbmVy"))
        let sel = decodeString("Y29udGFpbmVyV2l0aElkZW50aWZpZXI6ZXJyb3I6")
        let test = appContainer.perform(NSSelectorFromString(sel), with: "com.tencent.xin", with: nil)
        if test {
            print("存在该应用")
        }
    }
}
```

* 跳转系统相册 私有 **Url Schemes ** `photos-redirect://**`

  需要进行混淆，不然在上传到 itunes 被拒

``` swift
let urlStr = "photos-redirect://"

            if let url = URL(string:urlStr) {
               if #available(iOS 10.0, *) {
                    UIApplication.shared.open(url, options: Dictionary(), completionHandler: nil)
                } else {
                    // Fallback on earlier versions
                    UIApplication.shared.openURL(url)
                }
                // 变为未保存状态
            }
```

* presentViewController延迟跳转, 或者点击 2 次才跳转

解决方案**:**

把presentViewController放在主线程中执行.

```objective-c
dispatch_async(dispatch_get_main_queue(), ^{
    [self presentViewController: viewController animated:YES completion: nil];
});
```

* **Swift **中的 **deinit** 方法

deinit属于析构函数

析构函数(destructor) 与构造函数相反，当对象结束其生命周期时（例如对象所在的函数已调用完毕），系统自动执行析构函数和OC中 dealloc一样的,通常在deinit和dealloc中需要执行的操作有:

1. 对象销毁
2. KVO移除
3. 移除通知
4. NSTimer销毁

* **cocoapods** 无法安装最新版本的库

执行 `pod repo update` 更新一下就可以了

* **swift4** 写入 **string** 数据到 **txt** 文件

```swift
// 拿到目录
let filePath:String = NSHomeDirectory() + "/Documents/fringeItems.txt"

// 写入  response为string类型数据
do {
      try response.write(toFile: filePath, atomically: true, encoding: String.Encoding.utf8)
} catch { }

// 读取
do{
     let data: String = try String.init(contentsOfFile: filePath)
} catch {}

// 判断该文件是否存在
var isExist:Bool = FileManager.default.fileExists(atPath: filePath)
```

* **swift4** 获取模拟器沙盒目录

``` swift
let paths = NSSearchPathForDirectoriesInDomains(FileManager.SearchPathDirectory.documentDirectory, FileManager.SearchPathDomainMask.userDomainMask, true)
```

* 重写 hitTest 方法，使点击透过 view

```swift
    override func hitTest(_ point: CGPoint, with event: UIEvent?) -> UIView? {
        let hitView = super.hitTest(point, with: event)
        if hitView == self {
            return nil
        }
        return hitView
    }
```

* 如果因为 cell 复用导致内容重复 

  一般不推荐使用

```
        let cell = collectionView.dequeueReusableCell(withReuseIdentifier: "GUIDEPAGEVIEWCELLID", for: indexPath) as! GuidePageViewCell
        cell.contentView.subviews.forEach { (view) in
            view.removeFromSuperview()
        }
```

* 通过脚本群发 iMessage 信息

```shell
set csvData to read "/Users/ennisk/Desktop/phone.csv"
set csvEntries to paragraphs of csvData

tell application "System Events"
	tell application "Messages" to activate
end tell

tell application "Messages"
	repeat with i from 1 to count csvEntries
		set phone to (csvEntries's item i)'s text
		set myid to get id of first service
		set theBuddy to buddy phone of service id myid
		send "test message" to theBuddy
	end repeat
end tell
```

其中 phone.csv 的手机号码格式为  86158****3823

且只有已发过一次iMessage消息的前提下才能发得出去消息

* 输出所有字体

```
- (void)allFont { 
	for (NSString *allFont in [UIFont familyNames]) { 
		NSLog(@"%@", allFont); 
		for (NSString *fontName in [UIFont fontNamesForFamilyName:allFont]) {
        	NSLog(@" %@", fontName); 
        } 
     }	
 }
```

