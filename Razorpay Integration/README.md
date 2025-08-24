# 💳 Razorpay Payment Integration (Laravel)

This guide explains step-by-step how to integrate **Razorpay Checkout** with a Laravel project.

---

## Step 1: Install Package

```bash
composer require razorpay/razorpay
```

---

## Step 2: Add Credentials to `.env`

```env
RAZORPAY_KEY_ID=rzp_test_xxxxxxxxxxxx
RAZORPAY_KEY_SECRET=xxxxxxxxxxxxxxxxxxxx
RAZORPAY_WEBHOOK_SECRET=   # only if you enable webhooks
RAZORPAY_CURRENCY=INR
```

---

## Step 3: Update `config/services.php`

```php
'razorpay' => [
    'key'           => env('RAZORPAY_KEY_ID'),
    'secret'        => env('RAZORPAY_KEY_SECRET'),
    'webhook_secret'=> env('RAZORPAY_WEBHOOK_SECRET'),
    'currency'      => env('RAZORPAY_CURRENCY', 'INR'),
],
```

---

## Step 4: Create Database for Payments

```bash
php artisan make:model Payment -m
```

### Migration: `database/migrations/xxxx_xx_xx_create_payments_table.php`

```php
Schema::create('payments', function (Blueprint $t) {
    $t->id();
    $t->string('name');
    $t->string('email');
    $t->string('contact');
    $t->unsignedBigInteger('amount'); // in paise
    $t->string('currency', 5)->default('INR');
    $t->string('status')->default('created'); // created|paid|failed|signature_failed
    $t->string('order_id')->nullable();
    $t->string('payment_id')->nullable();
    $t->string('signature')->nullable();
    $t->json('payload')->nullable();
    $t->timestamps();
});
```

### Migrate Table
```bash
php artisan migrate
```

### Model: `app/Models/Payment.php`

```php
class Payment extends Model
{
    protected $fillable = [
        'name','email','contact','amount','currency','status',
        'order_id','payment_id','signature','payload'
    ];

    protected $casts = ['payload' => 'array'];
}
```

---

## Step 5: Define Routes

```php
use App\Http\Controllers\PaymentController;
use App\Http\Controllers\PaymentWebhookController;

Route::get('/pay', [PaymentController::class, 'form'])->name('pay.form');
Route::post('/pay/order', [PaymentController::class, 'createOrder'])->name('pay.order');
Route::post('/payment/verify', [PaymentController::class, 'verify'])->name('payment.verify');

// After Response from razorpay redirect to this routes for displaying success/failure messages
// You can directly send to another pages also
Route::get('/payment/success/{payment}', [PaymentController::class, 'success'])->name('payments.success');
Route::get('/payment/failed/{payment?}', [PaymentController::class, 'failed'])->name('payments.failed');

// optional webhook
Route::post('/razorpay/webhook', [PaymentWebhookController::class, 'handle'])->name('razorpay.webhook');
```

---

## Step 6: Create Controller

```bash
php artisan make:controller PaymentController
```

### Controller: `app/Http/Controllers/PaymentController.php`

