# kuaidi

一个用于 Yii2 的快递运单查询库，覆盖国内绝大多数快递公司。

基于 [wi1dcard/kuaidi](https://github.com/wi1dcard/kuaidi) 改造。

## 安装

```bash
composer require ynddddd/kuaidi
```

## 要求

- PHP >= 5.4
- yiisoft/yii2 ~2.0.5
- curl/curl ^1.5

## 查询渠道

| 渠道 | 类 | 是否需要凭据 |
| --- | --- | --- |
| 快递100 | `Shophx\Express\Trackers\Kuaidi100` | 否 |
| 快递鸟 | `Shophx\Express\Trackers\Kuaidiniao` | 是（`EBusinessID` / `AppKey`） |
| 快递网 | `Shophx\Express\Trackers\Kuaidiwang` | 否 |

## 用法

```php
use Shophx\Express\Waybill;
use Shophx\Express\Trackers\Kuaidi100;

$waybill = new Waybill();
$waybill->id = '780098068058';   // 运单号
$waybill->express = '中通';       // 快递公司名称

$traces = $waybill->getTraces(new Kuaidi100());

foreach ($traces as $trace) {
    // 每项为数组：time / desc / memo
    echo $trace['time'], ' ', $trace['desc'], PHP_EOL;
}

echo $waybill->status;   // 见下方状态常量
```

使用快递鸟渠道时需要传入凭据：

```php
use Shophx\Express\Trackers\Kuaidiniao;

$tracker = new Kuaidiniao([
    'EBusinessID' => 'your-ebusiness-id',
    'AppKey' => 'your-app-key',
]);

$traces = $waybill->getTraces($tracker);
```

查询某个渠道支持的快递公司：

```php
Kuaidi100::getSupportedExpresses();      // ['中通' => 'zhongtong', ...]
Kuaidi100::isSupported('中通');           // true
Kuaidi100::getExpressCode('中通');        // 'zhongtong'
```

## 运单状态

`Shophx\Express\Status` 中定义的常量：

| 常量 | 值 | 含义 |
| --- | --- | --- |
| `STATUS_UNKNOWN` | -1 | 未知 |
| `STATUS_PICKEDUP` | 0 | 揽件 |
| `STATUS_DEPART` | 1 | 发出 |
| `STATUS_TRANSPORTING` | 2 | 在途 |
| `STATUS_DELIVERING` | 3 | 派件 |
| `STATUS_DELIVERED` | 4 | 签收 |
| `STATUS_SELFPICKUP` | 5 | 自取 |
| `STATUS_REJECTED` | 6 | 疑难 |
| `STATUS_RETURNING` | 7 | 退回 |
| `STATUS_RETURNED` | 8 | 退签 |

## 异常

查询失败会抛出 `Shophx\Express\Exceptions\TrackingException`：

```php
use Shophx\Express\Exceptions\TrackingException;

try {
    $traces = $waybill->getTraces(new Kuaidi100());
} catch (TrackingException $e) {
    echo $e->getMessage();
}
```