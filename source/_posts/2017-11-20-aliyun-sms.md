---
title: laravel5接入阿里云短信验证码服务
date: 2017-11-20 19:05:42
category: Learn
toc: true
tags: 
    - laravel
---

### 阿里云开通短信服务

要申请两个东西

* 短信签名
* 短信模板

#### 申请短信签名

![](1.png)

申请完，审核时间一般两个小时以内，挺快的。

![](2.png)

审核状态变为通过，即为审核成功。

#### 申请短信模板

![](3.png)

里面的 ${code} 在之后调研api的时候是需要传进去的。

![](4.png)

模板里的code也是调用api的时候要使用到的。

#### 创建用户

调用短信服务的api，还需要两个参数

* AccessKeyId
* AccessKeySecret

因此还需要去访问控制里创建一个用户，并分配 **AliyunDysmsFullAccess 管理短信服务(SMS)的权限**

![](5.png)

创建好的时候记得保存 **AccessKeySecret**，之后就没机会再查看了。

### 整合到Laravel框架

#### 手动整合

👉👉👉 [SDK及DEMO下载](https://help.aliyun.com/document_detail/55359.html?spm=5176.doc55491.2.8.evWxXE)

> 之前尝试过用python版本，因为我的后台是flask，但是在flask框架里用的是python3的，官方的sdk不支持python3，改来改去，也还是有各种问题，因此采用了别的建议，这个接口单独用php的Laravel5框架来实现。
> Laravel整合的时候 也碰到各种各样的问题。

![](7.png)

把下载的包里面的 `api_sdk/lib/Core` 和 `api_sdk/lib/Api`复制一份放到项目 `app/libs/Aliyun` 下，没有 `libs` 文件夹就新建。

<img src="6.png" width="50%" />

然后打开项目根目录下的 `composer.json` 文件，找到下面的位置 `classmap`，加上图示代码

<img src="8.png" width="70%" />

然后终端(Terminal)进入项目根目录，执行

```bash
composer dumpautoload
```

执行后若出现如下图所示，则表示更新成功

![composer dumpautoload](9.png)

然后就可以直接使用sdk里面导入包的方式了。

![](10.png)

------

真正做的时候也不是这样一帆风顺的，官方的sdk和demo本身也有些问题。

##### Invalid argument supplied for foreach()

![](11.png)

直接把Demo复制过来，就会出现这样的问题

查了很多地方，发现是少了一句demo里少了一行代码

在 `SmsController.getAcsClient()` 里补上(原来是没的)

```
EndpointConfig::load();
```

![](12.png)

##### Can not find endpoint to access.

天真的以为问题都解决了，结果马上出来另外一个问题。

![](13.png)

找来源发现问题在这里 `app/libs/Aliyun/Core/DefaultAcsClient.doActionImpl()`，是程序主动抛出的错误。

![](15.png)

回去看看这个 `$domain` 是什么，结果官方注释说 $domain是产品域名,开发者无需替换。我都没替换，你报个什么错啊。

![](14.png) 

算了，既然不需要，那就直接这里吧 `$domain` 写死好了。

```
$domain = 'dysmsapi.aliyuncs.com';
```

![](16.png)

#### 具体实现

我就把 **smsDemo** 里的几个方法都移到 `SmsController` 里了，

```php
<?php
namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Support\Facades\DB;

use Aliyun\Core\Config;
use Aliyun\Core\Profile\DefaultProfile;
use Aliyun\Core\DefaultAcsClient;
use Aliyun\Api\Sms\Request\V20170525\SendSmsRequest;
use Aliyun\Api\Sms\Request\V20170525\QuerySendDetailsRequest;
use Aliyun\Core\Regions\EndpointConfig;

class SmsController extends Controller{

    // 可供调用的发送验证码api
    public function send(Request $request){
        header('Content-Type: text/plain; charset=utf-8');
        $phone = $request->input('phone');
        $code = $this::randomKeys(); // 随机生成6位数字验证码
        $response = SmsController::sendSms(
            "富春江app", // 短信签名
            "SMS_111785732", // 短信模板编号
            $phone, // 短信接收者
            Array(  // 短信模板中字段的值
              "code"=>$this::randomKeys(),
            ),
            "123456"   // 流水号,选填
        );
        $row = [];
        if($response->Code=='OK'){
            $row = ['status'=>1,'code'=>$code];
        }else{
            $row = ['status'=>0,'code'=>$response->Code,'message'=>$response->Message];
        }        
        $row = ['status'=>1,'code'=>$code];
        return json_encode($row);
    }

    
    // 随机生成6位数字验证码
    public function randomKeys($length = 6){
        $key='';
        $pattern='1234567890';
        for($i=0;$i<$length;++$i){
            $key .= $pattern{mt_rand(0,9)};    // 生成php随机数
        }
        return $key;
    }

    static $acsClient = null;

    /**
     * 取得AcsClient
     *
     * @return DefaultAcsClient
     */
    public static function getAcsClient() {
        //产品名称:云通信流量服务API产品,开发者无需替换
        $product = "深呼吸app";

        //产品域名,开发者无需替换
        $domain = "dysmsapi.aliyuncs.com";

        // TODO 此处需要替换成开发者自己的AK (https://ak-console.aliyun.com/)
        $accessKeyId = "LTAIp5j2DxU47O42"; // AccessKeyId

        $accessKeySecret = "EY9OdjeinAnEOeDyFNZb2OiT3Xg4ep"; // AccessKeySecret


        // 暂时不支持多Region
        $region = "cn-hangzhou";

        // 服务结点
        $endPointName = "cn-hangzhou";


        if(static::$acsClient == null) {

            //初始化acsClient,暂不支持region化
            $profile = DefaultProfile::getProfile($region, $accessKeyId, $accessKeySecret);

            EndpointConfig::load();
            // 增加服务结点
            DefaultProfile::addEndpoint($endPointName, $region, $product, $domain);

            // 初始化AcsClient用于发起请求
            static::$acsClient = new DefaultAcsClient($profile);
        }
        return static::$acsClient;
    }

    /**
     * 发送短信
     *
     * @param string $signName <p>
     * 必填, 短信签名，应严格"签名名称"填写，参考：<a href="https://dysms.console.aliyun.com/dysms.htm#/sign">短信签名页</a>
     * </p>
     * @param string $templateCode <p>
     * 必填, 短信模板Code，应严格按"模板CODE"填写, 参考：<a href="https://dysms.console.aliyun.com/dysms.htm#/template">短信模板页</a>
     * (e.g. SMS_0001)
     * </p>
     * @param string $phoneNumbers 必填, 短信接收号码 (e.g. 12345678901)
     * @param array|null $templateParam <p>
     * 选填, 假如模板中存在变量需要替换则为必填项 (e.g. Array("code"=>"12345", "product"=>"阿里通信"))
     * </p>
     * @param string|null $outId [optional] 选填, 发送短信流水号 (e.g. 1234)
     * @param string|null $smsUpExtendCode [optional] 选填，上行短信扩展码（扩展码字段控制在7位或以下，无特殊需求用户请忽略此字段）
     * @return stdClass
     */
    public static function sendSms($signName, $templateCode, $phoneNumbers, $templateParam = null, $outId = null, $smsUpExtendCode = null) {

        // 初始化SendSmsRequest实例用于设置发送短信的参数
        $request = new SendSmsRequest();

        // 必填，设置雉短信接收号码
        $request->setPhoneNumbers($phoneNumbers);

        // 必填，设置签名名称
        $request->setSignName($signName);

        // 必填，设置模板CODE
        $request->setTemplateCode($templateCode);

        // 可选，设置模板参数
        if($templateParam) {
            $request->setTemplateParam(json_encode($templateParam));
        }

        // 可选，设置流水号
        if($outId) {
            $request->setOutId($outId);
        }

        // 选填，上行短信扩展码
        if($smsUpExtendCode) {
            $request->setSmsUpExtendCode($smsUpExtendCode);
        }

        // 发起访问请求
        $acsResponse = static::getAcsClient()->getAcsResponse($request);

        // 打印请求结果
        // var_dump($acsResponse);
        return $acsResponse;
    }
}
```








