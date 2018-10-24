# Laravel随记

## 问题1：API的CSRF Token

将App/Http/kernel.php中的csrf-token中间件去掉即可

## 问题2：Laravel接收各类请求

在controller中加入

```php
use Illuminate\Support\Facades\Input;
```

之后在Controller中接收请求时：

```php
public function verify_code(Request $request)
    {
        $contact = Input::get('contact');
```

就可以获得请求中的参数

## 问题3：Laravel-Mysql外键的要求

- 外键在来源的表中必须是主键
- 添加外键的表，字段的类型必须和外键来源表的字段类型一样。

## 问题4：Laravel模型操作

如模型为：App\User

- 想要使用模型的配置：App\User<font color=#FF0033>()</font>->...
- 想要使用模型的方法：App\User->...

## 问题5：strcmp

strcmp若两个字符串相等则返回0