```php
namespace App\Http\Controllers;

use App\Models\Payment;
use Illuminate\Http\Request;
use Razorpay\Api\Api;
use Razorpay\Api\Errors\SignatureVerificationError;


class PaymentController extends Controller
{
    // Razorpay Api object
    protected Api $api;

    // Creating Razorpay Api object so we can use it in anywhere in this controller
    public function __construct()
    {
        $this->api = new Api(
            config('services.razorpay.key'),
            config('services.razorpay.secret')
        );
    }

    // Show the form -- checkout form or any other form from where we have button of making payments
    public function form()
    {
        return view('payments.form');
    }

    // 1) Create an order on server (REQUIRED by Razorpay)
    public function createOrder(Request $request)
    {
        // form validation
        $data = $request->validate([
            'full_name' => ['required', 'string', 'max:120'],
            'email' => ['required', 'email', 'max:120'],
            'contact' => ['required', 'regex:/^[0-9]{10,15}$/'],
            'amount' => ['required', 'numeric', 'min:1'], // amount in INR rupees
        ]);

        // convert amount from rupees to paise. razorpay accepts in paise
        $amountPaise = (int) round($data['amount'] * 100);

        // Create order via API - This will return an array with id, amount, currency, etc.
        $order = $this->api->order->create([
            'amount' => $amountPaise,
            'currency' => config('services.razorpay.currency', 'INR'),
            'receipt' => 'rcpt_' . now()->timestamp,
            'notes' => [
                'customer_name' => $data['full_name'],
                'customer_email' => $data['email'],
                'customer_phone' => $data['contact'],
            ],
        ]);

        // Save to DB
        $payment = Payment::create([
            'name' => $data['full_name'],
            'email' => $data['email'],
            'contact' => $data['contact'],
            'amount' => $amountPaise,
            'currency' => 'INR',
            'status' => 'created',
            'order_id' => $order['id'],
        ]);

        // Redirect to the checkout page with razorpay key, payment details, order details and form filled data
        // Render a page that immediately opens Razorpay Checkout
        return view('payments.checkout', [
            'key' => config('services.razorpay.key'),
            'order' => $order,
            'payment' => $payment,
            'prefill' => $data,
        ]);
    }

    // 2) Verify the signature (MANDATORY)
    public function verify(Request $request)
    {
        // validate data that are coming from payments.checkout page
        // that page makes a fetch api request
        $attrs = $request->validate([
            'razorpay_order_id' => 'required|string',
            'razorpay_payment_id' => 'required|string',
            'razorpay_signature' => 'required|string',
        ]);

        // get details of payment from db
        $payment = Payment::where('order_id', $attrs['razorpay_order_id'])->firstOrFail();

        // try-catch for error handling
        try {
            // Throws exception if invalid. Returns null on success.
            $this->api->utility->verifyPaymentSignature($attrs);

            // Optional: confirm status via API and (if needed) capture
            $rpPayment = $this->api->payment->fetch($attrs['razorpay_payment_id']);

            //check for verified or not
            if (($rpPayment['status'] ?? null) !== 'captured') {
                // If your Dashboard capture setting is Auto, this should already be captured.
                // Capture manually only if Auto Capture is OFF.
                $this->api->payment
                    ->fetch($attrs['razorpay_payment_id'])
                    ->capture(['amount' => $payment->amount]);
            }

            // if payment is verified update records in db
            $payment->update([
                'payment_id' => $attrs['razorpay_payment_id'],
                'signature' => $attrs['razorpay_signature'],
                'status' => 'paid',
                'payload' => $attrs,
            ]);

            // send response back to the payments.checkout page where our fetch api request gets this response
            return response()->json([
                'ok' => true,
                'redirect_url' => route('payments.success', $payment),
            ]);
        } catch (SignatureVerificationError $e) {
            // if payments verification fails then update record in db as failed
            $payment->update([
                'status' => 'signature_failed',
                'payload' => $attrs,
            ]);


            // send response back to the payments.checkout page where our fetch api request gets this response
            return response()->json([
                'ok' => false,
                'redirect_url' => route('payments.failed', ['payment' => $payment->id]),
            ], 422);
        }
    }

    public function success(Payment $payment)
    {
        // return redirect()->route('make.card.payments');
        // return view('payments.success', compact('payment'));
        return view('invoices.payments', compact('payment'));
    }

    public function failed($paymentId = null)
    {
        $payment = $paymentId ? Payment::find($paymentId) : null;
        return view('payments.failed', compact('payment'));
    }
}

```

---

## Step 7: Create Views

### `resources/views/payments/form.blade.php`

```html
<div class="card-body">
    <form method="POST" action="{{ route('pay.order') }}">
        @csrf
        <div class="mb-3">
            <label class="form-label" for="full_name">Full Name</label>
            <input type="text" name="full_name" class="form-control" id="full_name"
                value="{{ old('full_name') }}" placeholder="Full Name" required>
            @error('full_name')
                <div class="text-danger small">{{ $message }}</div>
            @enderror
        </div>

        <div class="mb-3">
            <label class="form-label" for="email">Email</label>
            <input type="email" name="email" class="form-control" id="email"
                value="{{ old('email') }}" placeholder="Email" required>
            @error('email')
                <div class="text-danger small">{{ $message }}</div>
            @enderror
        </div>

        <div class="mb-3">
            <label class="form-label" for="contact">Contact</label>
            <input type="text" name="contact" class="form-control" id="contact"
                value="{{ old('contact') }}" placeholder="10–15 digits" pattern="[0-9]{10,15}"
                required>
            @error('contact')
                <div class="text-danger small">{{ $message }}</div>
            @enderror
        </div>

        <div class="mb-3">
            <label class="form-label" for="amount">Amount (INR)</label>
            <input type="number" name="amount" class="form-control" id="amount"
                value="{{ old('amount', 1000) }}" placeholder="1000" min="1" step="1"
                required>
            @error('amount')
                <div class="text-danger small">{{ $message }}</div>
            @enderror
        </div>

        <button type="submit" class="btn btn-primary">Pay Now</button>
    </form>
</div>

```

### `resources/views/payments/checkout.blade.php`

