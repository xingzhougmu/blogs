#如何为Azure Service Bus和Azure IoT Hub生成SharedAccessSignature
很多服务在做验证的时候都会用到SharedAccessSignature，例如Azure Service Bus, Azure IoT Hub等。今天趟了回大坑，这里分享出来，希望对你有所帮助。
SharedAccessSignature的格式如下：
```csharp
SharedAccessSignature sig={signature-string}&se={expiry}&skn={policyName}&sr={URL-encoded-resourceURI}
```
