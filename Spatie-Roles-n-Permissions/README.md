# Laravel Roles & Permissions with Super Admin (Spatie)

## 1️⃣ Installation

```bash
composer require spatie/laravel-permission
```

## 2️⃣ Publish Config + Migrations

```bash
php artisan vendor:publish --provider="Spatie\Permission\PermissionServiceProvider"
```

## 3️⃣ Clear Cache & Migrate

```bash
php artisan optimize:clear
php artisan migrate
```

## 4️⃣ Add Trait to `User` Model

```php
use Spatie\Permission\Traits\HasRoles;

class User extends Authenticatable
{
    use Notifiable, HasRoles;
}
```

## 5️⃣ Define Super Admin (Bypass Permissions)

Add in `App\Providers\AuthServiceProvider`:

```php
public function boot(): void
{
    $this->registerPolicies();

    Gate::before(function ($user, string $ability) {
        return $user->hasRole('Super Admin') ? true : null;
    });
}
```

## 6️⃣ Use in Routes

```php
Route::middleware(['auth', 'role:admin|superadmin'])->group(function () {
    Route::get('/staff', [StaffController::class, 'index'])->name('staff.index');
});

Route::delete('/staff/{user}', [StaffController::class, 'destroy'])
    ->middleware(['auth', 'permission:delete-staff'])
    ->name('staff.destroy');
```

## 7️⃣ Use in Controllers

```php
if (auth()->user()->can('update-staff')) {
    // Allowed (Super Admin will always pass)
}
```

## 8️⃣ Use in Blade Files

```blade
@can('View Staff')
  <a href="{{ route('staff.show', $staff->id) }}" class="btn btn-primary btn-sm">View</a>
@endcan

@can('Update Staff')
  <a href="{{ route('staff.edit', $staff->id) }}" class="btn btn-primary btn-sm">Edit</a>
@endcan

@can('Delete Staff')
  <form method="POST" action="{{ route('staff.destroy', $staff->id) }}" style="display:inline-block;">
    @csrf @method('DELETE')
    <button class="btn btn-danger btn-sm">Delete</button>
  </form>
@endcan
```
