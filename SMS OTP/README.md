# Laravel SMS OTP Login Integration

This is a simple implementation of SMS-based OTP (One-Time Password) authentication for login in a Laravel application using Twilio as the SMS provider.

## 🚀 Features
- Send OTP to a user’s phone number
- Verify OTP to log the user in
- Automatically create user if not existing
- Simple session-based OTP storage

## 📦 Requirements
- Laravel 9.x or later
- PHP 8.x or later
- Twilio account with API credentials

## ✅ Installation Steps

### 1. Install Twilio SDK
```bash
composer require twilio/sdk
````

### 2. Configure Environment Variables

Add the following to your `.env` file:

```env
TWILIO_SID=your_twilio_sid
TWILIO_TOKEN=your_twilio_auth_token
TWILIO_PHONE=+1234567890
```

### 3. Create `OtpController`

Generate the controller:

```bash
php artisan make:controller OtpController
```

Add the following methods inside `OtpController`:

* `sendOtp()` – Sends an OTP to the provided phone number.
* `verifyOtp()` – Verifies the OTP and logs in the user.

Refer to the example code provided in this repository.
```bash
use Twilio\Rest\Client;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Session;

class OtpController extends Controller
{
    public function sendOtp(Request $request)
    {
        $request->validate(['phone' => 'required']);

        $otp = rand(100000, 999999);  
        Session::put('otp', $otp);
        Session::put('phone', $request->phone);

        $twilio = new Client(env('TWILIO_SID'), env('TWILIO_TOKEN'));
        $twilio->messages->create(
            $request->phone,
            ["from" => env('TWILIO_PHONE'), "body" => "Your OTP is: $otp"]
        );

        return response()->json(['message' => 'OTP sent successfully']);
    }

    public function verifyOtp(Request $request)
    {
        $request->validate(['otp' => 'required']);

        if ($request->otp == Session::get('otp')) {
            // ✅ OTP verified, login user
            $user = User::firstOrCreate(['phone' => Session::get('phone')]);
            auth()->login($user);

            Session::forget(['otp', 'phone']);
            return response()->json(['message' => 'Login successful']);
        }

        return response()->json(['message' => 'Invalid OTP'], 401);
    }
}

```


### 4. Define Routes

Add the routes in `routes/web.php`:

```php
Route::post('/send-otp', [OtpController::class, 'sendOtp']);
Route::post('/verify-otp', [OtpController::class, 'verifyOtp']);
```

### 5. Implement Frontend Forms

Create views to accept phone numbers and OTP input, and send requests to the routes defined above.

---

## 📂 Code Structure

```
app/
├── Http/
│   └── Controllers/
│       └── OtpController.php
routes/
└── web.php
.env
```

---

## ⚙️ Usage

1. User enters their phone number.
2. The backend generates a 6-digit OTP and sends it using Twilio.
3. The user inputs the OTP they received.
4. The backend verifies the OTP and logs the user in.

---

## ✅ Notes

* This is a basic demonstration using session storage for OTP.
* For production, consider:

  * Adding OTP expiration
  * Limiting OTP requests
  * Using Redis or database for storing OTP
  * Handling errors gracefully
  * Enhancing security features

---

## 📖 Further Improvements

* Implement rate limiting for OTP requests
* Add validation rules for phone numbers
* Encrypt sensitive data
* Integrate with other SMS providers like MSG91, Fast2SMS

---

## 📞 Contact

For questions or support, reach out to \[your email/contact info].
