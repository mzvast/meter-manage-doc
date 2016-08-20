# 协议格式特殊说明 #
1. 协议使用JSON格式，传送文件除外。
    - 客户端与浏览器交换JSON信息时：
        - **发送**：使用类似`JSON.stringify(msg)`的方法将对象转换成字符串
        - **接收**：使用类似`JSON.parse(string)`的方法将字符串转换成对象
2. 收到协议帧后，回复格式如下：
	-  协议内容正确，除明确说明外无需额外回复。
	-  协议内容错误，回复:
    	<pre>{
            "type":"illegal",
            "reason": "错误原因"
    	}</pre>


# 开发人员demo特殊说明(更新V25.0版本) #
## URL
[http://115.159.143.179:8000/beian/compare](http://115.159.143.179:8000/beian/compare)
## RAP

rap是我们团队前后端接口用的mock平台，由于最近阿里服务器每天崩溃，所以demo站暂时关闭了rap。所以导致一些需要获取rap数据的页面没有内容，这是正常现象，不影响比对。  
接下来介绍如何快速使用这个demo。

- 直接进入demo URL，展示的是验核情况，默认情况下由于没有配置参数，所以“验核失败，不能比对”。
- 点击“产生模拟数据”，此时系统模拟所有数据（除了文件hex内容）,请手动选中本机文件。
- 验核通过后点击“开始比对”，进入“开发人员模式”
- 前4项功能可以读取和设定比对模式。请F12打开浏览器控制台查看输出！
    - 读取模式：读取当前比对模式。
    - 单步运行，模式0：默认调试模式。每次只会执行一条动作，并且在时间线上展示结果。结束后不跳转到比对报告，如需查看需要手动跳转。
    - 备案比对，模,1：除了备案号不发送，其他都顺序执行。结束后跳转到比对报告。根据成功与否，生成不同的报告。
    - 供货比对，模,2：所有都发送，顺序执行。结束后跳转到比对报告。根据成功与否，生成不同的报告。
- 建议按顺序测试功能。其中第0条是测试浏览器的通知系统，可以无视。正式的比对流程是从1开始到最后。
- 每一步的反馈信息，都会在时间线上显示。点击清除时间线按钮，可以清除时间线信息。

## 浏览器单元测试
- 本模块功能点全部经过mock测试，以及服务器真机调试。
## 客户端Demo
- 支持所有类型的消息收发和自动回复。
- 不再需要使用Arraybuffer.js，全功能集成在index.js中
- PDF生成支持。由于性能原因，从浏览器转移到服务端！
## 错误排查
- 如果没有正确的反馈信息，建议检查：
    - 端口是否正确
    - 是否严格按照JSON字符串化发送信息

----------

**以下是协议内容（！注意：按照比对流程的时序撰写本文档）**
# 建立连接 #

- 发起连接（浏览器）
    - 连接类型:WebSocket
    - URL:`'ws://localhost:3456'`
- 发送握手信息（浏览器）。其中的`mode`提供额外信息，用于告知当前比对模式，客户端可针对不同模式优化测试流程。
    <pre>{
        "type":"hello"        
        "mode":0//取值0=调试模式,1=备案比对,2=供货比对
	}</pre>
- 成功返回信息（客户端）
    <pre>{
        "type":"welcome",
        "state":"success"
	}</pre>
- 失败返回信息（客户端）如果客户端自检发现问题在此阶段发出警告。
    <pre>{
        "type":"welcome",
        "state":"fail",
        "reason":"自检故障，不能测试，请联系管理员"
	}</pre>
#  比对信息 #

## 发送（浏览器）    （V25.1更新）
- 发送内容
    <pre>
    {
      "type": "info",
      "data": {
        "file_info": [{
          "cpu_id": 1          
        }, {
          "cpu_id": 2
        }],
        "cpu_info": [{
          "cpu_id": 1,
          "memory_addr": {
            "start": "4000",
            "end": "13fff"
          },
          "code_addr": {
            "start": "4000",
            "end": "97ff"
          },
          "protect_addr": [{
            "start": "12000",
            "end": "121ff"
          }, {
            "start": "13000",
            "end": "133ff"
          }],
          "reserve_addr": [{
            "start": "12000",
            "end": "121ff"
          }, {
            "start": "13000",
            "end": "133ff"
          }]
        }, {
          "cpu_id": 2,
          "memory_addr": {
            "start": "4000",
            "end": "13fff"
          },
          "code_addr": {
            "start": "4000",
            "end": "97ff"
          },
          "protect_addr": [{
            "start": "12000",
            "end": "121ff"
          }, {
            "start": "13000",
            "end": "133ff"
          }],
          "reserve_addr": [{
            "start": "12000",
            "end": "121ff"
          }, {
            "start": "13000",
            "end": "133ff"
          }]
        }],
        "meter_info": [{
          "bit": 1,
          "type": "single_phase",
          "costcontrol_type": "em_esam",
          "num": "xxxxxxxxxxxx",
          "addr": "xxxxxxxxxxxx",
          "vol": 220,
          "key_index": "04"
        }, {
          "bit": 2,
          "type": "single_phase",
          "costcontrol_type": "em_esam",
          "num": "xxxxxxxxxxxx",
          "addr": "xxxxxxxxxxxx",
          "vol": 220,
          "key_index": "04"
        }]
      }
    
    </pre>

包含内容说明：

- 文件信息
	- cpu_id
	- hex文件md5校验值

- cpu信息
	- cpu_id，
	- 存储器起始地址，
	- 代码段起始地址，
	- 保护区起始地址（2个）
	- 保留区起始地址（2个）

- 电表信息    
    - 表位（bit）：1~8
    - 表类型（type）：
        - "single\_phase" (单相电表）
        - "three\_phase"  (三相电表)
    - 费控类型（costcontrol_type）：
        - em_esam——嵌ESAM表，
        - ordinary——普通表
    - 表号（num）：xxxxxxxxxxxx——12位数字
    - 表地址（addr）：xxxxxxxxxxxx——12位数字
    - 额定电压（vol）：此处为220
    - 比对密钥索引（key_index）：默认为04
##  回复（客户端）
- 若正确回复:
    <pre>{
        "type": "info",
        "state": "success"
    }</pre>
- 若错误回复:
    <pre>{
        "type": "info",
        "state": "fail",
        "reason":"md5值长度不对"
    }</pre>
可能的错误原因有：  
    - 包含非法字符；
    - 地址信息不对，
    - 或者有重复等；
    - 表号和表地址长度不对;
    - 表位超出范围等。

#  发送HEX文件(V25.1更新)
##  发送文件信息（浏览器）
   <pre>{
    "type": "file",
    "extname":"hex",
    "md5": "d9fc6d737aea3345f681f24c8a2bb07c"
}</pre>
##  回复（客户端）
如果客户端支持该格式，并且后缀名和md5信息完整，则回复：
   <pre>{
    "type": "file",
    "state":"ready"
}</pre>
否则，回复：
   <pre>{
    "type": "file",
    "state":"fail",
    "reason":"extname undefine/md5 undefine/unknown extname（错误原因的表示，仅供参考，可以自己定，中文亦可）"
}</pre>
收到该信息浏览器会终止测试，让用户检查文件输入是否正确。
##  发送（浏览器）
使用二进制流发送文件，对于多CPU电表则分多次发送hex文件，每次只能发送一个文件。

## 回复（客户端）
收到文件后，进行md5检验，  

- (单文件跳过此步骤)多文件情况下，每当接收完一个文件，回复
    <pre>{
        "type": "file",
        "state": "next"
    }</pre>
- 接收完最后一个文件，回复
    <pre>{
        "type": "file",
        "state": "success"
    }</pre>

- 若错误回复
    <pre>{
        "type": "file",
        "state": "fail",
        "reason":"错误原因"
    }</pre>

# 发送备案号（供货比对专用）
##  发送（浏览器）
- 发送内容：
    <pre>{
        "type": "record_num",
        "value": "xxxxxxxxxxxxxxxx"//备案号为一16位数字    
    }</pre>

##  回复（客户端）
客户端收到备案号后，从电表处读出固化备案号，进行比对  

- 正确回复
    <pre>{
        "type": "record_num",
        "state": "success"
    }</pre>
- 错误回复
    <pre>{
        "type": "record_num",
        "state": "fail",
        "reason": "错误原因"
    }</pre>
#  开始比对命令
##  发送（浏览器）
- 发送内容
    <pre>{
      "type": "start_compare",
      "bit":[1,2]//一次可对单个表或者多个表进行比对。
    }</pre>
##  回复（客户端）
- 若正确返回
    <pre>{
      "type": "start_compare",
      "state": "success"
    }</pre>
- 若错误则返回
    <pre>{
        "type": "start_compare",
        "state": " fail ",
        "reason": "错误原因"
    }</pre>

#  比对结果

##  发送（客户端）
- 比对成功，发送内容：
    <pre>{
      "type": "compare_result",
      "state": "success"  
    }</pre>
- 比对失败，发送内容：
    <pre>{
      "type": "compare_result",
      "state": "fail",
      "reason":"通信不通，电源异常",//异常信息、失败原因
      "result": {
        "pass": [1,2], //通过的表位号
        "fail": [3,4] //失败的表位号
      }
    }</pre>


----------
版本号 | 说明  
:----|:--------
V25.1|修改文件部分协议
V25.0|HEX文件做完。流程优化。
V24.0|Arg增加localStrorage记忆
V24.0|HEX文件上传完成。
V23.0|针对IE无法下载pdf的问题修复
V22.0|性能原因从浏览器渲染PDF转移到服务端渲染。
V21.0|三种模式分离，可以很容易把开发者模式移除掉做准备。
V20.0|增加单步运行模式下PDF报告的生成（浏览器端）。
V19.0|根据不同模式和结果生成不同比对报告
V18.0|增加测试完成自动跳转到测试结果
V17.0|增加pdf生成功能，支持中文，字体雅黑
V16.0|demo server用例升级到最新版本，修复bug，可提供完整体验
V15.0|增加工作流自动切换
V14.0|增加模式切换
V13.0|增加模式定义
V12.0|demo增加备案号
V11.0|握手部分增加模式信息
V10.0|完善握手部分自检协议
V9.0|完善开发人员说明包括调试等 
V8.0|增加开发人员模式特殊说明  
V7.0|增加协议格式特殊说明
V6.0|增加欢迎握手信息
V5.0|使用markdown生成
V4.0|全面重构
V3.0|增加对话上下文内容 
V2.0|修复N多错误 
V1.3|表信息中增加费控类型
V1.2|协议增加供货比对时发送备案号
V1.1|修订一个拼写错误  

