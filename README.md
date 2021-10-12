# Laravel PayPal

This plugin only supports Laravel 5.1 to 5.8.

- [Introduction](#introduction)
- [Installation](#installation)
- [Configuration](#configuration)
- [Usage](#usage)
- [Support](#support)

    
<a name="introduction"></a>
## Introduction

By using this plugin you can process or refund payments and handle IPN (Instant Payment Notification) from PayPal in your Laravel application.

**This plugin supports the new paypal rest api.**

This package uses the new paypal rest api. Refer to this link on how to create API credentials:

https://developer.paypal.com/docs/api/overview/

<a name="installation"></a>
## Installation

### 1. Install:

```bash
composer require artwl/laravel-paypal:~2.0
```

### 2. [Optional] Add the service provider in `config/app.php`: 

```php
Srmklive\PayPal\Providers\PayPalServiceProvider::class
```

### 3. [Optional] Add the alias in `config/app.php`: 

```php
'PayPal' => Srmklive\PayPal\Facades\PayPal::class
```

### 4. Publish configuration:

```bash
php artisan vendor:publish --provider "Srmklive\PayPal\Providers\PayPalServiceProvider"
```

<a name="configuration"></a>
## Configuration

* After installation, you will need to add your paypal settings. Following is the code you will find in **config/paypal.php**, which you should update accordingly.

```php
return [
    'mode'    => env('PAYPAL_MODE', 'sandbox'), // Can only be 'sandbox' Or 'live'. If empty or invalid, 'live' will be used.
    'sandbox' => [
        'client_id'         => env('PAYPAL_SANDBOX_CLIENT_ID', ''),
        'client_secret'     => env('PAYPAL_SANDBOX_CLIENT_SECRET', ''),
        'app_id'            => 'APP-80W284485P519543T',
    ],
    'live' => [
        'client_id'         => env('PAYPAL_LIVE_CLIENT_ID', ''),
        'client_secret'     => env('PAYPAL_LIVE_CLIENT_SECRET', ''),
        'app_id'            => '',
    ],

    'payment_action' => env('PAYPAL_PAYMENT_ACTION', 'Sale'), // Can only be 'Sale', 'Authorization' or 'Order'
    'currency'       => env('PAYPAL_CURRENCY', 'USD'),
    'notify_url'     => env('PAYPAL_NOTIFY_URL', ''), // Change this accordingly for your application.
    'locale'         => env('PAYPAL_LOCALE', 'en_US'), // force gateway language  i.e. it_IT, es_ES, en_US ... (for express checkout only)
    'validate_ssl'   => env('PAYPAL_VALIDATE_SSL', true), // Validate SSL when creating api client.
];
```

* Add this to `.env`

```
#PayPal API Mode

# Values: sandbox or live (Default: live)
PAYPAL_MODE=live

#PayPal Setting & API Credentials - sandbox
PAYPAL_SANDBOX_CLIENT_ID=
PAYPAL_SANDBOX_CLIENT_SECRET=

#PayPal Setting & API Credentials - live
PAYPAL_LIVE_CLIENT_ID=
PAYPAL_LIVE_CLIENT_SECRET=
```

<a name="usage"></a>
## Usage

Following are some ways through which you can access the paypal provider:

```php
// Import the class namespaces first, before using it directly
use Srmklive\PayPal\Services\PayPal as PayPalClient;


public function orderCreate(){
    $provider = new PayPalClient;
    $provider->setApiCredentials(config('paypal'));
    $provider->getAccessToken();
    
    $orderResponse = $provider->createOrder([
        "intent"=> "CAPTURE",
        "purchase_units"=> [
            0 => [
                "amount"=> [
                    "currency_code"=> "HKD",
                    "value"=> "12.00"
                ]
            ]
        ]
    ]);
    
    //order create success
    if ($orderResponse["status"] == "CREATED") {
        $orderId = $orderResponse["id"];
    }
}


//notify_url and webhook url, route need except csrf
public function payResult(Request $request) {
    $data = json_decode($request->getContent(), true);
    //pay success
    if ($data["event_type"] == "CHECKOUT.ORDER.APPROVED") {
        $orderId = $data["resource"]["id"];
    }
}
```
 
