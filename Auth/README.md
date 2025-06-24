# Laravel Custom Authentication System

This guide will help you build a custom authentication system in Laravel without using Breeze, Jetstream, or Laravel UI.

## 📋 Requirements

- PHP >= 8.1
- Composer
- Laravel >= 10
- MySQL or any DB
- Node.js (optional for frontend)

## ⚙️ Step 1: Laravel Project Setup

```bash
composer create-project laravel/laravel auth-system
cd auth-system
php artisan serve
```

## 🔧 Step 2: Database Configuration

Update `.env` file:

```env
DB_DATABASE=auth_db
DB_USERNAME=root
DB_PASSWORD=
```

Then migrate:

```bash
php artisan migrate
```

## 👤 Step 3: Create AuthController

```bash
php artisan make:controller AuthController
```

## 📁 Step 4: Routes (web.php)

```php
use App\Http\Controllers\AuthController;

Route::get('/register', [AuthController::class, 'showRegisterForm'])->name('register');
Route::post('/register', [AuthController::class, 'register']);

Route::get('/login', [AuthController::class, 'showLoginForm'])->name('login');
Route::post('/login', [AuthController::class, 'login']);

Route::post('/logout', [AuthController::class, 'logout'])->name('logout');

Route::get('/dashboard', function () {
    return view('dashboard');
})->middleware('auth')->name('dashboard');
```

## 🧠 Step 5: AuthController Logic

```php
namespace App\Http\Controllers;

use App\Models\User;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Hash;
use Illuminate\Support\Facades\Auth;

class AuthController extends Controller
{
    public function showRegisterForm()
    {
        return view('auth.register');
    }

    public function register(Request $request)
    {
        $request->validate([
            'name' => 'required|string|max:255',
            'email' => 'required|string|email|unique:users',
            'password' => 'required|string|min:6|confirmed',
        ]);

        User::create([
            'name' => $request->name,
            'email' => $request->email,
            'password' => Hash::make($request->password),
        ]);

        return redirect()->route('login')->with('success', 'Registration successful.');
    }

    public function showLoginForm()
    {
        return view('auth.login');
    }

    public function login(Request $request)
    {
        $credentials = $request->only('email', 'password');

        $request->validate([
            'email' => 'required|email',
            'password' => 'required',
        ]);

        if (Auth::attempt($credentials)) {
            $request->session()->regenerate();
            return redirect()->intended('dashboard');
        }

        return back()->withErrors([
            'email' => 'The provided credentials do not match.',
        ]);
    }

    public function logout(Request $request)
    {
        Auth::logout();
        $request->session()->invalidate();
        $request->session()->regenerateToken();

        return redirect()->route('login');
    }
}
```

## 🗂️ Step 6: Blade Views

### resources/views/auth/register.blade.php

```blade
<!DOCTYPE html>
<html>
<head><title>Register</title></head>
<body>
    <h2>Register</h2>
    <form action="{{ route('register') }}" method="POST">
        @csrf
        <input type="text" name="name" placeholder="Name" required>
        <input type="email" name="email" placeholder="Email" required>
        <input type="password" name="password" placeholder="Password" required>
        <input type="password" name="password_confirmation" placeholder="Confirm Password" required>
        <button type="submit">Register</button>
    </form>
    <a href="{{ route('login') }}">Already have an account?</a>
</body>
</html>
```

### resources/views/auth/login.blade.php

```blade
<!DOCTYPE html>
<html>
<head><title>Login</title></head>
<body>
    <h2>Login</h2>
    @if(session('success'))
        <p>{{ session('success') }}</p>
    @endif
    <form action="{{ route('login') }}" method="POST">
        @csrf
        <input type="email" name="email" placeholder="Email" required>
        <input type="password" name="password" placeholder="Password" required>
        <button type="submit">Login</button>
    </form>
    <a href="{{ route('register') }}">Don't have an account?</a>
</body>
</html>
```

### resources/views/dashboard.blade.php

```blade
<!DOCTYPE html>
<html>
<head><title>Dashboard</title></head>
<body>
    <h2>Welcome, {{ Auth::user()->name }}</h2>
    <form method="POST" action="{{ route('logout') }}">
        @csrf
        <button type="submit">Logout</button>
    </form>
</body>
</html>
```

## ✅ Coding Standards Followed

- PSR-12 Compliant
- Clean validation and redirection flow
- Secure password hashing with Bcrypt
- Mass assignment protection
- Session regeneration on login

## 📌 Optional Enhancements

- Email Verification
- Password Reset
- Social Login
- Rate Limiting

## 📂 Folder Structure Snapshot

```
resources/views/auth/
├── login.blade.php
└── register.blade.php

resources/views/
└── dashboard.blade.php

routes/
└── web.php

app/Http/Controllers/
└── AuthController.php
```
