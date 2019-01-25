## 2019年1月25日 HZ 晴朗  风暴-0125

今天解决了一个由于分布式系统中Cookie中读取不到的问题，导致引起了奇奇怪怪的问题。

这个问题今天终于解决了，原来是我们依赖的框架会对生成的Cookie加密，在服务端会根据签名检查Cookie有没有被篡改过。


Cookie的Value 被设置的时候设置为

```php 
hashData(serialize([$cookie->name, $value]), $validationKey);
```


``` php
   public function hashData($data, $key, $rawHash = false)
    {
        $hash = hash_hmac($this->macHash, $data, $key, $rawHash);
        if (!$hash) {
            throw new InvalidConfigException('Failed to generate HMAC with hash algorithm: ' . $this->macHash);
        }

        return $hash . $data;
    }

```
这里生成的Cookie的Value增加了一段前面 hash. 