```html
<!doctype html>
<html>
<head>
    <meta name="csrf-token" content="{{ csrf_token() }}">
    <title>Processing Payment...</title>
</head>
<body>
<script src="https://checkout.razorpay.com/v1/checkout.js"></script>
<script>
(function () {
    const options = {
        key: "{{ $key }}",
        amount: "{{ $order['amount'] }}",
        currency: "{{ $order['currency'] }}",
        name: "{{ config('app.name') }}",
        description: "Payment for Order {{ $order['id'] }}",
        order_id: "{{ $order['id'] }}", // ties payment to server-created order
        prefill: {
            name:   @json($prefill['full_name']),
            email:  @json($prefill['email']),
            contact:@json($prefill['contact']),
        },
        notes: {
            payment_db_id: "{{ $payment->id }}"
        },
        handler: function (response) {
            // Send payment_id, order_id & signature to server for verification
            fetch(@json(route('payment.verify')), {
                method: "POST",
                headers: {
                    "Content-Type": "application/json",
                    "X-CSRF-TOKEN": document.querySelector('meta[name="csrf-token"]').content
                },
                body: JSON.stringify(response)
            })
            .then(r => r.json())
            .then(({redirect_url}) => window.location.href = redirect_url)
            .catch(() => window.location.href = @json(route('payments.failed', ['payment' => $payment->id])));
        },
        modal: {
            ondismiss: function () {
                window.location.href = @json(route('payments.failed', ['payment' => $payment->id]));
            }
        }
    };
    const rzp = new Razorpay(options);
    rzp.on('payment.failed', function () {
        window.location.href = @json(route('payments.failed', ['payment' => $payment->id]));
    });
    rzp.open();
})();
</script>
</body>
</html>

```

### `resources/views/payments/success.blade.php`

```html
<h2>✅ Payment Successful</h2>
<p>Order: {{ $payment->order_id }}</p>
<p>Payment: {{ $payment->payment_id }}</p>
<p>Amount: ₹{{ number_format($payment->amount/100, 2) }}</p>
```

### `resources/views/payments/failed.blade.php`

```html
<h2>❌ Payment Failed or Cancelled</h2>
@if($payment)
    <p>Order: {{ $payment->order_id }}</p>
    <p>Status: {{ $payment->status }}</p>
@endif
<a href="{{ route('pay.form') }}">Try Again</a>
```

---

## Step 8: (Optional but recommended) Webhook for robust reconciliation
Set up at Dashboard → Settings → Webhooks (Test mode first). Verify the X-Razorpay-Signature header with your webhook secret and the raw request body.

1. Create a controller.

```php
php artisan make:controller PaymentWebhookController
```

2. Use the provided `PaymentWebhookController.php` to verify and update payments.

```php
// app/Http/Controllers/PaymentWebhookController.php
namespace App\Http\Controllers;

use App\Models\Payment;
use Illuminate\Http\Request;
use Razorpay\Api\Api;
use Razorpay\Api\Utility;
use Symfony\Component\HttpFoundation\Response;

class PaymentWebhookController extends Controller
{
    public function handle(Request $request)
    {
        $secret    = config('services.razorpay.webhook_secret');
        $signature = $request->header('X-Razorpay-Signature');
        $payload   = $request->getContent(); // raw body is required

        try {
            Utility::verifyWebhookSignature($payload, $signature, $secret); // throws on failure

            $event = $request->input('event');
            $data  = $request->input('payload');

            if ($event === 'payment.captured') {
                $orderId   = $data['payment']['entity']['order_id'] ?? null;
                $paymentId = $data['payment']['entity']['id'] ?? null;
                if ($orderId && $payment = Payment::where('order_id', $orderId)->first()) {
                    $payment->update([
                        'status'     => 'paid',
                        'payment_id' => $paymentId,
                        'payload'    => $request->all(),
                    ]);
                }
            }

            return response()->json(['ok' => true]);
        } catch (\Throwable $e) {
            // log $e->getMessage()
            return response()->json(['ok' => false], Response::HTTP_BAD_REQUEST);
        }
    }
}
```

---

# 7) Test, then go live

1. Keep **Test** API keys in `.env` and try test transactions.
2. Confirm **signature verification passes** and your DB row flips to `paid`. ([Razorpay][2])
3. (Recommended) Enable **Auto Capture** in Dashboard, or capture programmatically as shown. ([Razorpay][2])
4. Add a webhook subscription for `payment.captured` / `order.paid` and verify using ngrok or a webhook testing bin. ([Razorpay][3])
5. Before granting access/benefits, **check status is `captured`/`paid`** (belt-and-suspenders). ([Razorpay][6])
6. Swap to **Live** API keys in `.env` when ready. ([Razorpay][7])
