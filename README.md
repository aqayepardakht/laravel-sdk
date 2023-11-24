sdk اتصال به api پرداخت آقای پرداخت

## نحوه نصب

نصب توسط کامپوزر

```bash
composer require aqayepardakht/laravel-sdk
```

##  نحوه استفاده سریع
افزودن پین به فایل .env

PIN=your pin 

ساخت فاکتور و ارسال به درگاه بانک
```php
namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Aqayepardakht;

class PayController extends Controller {
    public function pay() {
        try {       
            $pay = Aqayepardakht::gateway(env('PIN'))
                    ->invoice([
                        'amount'        => 2000,
                        'callback'      => 'http://example.com/callback',
// بقیه پارامتر های موردنیاز بر اساس داکیومنت آقای پرداخت
                    ])
                    ->create();

            $traceCode = $pay->getTrackingCode(); // دریافت کد رهگیری
        // بروزرسانی وضعیت خرید در دیتابیس
            $pay->start(); // ریدایرکت کاربر به صفحه پرداخت
        } catch (Exception $e) { 
            echo $e->getCode().' : '.$e->getMessage();
        }
    }
    // تایید تراکنش پس از بازگشت از صفحه بانکی
    public function verify(Request $request) {
        $trackingNumber = $request->tracking_number; // کد رهگیری بانکی
        $trackingCode   = $request->tracking_code; // کد رهگیری برای تایید تراکنش
    
        try {
            Aqayepardakht::gateway(env('PIN'))
                    ->invoice([
                        'amount' => 2000,
                    ])
                    ->verify($trackingCode);

        // بروزرسانی وضعیت خرید در دیتابیس
        } catch (Exception $e) {
            // مدیریت اررور های مربوط به پرداخت
            if ($e->getCode() === -34) {
                echo "پرداخت توسط مشتری لغو شده است";
            }
        }
    }
}
```
