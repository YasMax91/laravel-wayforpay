This repository is a fork from original https://github.com/maksa988/laravel-wayforpay, i decided to fork and work independent because the original one was not being updated for long time and keep doing support for the application.

# Laravel payment processor package for WayForPay gateway

Accept payments via WayForPay ([wayforpay.com](https://wayforpay.com/)) using this Laravel framework package ([Laravel](https://laravel.com)).

- receive payments, adding just the two callbacks

#### Laravel >= 8.0.2

## Installation

Require this package with composer.

``` bash
composer require "sonix/laravel-wayforpay"
```

If you don't use auto-discovery, add the ServiceProvider to the providers array in `config/app.php`

```php
Sonix\WayForPay\WayForPayServiceProvider::class,
```

Add the `WayForPay` facade to your facades array:

```php
'WayForPay' => Sonix\WayForPay\Facades\WayForPay::class,
```

Copy the package config to your local config with the publish command:
``` bash
php artisan vendor:publish --provider="Sonix\WayForPay\WayForPayServiceProvider"
```

## Configuration

Once you have published the configuration files, please edit the config file in `config/wayforpay.php`.

- Create an account and merchant on [wayforpay.com](http://wayforpay.com)
- Add your project, copy the `merchantAccount`, `merchantAccount`, `merchantSecretKey` params and paste into `config/wayforpay.php`
- After the configuration has been published, edit `config/wayforpay.php`
 
## Usage

This package using official WayForPay SDK for PHP. You can find a full description and content of the classes used by this package in the official SDK repository - [wayforpay/php-sdk](https://github.com/wayforpay/php-sdk)

#### 1. Purchase

Purchase request is used to effect payment with client on the protected wayforpay site.

Official documentation - https://wiki.wayforpay.com/en/view/852102

The method `purchase()` allows you to prepare data for a widget or form. And also you can get an array with data to build your form.

```php
$order_id = time(); // Payment`s order ID
$amount = 100; // Payment`s amount
$client = new \Sonix\WayForPay\Domain\Client('John', 'Doe', 'johndoe@gmail.com');
$products = new \Sonix\WayForPay\Collection\ProductCollection([
    new \WayForPay\SDK\Domain\Product('iPhone 12', 10, 1),
]);
//
$data = WayForPay::purchase($order_id, $amount, $client, $products)->getData(); // Array of data for using to create your own form.
$form = WayForPay::purchase($order_id, $amount, $client, $products)->getAsString($submitText = 'Pay', $buttonClass = 'btn btn-primary'); // Get html form as string
```

You can get JS code for widget (https://wiki.wayforpay.com/en/view/852091) using `getWidget` method after call `purchase` method.

```php
$widget = WayForPay::purchase($order_id, $amount, $client, $products)->getWidget($callbackJsFunction = null, $buttonText = 'Pay', $buttonClass = 'btn btn-primary'); // Get html form as string
```

#### 2. Charge

Charge request is used for quick payment making in one action. It is performed within the limits of single-staged pattern.

The result of request processing is the withdrawal of monetary assets from client’s card.

Official documentation - https://wiki.wayforpay.com/en/view/852194

The method `charge()` allows you to send request for charge operation and get object of response.

```php
$card = new \Sonix\WayForPay\Domain\Card('5276999765600381', '05', '2021', '237', 'JOHN DOU');
$cardToken = new \Sonix\WayForPay\Domain\Card('1aa11aaa-1111-11aa-a1a1-0000a00a00aa');
```

You can use `\Sonix\WayForPay\Domain\Card::class` instead of `WayForPay\SDK\Domain\Card` and or `WayForPay\SDK\Domain\CardToken`. This class simplify input card and card token using one class.
When you put only first argument, this card defined as card-token. If you are put all arguments, this be defined as bank card.

```php
$response = WayForPay::charge($order_id, $amount, $client, $products, $card);
$response = WayForPay::charge($order_id, $amount, $client, $products, $cardToken);
echo "Status: ". $response->getTransaction()->getStatus();
```

#### 3. Check Status

Check Status  request is used for checking of payment status on orderReference. 

Official documentation - https://wiki.wayforpay.com/en/view/852117

The method `check()` allows you to send request for check status of your order using order id.

```php
$order = WayForPay::check($order_id)->getOrder();
echo "Status: ". $order->getStatus();
```

#### 4. Refund

Refund  request is to be used for making of assets refund or cancellation of payment.

Official documentation - https://wiki.wayforpay.com/en/view/852115

The method `refund()` allows you to send request for refund of payment.

```php
WayForPay::refund($order_id, $amount, $currency, $comment)->getTransactionStatus();
```

#### 5. Create invoice

The present API allows to issue invoices to the clients for payment for goods/services.

Official documentation - https://wiki.wayforpay.com/en/view/608996852

The method `createInvoice()` allows you to create invoice.

```php
$invoice = WayForpay::createInvoice($order_id, $amount, $client, $products);
$url = $invoice->getInvoiceUrl();
$qrCode = $invoice->getQrCode();
```

#### 6. Complete 3DS

In case of merchantTransactionSecureTtype= 3DS, there is initially performed the checking of the card for participation in 3d secure program.
If the card supports 3D Secure  verification the system Wayforpay will return the parameters for authentication of the client.
With these parameters the merchant has to transfer the client to url of issuer for authentication. 
The time during which the session for verification is active - 10 minutes. If within 10 minutes COMPLETE_3DS will not be obtained the system will cancel transaction as unsuccessful.

```php
$response = WayForPay::complete3ds($authTicket, $d3Md, $d3Pares);
$response->getTransaction();
```

#### 7. Handle payment

You can use handle payment process using service url at WayForPay. And using controller at Laravel with this package you can handle payment.

For handle request from wayforpay you should create controller and action. In the action you should use method `handleServiceUrl()`.

In the first argument you should put array of request data or `Arrayble` class. In the second argument you should put `Closure` what be called when payment is success, and this function will be passed two parameters:
`Transaction` object and `Closure` using for create success response.

```php
// Controller action
public function handle(Request $request)
{
    return WayForPay::handleServiceUrl($request, function (\WayForPay\SDK\Domain\Transaction $transaction, $success) {
        if($transaction->getReason()->isOK()) {
            // Payment confirmation process and etc...
            return $success();
        }
        return "Error: ". $transaction->getReason()->getMessage();
    });
}
```

## Changelog

Please see [CHANGELOG](CHANGELOG.md) for more information on what has changed recently.

## Contributing

Please see [CONTRIBUTING](CONTRIBUTING.md) for details.

## Security

If you discover any security related issues, please send me an email at maksa988ua@gmail.com instead of using the issue tracker.

## Credits

- [Sonix](https://github.com/YasMax91)
- [All Contributors](../../contributors)

## License

The MIT License (MIT). Please see [License File](LICENSE.md) for more information.
