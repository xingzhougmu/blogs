#如何为Azure Service Bus和Azure IoT Hub生成SharedAccessSignature
很多服务在做验证的时候都会用到SharedAccessSignature，例如Azure Service Bus, Azure IoT Hub等。今天趟了回大坑，这里分享出来，希望对你有所帮助。
SharedAccessSignature的格式如下：
```
SharedAccessSignature sig={signature-string}&se={expiry}&skn={policyName}&sr={URL-encoded-resourceURI}
```
因此，凭直觉针对不同的服务只要正确指定policyname,resourceURI 就可以计算出相应的SAS。但实际上，对于不同的服务，他们对如何利用秘钥生成signature会存在细微的差别。如果没有注意到这个细微的差别，会让你抓狂两小时。

下面拿Azure Service Bus和Azure IoT Hub进行举例，如果后续我碰到Azure 其他服务有类似的问题。我会更新此文。

首先是Azure IoT Hub, 针对如何生成signature [官方文档][1]说明如下：
>{signature} ：An HMAC-SHA256 signature string of the form: {URL-encoded-resourceURI} + "\n" + expiry. Important: **The key is decoded from >base64 and used as key** to perform the HMAC-SHA256 computation.
再来看Azure Service Bus 的[官方文档][2]说明：
>The signature for the SAS token is computed using the HMAC-SHA256 hash of a string-to-sign **with the PrimaryKey property of an authorization rule**.

两者的区别在于是否需要对秘钥进行decode。代码分别如下：
```csharp
HMACSHA256 hmac = new HMACSHA256(Convert.FromBase64String(key)); // decode the key
HMACSHA256 hmac = new HMACSHA256(Encoding.UTF8.GetBytes(key)); // don't decode the key
```
在[《Azure IoT Hub 入门 - 权限管理》][3]一文中，我跟大家share了生成SAS的source code 和DLL。该[DLL][4] 现支持生成Service Bus 和IoT Hub的SAS，两者通过TargetService来区分，调用方法如下：
## Service Bus:
```csharp
// servicebus
string targetURL = "sb://<service bus namespace>.servicebus.chinacloudapi.cn";
string keyName = "DefaultFullSharedAccessSignature";
string key = <key>;
double ttlValue = 1;
var sasBuilder = new SharedAccessSignatureBuilder()
{
    TargetService = "servicebus",
    Target = targetURL,
    KeyName = keyName,
    Key = key,
    TimeToLive = TimeSpan.FromDays(ttlValue)
};
string sas = sasBuilder.ToSignature();
```

## IoT Hub: (TargetService是可选项，默认是生成IoT hub的SAS)
```csharp
//iot hub
string targetURL = "<iot hub name>.azure-devices.cn/devices";
string keyName = <policy name>;
string key = <key>;

var sasBuilder1 = new SharedAccessSignatureBuilder()
{
    Target = targetURL,
    KeyName = keyName,
    Key = key,
    TimeToLive = TimeSpan.FromDays(ttlValue)
};

string sas1 = sasBuilder1.ToSignature();

var sasBuilder2 = new SharedAccessSignatureBuilder()
{
    TargetService="iothub", // optional
    Target = targetURL,
    KeyName = keyName,
    Key = key,
    TimeToLive = TimeSpan.FromDays(ttlValue)
};

string sas2 = sasBuilder1.ToSignature();
```


[1]: https://azure.microsoft.com/en-us/documentation/articles/service-bus-shared-access-signature-authentication/
[2]: https://azure.microsoft.com/en-us/documentation/articles/service-bus-shared-access-signature-authentication/
[3]: http://www.cnblogs.com/xingzhou/p/6046639.html
[4]: http://files.cnblogs.com/files/xingzhou/SharedAccessSignatureGenerator.zip
