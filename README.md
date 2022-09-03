# voiceId
- 借鉴CapsWriter修改的windows端语音输入
  - 最近一直在寻找win端的方便语音输入的工具，诸多尝试后，发现了[CapsWriter](https://github.com/HaujetZhao/CapsWriter)
- 进行了一些小修改，足够自己用了，同时想分享给大家
  - 可修改按键按住的有效时间阈值、可修改按键快捷键、可修改语音识别文本句末是否保留句号、可修改是否重置按键按之前状态
# 目前的痛点
- win10与win11自带的语音输入
  - 快捷键win+H即可使用。win11还对win10的语音输入进行了升级，确实能够开关语音了，win11端好用一点。但是仍有诸多问题，不能满足需要
  - 首先是网络经常出错。win10端好一点点，也常常出现失效；win11端伴随不同版本更新与bug问题，基本都会出问题。真受够了
  - 其次是操作逻辑头疼。win11端改进一点点，win+H能够开始停止了。常用的win10端，操作不方便，逻辑头疼，不适合。
  - 其次是识别与输入速度缓慢，响应差，即时性低
- 输入法方式
  - 如讯飞、腾讯、百度输入法等自带的输入法方式。仍然不尽如人意
  - 操作不舒畅。例如讯飞，必须手动或快捷键启动语音模块后，使用F键进行开关，不流畅不方便
  - 速度很慢。例如讯飞，基本上两秒以上出一个简单的词的结果
- 寻找工具
  - 由于win端不是互联网主战场，语音输入成了一个问题，相比于手机端差的很多。同时能够寻找的工具或软件也比较少
  - 在多方查找后，幸亏找到由**淳帅二代**同学做的CapsWriter软件，挺不错的
  - 原版相关链接：[github](https://github.com/HaujetZhao/CapsWriter)，[B站教程](https://www.bilibili.com/video/BV12A411p73r?vd_source=5be2e97716b42495ab98839ca452e5d1)，[思源点子](https://ld246.com/article/1594371212477)
# 简单的改进
- 花了一两天理解了这个软件的python代码，对软件第一版本（无交互页面版本）进行了简单的更改
- 首先是添加了四个可以通过ini文本设置的功能
  - 功能说明
    - 四个功能的接口，放在与exe文件同级的voiceSet.ini文件中。如果预先没有voieSet.ini文件，初次运行exe文件会生成，同时exe文件报错。浏览并填写确认好内容，再次运行exe文件
    - 第一行：按键阈值时间，单位秒
    - 第二行：语音识别快捷键，使用标准按键名。实在找不到可参考[keyboardName](https://github.com/boppreh/keyboard/blob/master/keyboard/_canonical_names.py#L1233)，进行搜索
    - 第三行：是否保留句末句号。默认True表示保留语音识别末尾句号
    - 第四行：是否重置按键状态。默认False表示不重置按键状态
  - 自定义阈值时间
    - 用来衡量按键是普通使用还是调用语音识别。长按时间超过阈值，则认为是调用语音识别，不超过则认为是普通按键
    - 时间设置如果较短，则容易被误触发，影响日常使用。如果时间设置足够久，相当于将功能进行失效，语音输入也不方便。自己感觉设置0.75秒挺适合自己的，这个根据体验自己设置
  - 自定义快捷键
    - 在voiceSet文件中写入要使用的快捷键，常用的如：Alt、Shift、Caps lock、Ctrl等
  - 句末句号是否保留
    - 语音识别默认会带有句号。如果设置False表示末尾不带句号，方便手动添加
  - 是否恢复操作前键盘状态四个功能
    - 标准键盘例如caps键控制大小写，shift控制输入法，使用长按后相当于也进行了一次按键。如果要恢复，就设置为True，会进行一次恢复按键，回到按键前状态
    - 同时例如Ctrl键，按一下对电脑没有影响，这样就无需进行重置，则设置为False即可
- 快速启动的思路更改
  - 在CapsWriter2.0版本，作者为加速识别过程，实现了在按下快捷键便立即进行录音，相较于第一版按下按键后要等待一小段时间才开始录音更快捷。
    - 作者的实现方式是在按下快捷键时，进行第一段录音并保存数据。如果仍然按下按键，继续进行语音输入，则将后续数据与第一段录音数据合并添加，并对数据进行发送
  - 更简单的快捷实现
    - 为了加速录音效果，在按下按键就能立即进行录音，同时能够利用**时间阈值**来判断按键操作是否在进行语音识别操作
    - 这里通过相似但简单一点的方式实现。在按下按键后，则直接进行录音与语音识别，此时其实已经接收到云端返回的消息数据。在松开按键后，对按下按键到松开按键的时间进行计时，如果差值时间（即按下键盘按键时间）超过了设定的阈值时间，说明此时在进行语音识别操作，则将整个过程的语音识别结果发送到文本框中。如果差值时间小于阈值时间，说明此时为一般的按键操作，软件不将语音识别结果发送到文本框
 # 软件内容
   - 软件共6个文件，核心的是exe文件：voiceId.exe。其他的可以不需要，需要的运行后报错自动生成，再次运行exe文件即可
   - 6个内容解读![image](https://user-images.githubusercontent.com/57529790/188251018-a8801985-972c-454c-96b4-8449a031703d.png)
     - alispeech.log
       - 阿里引擎运行消息文件，可以不用管
     - killVoiceId.cmd
       - 自己写的简单的关闭进程的cmd文件
       - 想要关闭运行的语音识别软件进程，点击运行一下就可以
       - 或者可以直接在任务管理器，进程中寻找到voiceId，点击关闭也可以2。![image](https://user-images.githubusercontent.com/57529790/188252104-60fe5dcd-677d-4e63-a93c-317dbb595961.png)
     - token.ini
       - 需要进行设置的文件，与阿里云引擎设置有关，保留了原作者的方式
       - 需要进行设置accesskeyid、accesskeysectet、appkey三个，前两个id与expiretime不用管![image](https://user-images.githubusercontent.com/57529790/188251276-2bba71fa-9d3b-49f9-8390-5c0cecf8abd4.png)

       - 具体操作可参考作者视频关于阿里云引擎的内容[B站教程](https://www.bilibili.com/video/BV12A411p73r?vd_source=5be2e97716b42495ab98839ca452e5d1)
     - voiceId.exe
       - 软件exe文件，双击即可运行
       - 利用pyinstaller进行的打包得到: pyinstaller -F -w -i voiceId.ico voiceId.py
     - voiceSet.ini
       - 软件四功能设置文件，可windows上(右键文件-编辑)进行编辑操作
     - voiceTest.txt
       - 没什么功能的测试文件，方便运行后打开文件进行语音输入测试
# 软件使用
   - 下载运行功能包，解压缩
   - 在token.ini文件中，写入三个阿里云引擎参数accesskeyid、accesskeysectet、appkey
   - 在voiceSet.ini文件，根据自己需要更正
   - 运行voiceId.exe即可，进行简单测试
   - 使用killVoiceId.cmd进行关闭，或者在任务管理器中进行关闭（修改参数后，需要关闭进程再开启才能生效）
# 其他想法
  - 语音引擎
    - 阿里云与讯飞引擎
      - 本次工作尝试了讯飞引擎和阿里云引擎，阿里云引擎速度确实更快一些
    - api方式与sdk方式
      - 一开始想要利用Quicer进行进行与讯飞和阿里云引擎的api方式通讯，利用Quicer的hppt功能，通过post方式，获取内容，进行处理。但是暂时找不到仅仅通过quicer进行按键监听、响应的方法。如果利用Quicer对python文件进行打开，需要电脑安装pythyon和相关环境，实现也不方便不太灵活，就放弃quicer了
    - 使用中，感觉使用sdk方式，要比使用api方式响应速度快
  - 按键相关
    - 我在quicker中将caps映射为ctrl+space，以此实现caps键切换中英文，比shift方便好用。但是就无法用caps来语音识别了，会冲突
    - 日常使用的按键，进行语音识别多少会有些影响，shift键在利用shift多选时也会有些影响。暂时还没想怎么可以更优化，不影响日常使用，暂时可能就是调整好按键阈值时间来权衡吧
  - 源码使用
    - voiceId-src.zip中放置了实现的源代码，主要是voiceId.py，是对原作者进行了小小的修改而成
    - 如果要自己修改，参考作者的文档进行python安装与环境安装，使用pyinstaller打包
