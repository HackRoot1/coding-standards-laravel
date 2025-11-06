# 📦 Laravel Fast2SMS Integration (DLT Compliant)

This guide explains how to integrate **Fast2SMS** with your Laravel application using the [`itxshakil/laravel-fast2sms`](https://github.com/itxshakil/laravel-fast2sms) package.  
It allows you to send OTP messages and notifications using **DLT-approved templates** easily.

---

## 🧾 Requirements

- **PHP version:** `>= 8.3`
- **Laravel version:** `>= 10.x`
- **Fast2SMS Account** with DLT approval (Entity ID, Sender ID, and Template ID)

---

## ⚙️ Step 1: Install Laravel Fast2SMS Package

Install the package via Composer:

```bash
composer require itxshakil/laravel-fast2sms
```

---

## 🔑 Step 2: Add Fast2SMS Credentials

Add your Fast2SMS API key and default sender ID in the `.env` file:

```env
FAST2SMS_API_KEY="your_api_key_from_fast2sms"
FAST2SMS_DEFAULT_SENDER_ID="YourSenderID" # Must match DLT-approved Sender ID
```

---

## 🧩 Step 3: Configure Fast2SMS in Laravel (Optional)

Publish the configuration file if you want to customize it:

```bash
php artisan vendor:publish --provider="Shakil\Fast2sms\ServiceProvider"
```

---

## 🚀 Step 4: Send OTP Using Fast2SMS Facade

Use the `Fast2sms` facade in your controller or service class to send OTP messages via DLT-compliant templates.

```php
use Shakil\Fast2sms\Facades\Fast2sms;

public function sendOtp($phoneNumber, $otp)
{
    return Fast2sms::dlt(
        numbers: $phoneNumber,
        templateId: 'your_dlt_template_id', // DLT-approved template ID
        variablesValues: [$otp],             // OTP value for template replacement
        senderId: env('FAST2SMS_DEFAULT_SENDER_ID') // Your registered Sender ID
    );
}
```

---

## 📱 Step 5: Implement Login with Phone Number & OTP Workflow

### **Database**
- Add a `phone` column in your `users` table (or use an existing one).

### **Endpoint 1: Request OTP**
1. User enters their phone number.  
2. Generate a random OTP (e.g., 6 digits).  
3. Save OTP and phone number in session or database (with expiry).  
4. Call `sendOtp()` method to send the OTP SMS.

### **Endpoint 2: Verify OTP**
1. User submits the OTP.  
2. Verify it against the saved OTP for that phone number.  
3. If valid → log the user in or generate an auth token.  
4. If invalid or expired → return an error response.

---

## 🧠 Step 6: Example Controller Methods

```php
use Illuminate\Http\Request;
use Shakil\Fast2sms\Facades\Fast2sms;

class OtpController extends Controller
{
    // Request OTP
    public function requestOtp(Request $request)
    {
        $request->validate(['phone' => 'required|digits:10']);

        $otp = rand(100000, 999999);

        session([
            'otp' => $otp,
            'phone' => $request->phone,
            'otp_expires' => now()->addMinutes(5),
        ]);

        Fast2sms::dlt(
            numbers: $request->phone,
            templateId: 'your_dlt_template_id',
            variablesValues: [$otp],
            senderId: env('FAST2SMS_DEFAULT_SENDER_ID')
        );

        return response()->json(['message' => 'OTP sent successfully']);
    }

    // Verify OTP
    public function verifyOtp(Request $request)
    {
        $request->validate([
            'phone' => 'required|digits:10',
            'otp' => 'required|digits:6',
        ]);

        if (
            session('otp') == $request->otp &&
            session('phone') == $request->phone &&
            now()->lt(session('otp_expires'))
        ) {
            // OTP verified successfully — log user in or generate token
            return response()->json(['message' => 'OTP verified successfully']);
        }

        return response()->json(['message' => 'Invalid or expired OTP'], 401);
    }
}
```

---

## 🧪 Step 7: Testing & Deployment

1. Test using real phone numbers and verify OTP delivery.  
2. Ensure **DLT details** (Entity ID, Template ID, Sender ID) match your Fast2SMS account.  
3. Double-check `.env` credentials before deploying.  
4. Deploy your Laravel app and confirm SMS sending works in production.

---

## ✅ Summary

| Step | Action |
|------|--------|
| 1 | Install Fast2SMS package |
| 2 | Configure `.env` credentials |
| 3 | (Optional) Publish config |
| 4 | Send OTP using facade |
| 5 | Build phone login & OTP verify endpoints |
| 6 | Test and deploy |

---

## 💡 Pro Tip

For better security:
- Store OTPs in a database with expiry timestamps instead of sessions in production.
- Limit OTP requests per number to prevent abuse (rate limiting).
- Use Laravel’s notification system for scalable SMS handling.