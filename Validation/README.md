

# This is for all input fields validation: 


## Following is an example of Controller:


- Controller File:
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



- Blade template: 
```
<form method="POST" enctype="multipart/form-data">
    @csrf
    <input type="text" name="name">
    <input type="email" name="email">
    <input type="password" name="password">
    <input type="password" name="password_confirmation">
    <input type="tel" name="phone">
    <input type="number" name="age">
    <textarea name="bio"></textarea>
    <input type="checkbox" name="is_active" value="1">
    <select name="gender">
        <option value="male">Male</option>
        <option value="female">Female</option>
        <option value="other">Other</option>
    </select>
    <select name="role_id">
        <option value="1">Admin</option>
        <option value="2">User</option>
    </select>
    <input type="file" name="profile_picture">
    <input type="url" name="website">
    <input type="date" name="dob">
    <input type="time" name="appointment_time">
    <input type="color" name="color_preference">
    <input type="range" name="range_input" min="1" max="10">
    <input type="hidden" name="hidden_token" value="12345">
    <input type="checkbox" name="terms" value="1"> I agree
    <button type="submit">Submit</button>
</form>

```
