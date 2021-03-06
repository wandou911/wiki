# iOS内购流程

## 1. 背景

* iOS IAP 的机制是有问题的。并不是金融级别的支付校对流程
* iOS IAP是在客户端完成的，是单向的过程，使用 IAP 内购会有可能导致订单丢失

## 2. 支付状态

* SKPaymentTransactionStatePurchasing：正在支付
* SKPaymentTransactionStatePurchased：已支付
* SKPaymentTransactionStateFailed：支付失败
* SKPaymentTransactionStateRestored：恢复购买，例如非消耗商品在iPad已经购买了，在iPhone恢复，或者卸载了App，重装没有及时更新状态，可以用这个恢复，用于非消耗品
* SKPaymentTransactionStateDeferred：未确定状态，由于外部原因导致的（如家长控制，未测试）

## 3. IAP支付流程

* 1 根据productId（com.nsdk.sdk.6）获取SKProduct

* 2 把SKProductId加到购买队列里面，并且把外部的orderId，绑定到SKPayment的applicationUsername上

![](https://blog.bombox.org/images/post/image2018-7-5_20-42-59.png)

* 3 从SKPaymentTransactionObserver监听支付结果，在监听到支付成功后，可以拿到SKPayment绑定的CustomOrderId，把苹果的订单和我们订单绑定上（根据网上描述的掉单情况，有一定概率是在回调的地方获取不到CustomOrderId，这个时候就会出现掉单的情况，而如果这个时候绑定了一个其他的orderId，就会出现串单的情况，但是没有验证成功）

![](https://blog.bombox.org/images/post/image2018-7-5_22-9-13.png)

* 4 当客户端调用finishTransaction时，则表示订单已经完成，则客户端不再接收到支付成功的回调，如果没有finishTransaction，则苹果会一直回调（每次打开App(监听)就会回调，直到调用finishTransaction完成订单）

## 4. 调研汇总

同一个商品，如果上次支付用户支付成功SKPaymentTransactionStatePurchased，但是没有调用finishTransaction），再次下单购买的时候，会提示恢复购买，只会调用Purchasing，不会监听到其他状态，并且用户不会扣钱，如果重新打开App，重新监听SKPaymentTransactionObserver，会收到多条回调，并且对应的transactionId一样，也就是同一个商品，再未完成前，不会重复扣款，只有上一个订单完成后，才会继续支付扣款

![](https://blog.bombox.org/images/post/image2018-7-5_20-25-28.png)

purchasing状态下还没有唯一标识transactionIdentifier，只有在purchased和restore状态下才有

在iOS7以后，苹果的支付票据保存在Bundle.main.appStoreReceiptURL，票据只有一份，并且是加密的，无法再客户端进行拆分订单，用户支付成功后，信息会存在票据中，当订单完成（finishTransaction），会从票据中把相应的订单删除，客户端无法知道票据包含哪些支付成功的订单，同一个票据里面，会有多个订单，可以从苹果的验证接口返回数据，看到票据包含哪些订单，通常情况下只有一个，不排除有多个，如下

```
{
	"status": 0,
	"environment": "Sandbox",
	"receipt": {
		"receipt_type": "ProductionSandbox",
		"adam_id": 0,
		"app_item_id": 0,
		"bundle_id": "com.nsdk.sdk",
		"application_version": "1",
		"download_id": 0,
		"version_external_identifier": 0,
		"receipt_creation_date": "2018-07-05 12:31:44 Etc/GMT",
		"receipt_creation_date_ms": "1530793904000",
		"receipt_creation_date_pst": "2018-07-05 05:31:44 America/Los_Angeles",
		"request_date": "2018-07-05 12:32:20 Etc/GMT",
		"request_date_ms": "1530793940135",
		"request_date_pst": "2018-07-05 05:32:20 America/Los_Angeles",
		"original_purchase_date": "2013-08-01 07:00:00 Etc/GMT",
		"original_purchase_date_ms": "1375340400000",
		"original_purchase_date_pst": "2013-08-01 00:00:00 America/Los_Angeles",
		"original_application_version": "1.0",
		"in_app": [{
				"quantity": "1",
				"product_id": "com.nsdk.sdk.6",
				"transaction_id": "1000000414405534",
				"original_transaction_id": "1000000414405534",
				"purchase_date": "2018-07-05 12:23:43 Etc/GMT",
				"purchase_date_ms": "1530793423000",
				"purchase_date_pst": "2018-07-05 05:23:43 America/Los_Angeles",
				"original_purchase_date": "2018-07-05 12:23:43 Etc/GMT",
				"original_purchase_date_ms": "1530793423000",
				"original_purchase_date_pst": "2018-07-05 05:23:43 America/Los_Angeles",
				"is_trial_period": "false"
			},
			{
				"quantity": "1",
				"product_id": "com.nsdk.sdk.12",
				"transaction_id": "1000000414404413",
				"original_transaction_id": "1000000414404413",
				"purchase_date": "2018-07-05 12:20:20 Etc/GMT",
				"purchase_date_ms": "1530793220000",
				"purchase_date_pst": "2018-07-05 05:20:20 America/Los_Angeles",
				"original_purchase_date": "2018-07-05 12:20:20 Etc/GMT",
				"original_purchase_date_ms": "1530793220000",
				"original_purchase_date_pst": "2018-07-05 05:20:20 America/Los_Angeles",
				"is_trial_period": "false"
			}
		]
	}
}
```

用户购买了”元宝6”和”元宝12”两个商品，并且支付成功，但由于网络原因，回调的时候没有同步到服务器（还没finishTransaction），下次启动（监听SKPaymentTransactionObserver）时，两个订单都会回调支付成功

如果用户购买了”元宝6”，并且支付成功，但是由于网络原因，回调的时候没有同步到服务器（还没finishTransaction），用户又重新购买了几次（提示已购买，此项目将免费恢复），然后重启App，SKPaymentTransactionObserver将会回调多次，并且订单transactionIdentifier相同（这里是个坑，用户在SDK下单多次，但是在苹果支付只一次，而回调成功会多次，而客户端逻辑会以为多次下单都成功了，需要服务端做去重控制）

刚打开App的时候，通常会监听订单状态，之前没有同步成功的订单会收到通知，由于苹果的票据只有一份，每个订单都会使用同一个票据去服务端校验（有可能同一个订单会回调多次），服务端需要考虑去重的问题（transactionId），避免同一个票据刷多次
如果支付没有完成，卸载App，重装，也能收到回调

正常情况下下单，update回调会先触发purchasing，然后触发purchased或failed

由于情况1的存在，如果已存在一个已支付但是未完成的订单，这个时候再下一个新的单（productId相同），监听回调只有一次purchasing，不会有purchased或failed回调（巨坑）

这个时候调用的地方就不知道用户什么时候支付完成了，造成的问题是支付前显示Loading，而没有关闭回调，导致loading一直显示

推荐解决方案：在监听paymentQueue:updatedTransactions方法时，使用queue.transactions，而不是用参数transaction，如下

```
- (void)paymentQueue:(SKPaymentQueue *)queue updatedTransactions:(NSArray<SKPaymentTransaction *> *)transactions {
    // 原
    // for (SKPaymentTransaction *transaction in transactions) {
    
    // 改
    for (SKPaymentTransaction *transaction in queue.transactions) {
        // TODO:
        switch (transaction.transactionState) {
            case SKPaymentTransactionStateFailed:
                break;
            case SKPaymentTransactionStateDeferred:
                break;
            case SKPaymentTransactionStateRestored:
                break;
            case SKPaymentTransactionStatePurchased:
                break;
            case SKPaymentTransactionStatePurchasing:
                break;
        }
    }
}
```

TODO：上面还有一个问题，由于取的是数组，可能会有多个，导致触发回调多次，发货也触发多次，就是可能会回调多次

## 5. 常见问题

串单

由于正在处理的订单可能不止一个，如果使用单例共用状态orderId，如果时机不对（多个单一起出现），会出现orderId和苹果订单对应不上
应该通过applicationUsername把苹果订单和我们的orderId绑定起来，避免关系错乱，导致串单
由于苹果票据只有一份，多个订单也会使用同一个票据，而验证通过，这一点需要服务端也做相关的去重处理

掉单

客户端把票据传给服务器后，就标识订单finish，服务端校验苹果票据是异步的，如果校验失败，则会调单
由于票据只有一份，并且可能包含多个订单，服务端验证票据时，需要进行分别判断，很多人的做法是只取第一个，导致校验失败，从而调单

刷单

如果服务器端对苹果平局没有做去重校验，同一个票据可以被校验多次，用户可能会因为这个无意刷单，支付一次，发货多次（概率很低，但是有）

## 6. 措施

* 添加applicationUsername用于绑定苹果订单和SDK订单，去除共用orderId
* 服务端优化订单校验，校验失败的处理（改为同步，并返回给客户端？）
* 由于苹果票据可能含有多个订单，服务端在做订单校验的时候需要针对指定的订单处理
* 由于苹果一个票据对应多个订单，客户端可能多个订单使用同一个票据，服务端需要做苹果订单去重处理，同一个票据不应该校验两次（根据transactionId）

## 7. 不可避免问题

苹果订单和SDK订单的绑定关系丢失（网上很多丢单问题这么说，我没用重现出来，只是可能，不能确定，考虑埋点统计）（orderId == null）

补救：后台做记录，根据用户反馈手动补单

## 8. 其他

上面描述不包含订阅类型

## 9 引用

https://stackoverflow.com/questions/25510678/how-to-test-skpaymenttransactionstatedeferred/27367749

#### 参考链接

1 [iOS内购掉单问题](https://blog.bombox.org/2018-07-14/ios-iap/)

2 [Keep客户端 In-App Purchase 掉单踩坑指南](http://tech.gotokeep.com/post/2018/12/in-app-purchase/)
