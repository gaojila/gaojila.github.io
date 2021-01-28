# 优联k230使用usb2usb


# 场景

老是心水优联超强的续航，寻思整一个优联主控的键盘，现是面上有 k230、k270、k375s 主控，  
在这里我选择了 k230 主控，毕竟我对蓝牙没有需求，在这里就不演示如何组装一把优联机械键  
键盘了，演示下 Linux 下如何对 u2u 刷固件，毕竟优联的 fn 键很弱鸡。

# u2u 介绍

u2u 是[github](https://github.com/tmk/USB2USB_Converter)上的一个开源项目，是对普通键盘键位更改，实现多层和宏。

# u2u 如何刷固件

在这里我们需要使用以下 2 个网站[kle](kle.ydkb.io) 和[tkg](ydkb.tkg.io) kle 是用来生成键盘配列，tkg 使用来时生成固件和刷写固件的，将你的键盘配列在 kle 上画好，说下各个层的作用，原始层必须和你的优联键盘键位一致(我改的 84 键盘所以配列就是网站上的凯酷 84 改的)，第 0 层是你接上电脑时显示的那一层，第 1 层是你定义的 fn1 的第一层，第 2 层依次类推。

## 使用 kle 修改配列

- 优联 84 原始层
  [k230](https://gist.github.com/gaojila/f4f4dcfb53406d1963a5e367ab0574b3)
  ![](https://raw.githubusercontent.com/gaojila/images/master/%E4%BC%98%E8%81%94k230%E4%BD%BF%E7%94%A8usb2usb/20190927-094622.png)

- 优联 84 第 0 层
  [k2300](https://gist.github.com/gaojila/01bc5891934117a3b935b194780ba7ad)
  ![](https://raw.githubusercontent.com/gaojila/images/master/%E4%BC%98%E8%81%94k230%E4%BD%BF%E7%94%A8usb2usb/20190927-095515.png)

- 优联 84 第 1 层
  [k2301](https://gist.github.com/gaojila/16da8f918482343ce4a4ef831ee2ed5a)
  ![](https://raw.githubusercontent.com/gaojila/images/master/%E4%BC%98%E8%81%94k230%E4%BD%BF%E7%94%A8usb2usb/20190927-100129.png)

## 使用 tkg 刷固件

- 将原始层填入设定中
  ![](https://raw.githubusercontent.com/gaojila/images/master/%E4%BC%98%E8%81%94k230%E4%BD%BF%E7%94%A8usb2usb/20190927-100421.png)
- 将第 0 层和第一层填入层中
  ![](https://raw.githubusercontent.com/gaojila/images/master/%E4%BC%98%E8%81%94k230%E4%BD%BF%E7%94%A8usb2usb/20190927-100726.png)
- fn 设定
  优联 84 的原始 fn 键是多媒体按键，在这里我设置 fn0 给原 fn 并不做任何操作，设置 space 键为 fn1 键并保留空格键功能。
  ![](https://raw.githubusercontent.com/gaojila/images/master/%E4%BC%98%E8%81%94k230%E4%BD%BF%E7%94%A8usb2usb/20190927-101824.png)
- chrome 安装 TKG Chrome App 插件
  ![](https://raw.githubusercontent.com/gaojila/images/master/%E4%BC%98%E8%81%94k230%E4%BD%BF%E7%94%A8usb2usb/20190927-102019.png)
- 听说 windows 下要安全驱动？？？？linux 不存在的(有 avr 编程环境就行)
  ![](https://raw.githubusercontent.com/gaojila/images/master/%E4%BC%98%E8%81%94k230%E4%BD%BF%E7%94%A8usb2usb/20190927-102327.png)
- 按下刷机键就开始烧写吧！
  ![](https://raw.githubusercontent.com/gaojila/images/master/%E4%BC%98%E8%81%94k230%E4%BD%BF%E7%94%A8usb2usb/20190927-102459.png)
- [当然你也可以使用 avrdude 刷](https://electronics.stackexchange.com/questions/67121/how-do-i-upload-a-hex-file-firmware-to-a-target-board-without-using-the-arduino)

