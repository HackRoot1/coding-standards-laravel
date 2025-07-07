

# This is for all input fields validation: 


## Following is an example of Controller:
```
use Illuminate\Support\Facades\Validator;

$requestData = $request->all();

$validator = Validator::make($requestData, [
    'name'              => 'required|string|max:255',
    'email'             => 'required|email|unique:users,email',
    'password'          => 'required|string|min:6|confirmed',
    'phone'             => 'nullable|digits:10',
    'age'               => 'nullable|integer|min:18|max:100',
    'bio'               => 'nullable|string|max:500',
    'is_active'         => 'nullable|boolean',
    'gender'            => 'required|in:male,female,other',
    'role_id'           => 'required|exists:roles,id',
    'profile_picture'   => 'nullable|image|mimes:jpeg,png,jpg|max:2048',
    'website'           => 'nullable|url',
    'dob'               => 'nullable|date|before:today',
    'appointment_time'  => 'nullable|date_format:H:i',
    'color_preference'  => 'nullable|string',
    'range_input'       => 'nullable|numeric|min:1|max:10',
    'hidden_token'      => 'nullable|string',
    'terms'             => 'accepted', // For checkbox
]);
```
