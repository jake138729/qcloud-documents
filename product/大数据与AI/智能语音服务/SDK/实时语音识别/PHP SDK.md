

## 功能简介
语音识别（ASR）可以把音频数据转换为文本，需要持续对音频进行识别的场景，推荐使用实时语音识别，例如视频录制时候的实时字幕，语音对话机器人等。  

- 语言和方言：目前语音识别服务支持的主语言为中文普通话，可以识别有一定方言口音的普通话，支持在普通话中掺杂少量英文字母和单词。  
- 采样率和位深度：支持16bit、8k或者16k采样率的单声道或双声道的中文音频识别。
- 我们建议每300或者500毫秒发送一次音频，对此，客户端需要做一些必要的缓存逻辑。
- VAD（Voice Activity Detection）指对语音进行分段的技术，是算法通过对语音之间的停顿进行检测，判断用户说话间的的分句。
- voice_id 用于识别单次对话请求。如果用户持续说话一段时间，包含了很多句话，可以采用一个 voice_id 发送一系列的语音数据，seq 字段表示序号，从0开始。voice_id 不能重复，否则会导致识别错误。

例如，用户说：“今天天气好。”，大概2到3秒的时间。假设1秒发3个请求，则共计会发送8个左右的请求。服务器会返回相应个回包。类似于：
```
{"code":0,"message":"成功","voice_id":"3b53e9b909b55929","seq":0,"text":"今"}
{"code":0,"message":"成功","voice_id":"3b53e9b909b55929","seq":1,"text":"今天"}
{"code":0,"message":"成功","voice_id":"3b53e9b909b55929","seq":2,"text":"今天"}
{"code":0,"message":"成功","voice_id":"3b53e9b909b55929","seq":3,"text":"今天天气"}
{"code":0,"message":"成功","voice_id":"3b53e9b909b55929","seq":4,"text":"今天天气好"}
{"code":0,"message":"成功","voice_id":"3b53e9b909b55929","seq":5,"text":"今天天气好。"}
```

## 开发环境
**环境依赖**
此版本 SDK 适用于 PHP5.4.16 及以上版本。

**下载：SDK**
实时语音识别 PHP SDK [下载地址](https://main.qcloudimg.com/raw/a4c791c4ee4a9585070ca52cce091833/php_realtime_asr_sdk.tar.gz)。
**安装 SDK**
**源码安装**。
根据下载地址下载源码，将源码中的 * .php 复制到项目中即可使用。

**卸载 SDK**
卸载方式即删除 * .php 即可。

## 获取用户信息
**获取用户鉴权信息及申请使用**
- 使用本接口之前需要先 [注册](https://cloud.tencent.com/register) 腾讯云账号，获得 AppID，SecretID 及 SecretKey。 并在 [语音识别](https://cloud.tencent.com/product/asr) 页面单击【立即使用】。
- 关于云 API 账号中的 AppID、SecretId 与 SecretKey 查询方法，可参考 [鉴名签权](https://cloud.tencent.com/document/product/441/6203)。 
- 具体路径为：单击 [腾讯云控制台](https://cloud.tencent.com/login?s_url=https%3A%2F%2Fconsole.cloud.tencent.com%2F) 右上角您的账号，选择【访问管理】>【访问密钥】>【API 密钥管理】界面查看 AppID 和 key。

**配置用户信息**
**将 AppID、SecretId、SecretKey 配置到 SDK 中。**
将查询到的用户信息更改到 ```php\_realtime\_asr\_sdk/Config.php```中。

```
#需要配置成用户账号信息
static $SECRET_ID = "AKID********************************";
static $SECRET_KEY = "kKm2*******************************";
static $APPID = 1255*********;
```

## 开发相关
**请求参数**

| 参数名称 | 必选 | 类型 | 描述 |  
| --- | --- | --- | --- |
| appid |  是 | Int | 用户在腾讯云注册账号的 AppId，具体可以参考 [获取用户信息](https://cloud.tencent.com/document/product/441/19635?!editLang=zh&!preview#.E8.8E.B7.E5.8F.96.E7.94.A8.E6.88.B7.E4.BF.A1.E6.81.AF)。 |
| secretid | 是 | String | 用户在腾讯云注册账号 AppId 对应的 SecretId，获取方法同上。 |
| sub\_service\_type | 否 | Int | 子服务类型。1：实时流式识别。|
| engine\_model\_type | 否 | String | 引擎类型引擎模型类型。8k_0:8k 通用，16k_0:16k 通用，16k_en:16k英文。|
| result\_text\_format | 否 | Int | 识别结果文本编码方式。0：UTF-8；1：GB2312；2：GBK；3：BIG5。|
| res_type | 否 | Int | 结果返回方式。1：同步返回；0：尾包返回。|
| voice_format | 否 | Int | 语音编码方式，可选，默认值为 4。1：wav（pcm）；4：speex（sp）；6：silk；8：mp3（仅16k_0模型支持）。|
| seq | 是 | Int | 	语音分片的序号从0开始。|
| end | 是 | Int | 是否为最后一片，最后一片语音片为1，其余为0。 |
| source | 是 | Int | 设置为0。 |
| voice_id | 是 | String | 16位 String 串作为每个音频的唯一标识，用户自己生成。|
| timestamp | 是 | Int | 当前 UNIX 时间戳，可记录发起 API 请求的时间。如果与当前时间相差过大，会引起签名过期错误。SDK会自动赋值当前时间戳。|
| expired | 是 | Int | 签名的有效期，是一个符合 UNIX Epoch 时间戳规范的数值，单位为秒；Expired 必须大于 Timestamp 且 Expired-Timestamp 小于90天。SDK 默认设置1小时。|
| timeout | 是 | Int | 设置超时时间单位为毫秒。|
| nonce | 是 | Int | 随机正整数。用户需自行生成，最长10位。|

**请求url参数示例**

```
https://aai.qcloud.com/asr/v1/125000001?
end=0&
engine_model_type=16k_0&
expired=1558016577&
nonce=434303218&
res_type=0&
result_text_format=0&
secretid=XXXXXXXXXXXXXXXXXXXXXXX&
seq=0&
source=0&
sub_service_type=1&
timeout=5000&
timestamp=1558010577&
voice_format=1&
voice_id=82510017d7deb33e
```
其中v1表示 API 的版本，v1.0，后面125000001是 AppID，各个参数的说明参考上表。

## PHP 快速入门示例
参考 php\_realtime\_asr\_sdk/RasrClient.php

```
<?php
require ('RASRsdk_v1.1.php');

# 1. 先修改好Config.php文件中的配置

# 2. 然后开始调用：
//调用 RASRsdk 中的 sendvoice 函数获得识别结果
$filepath = "test_wav/8k/8k.wav";
$result = sendvoice($filepath, false);
echo "<br>8K Result is: ".$result;


# ---------------------------------------------------------------------
# 3. 若需中途调整参数值，可直接修改，然后继续发请求即可。比如：
Config::$ENGINE_MODEL_TYPE = "16k_0";

$filepath = "test_wav/16k/16k.wav";
$result = sendvoice($filepath, false);
echo "<br>16K Result is: ".$result;

?>
```












