# ClashX

## 配置不走魔法的地址

在`~/.config/clash`地址下添加一个文件`proxyIgnoreList.plist`

在其中添加如下内容:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
  <array>
    <string>192.168.0.0/16</string>
    <string>10.0.0.0/8</string>
    <string>172.16.0.0/12</string>
    <string>127.0.0.1</string>
    <string>localhost</string>
    <string>*.local</string>
    <string>*.crashlytics.com</string>
    <!-- 上面的不能删掉 -->
    <!-- 下面的则为屏蔽的后缀 -->
    <string>*.baidu.com</string>
    <string>*.bilibili.com</string>
    <!-- qt -->
    <string>*.flashhold.com</string>
    <string>*.quicktron.com</string>
  </array>
</plist>
```

