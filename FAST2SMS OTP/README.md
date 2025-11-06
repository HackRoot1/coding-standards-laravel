# Using package setup

# Requirements 

1. php version >= 8.3

## Step 1: Install Laravel Fast2SMS PackageUse the itxshakil/laravel-fast2sms package which simplifies Fast2SMS integration with DLT:

```
composer require itxshakil/laravel-fast2sms
```

## Step 2: Add credentials 
Add your Fast2SMS API key and default sender ID in .env file:

```
FAST2SMS_API_KEY="your_api_key_from_fast2sms"
FAST2SMS_DEFAULT_SENDER_ID="YourSenderID"  # must match DLT-approved sender ID
```

## Step 3: Configure Fast2SMS in LaravelPublish the package configuration (optional):

```
php artisan vendor:publish --provider="ShakilFast2smsServiceProvider"
```

## Step 4: Use the Fast2sms facade in your controller or service for sending OTP using DLT-compliant API:

```
use ShakilFast2smsFacadesFast2sms;

// Method to send OTP
public function sendOtp($phoneNumber, $otp)
{
    return Fast2sms::dlt(
        numbers: $phoneNumber,
        templateId: 'your_dlt_template_id',    // from DLT-approved template
        variablesValues: [$otp],                 // OTP value to replace in template
        senderId: env('FAST2SMS_DEFAULT_SENDER_ID')  // your registered sender ID
    );
}
```

## Step 5: Build Login with Phone Number and OTP WorkflowCreate a user table with phone number column or use existing.Endpoint 1: User enters phone number for login request.Generate OTP (e.g. 6-digit random).Save OTP and phone number in session or database with expiry time.Call sendOtp method to send OTP SMS.Endpoint 2: User submits OTP.Verify OTP against saved OTP for that phone number.If valid, create user session or generate auth token and log the user in.If invalid or expired, return error.

## Step 6: Implement Backend Logic (Example)

```
// Controller method for sending OTP
public function requestOtp(Request $request)
{
    $request->validate(['phone' => 'required|digits:10']);
    $otp = rand(100000, 999999);
    session(['otp' => $otp, 'phone' => $request->phone, 'otp_expires' => now()->addMinutes(5)]);

    Fast2sms::dlt(
        numbers: $request->phone,
        templateId: 'your_dlt_template_id',
        variablesValues: [$otp],
        senderId: env('FAST2SMS_DEFAULT_SENDER_ID')
    );

    return response()->json(['message' => 'OTP sent successfully']);
}

// Controller method to verify OTP
public function verifyOtp(Request $request)
{
    $request->validate(['phone' => 'required|digits:10', 'otp' => 'required|digits:6']);
    if (session('otp') == $request->otp && session('phone') == $request->phone && now()->lt(session('otp_expires'))) {
        // OTP valid, log user in
        // Fetch or create user and generate token/session
        return response()->json(['message' => 'OTP verified successfully']);
    }
    return response()->json(['message' => 'Invalid or expired OTP'], 401);
}
```


## Step 7: Test Endpoints and DeploymentTest with real phone numbers and validate OTP flow.Ensure all DLT fields like entity ID, template ID, and sender ID match the registered details.Deploy your Laravel app and ensure environment variables are set correctly.
