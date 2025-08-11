# Laravel Activity Logs

This guide explains how to set up and use **Spatie's Laravel Activitylog** package to track changes to your models and log activity in your Laravel application.

---

## 📦 Installation

1. **Install the package via Composer**
```bash
composer require spatie/laravel-activitylog
````

2. **Clear the configuration cache**
   *(Ensures new config values are loaded)*

```bash
php artisan config:clear
```

3. **Publish the migrations**

```bash
php artisan vendor:publish --provider="Spatie\Activitylog\ActivitylogServiceProvider" --tag="activitylog-migrations"
```

4. **Run the migrations**

```bash
php artisan migrate
```

---

## ⚙️ Usage Example

To log activities for a model (e.g., `User` model):

### 1. **Open the model file**

`app/Models/User.php`

### 2. **Import the required trait**

```php
use Spatie\Activitylog\Traits\LogsActivity;
use Spatie\Activitylog\LogOptions;
```

### 3. **Add the trait to the model**

```php
use HasFactory, Notifiable, HasRolesAndPermissions, LogsActivity;
```

### 4. **Configure logging behavior**

```php
public function getActivitylogOptions(): LogOptions
{
    return LogOptions::defaults()
        ->logAll() // 🔹 Logs every attribute change
        ->logOnlyDirty(); // 🔹 Only logs changes, not unchanged values
}

public function getDescriptionForEvent(string $eventName): string
{
    return "User has been {$eventName}";
}
```

---

## 📝 Example Activity Log Entry

When you update a user’s email, an activity record will be stored in the database like:

| id | log\_name | description           | subject\_id | subject\_type   | causer\_id | causer\_type     | properties                                                             | created\_at         |
| -- | --------- | --------------------- | ----------- | --------------- | ---------- | ---------------- | ---------------------------------------------------------------------- | ------------------- |
| 1  | default   | User has been updated | 5           | App\Models\User | 1          | App\Models\Admin | {"attributes": {"email": "[new@example.com](mailto:new@example.com)"}} | 2025-08-11 10:00:00 |

---

## 📖 Additional Options

* `logOnly(['name', 'email'])` → Logs only specified attributes.
* `dontSubmitEmptyLogs()` → Prevents empty log entries when no changes occur.
* You can customize the `log_name` for different models by calling:

```php
return LogOptions::defaults()->useLogName('user_logs');
```

---

## 📂 Log Storage

The activity logs are stored in the `activity_log` table by default.
You can query them via the `Spatie\Activitylog\Models\Activity` model.

Example:

```php
use Spatie\Activitylog\Models\Activity;

$activities = Activity::all();
```

---

## ✅ Best Practices

* Use `logOnlyDirty()` to avoid duplicate log entries when no changes occur.
* Assign different `log_name` values for different models to make filtering easier.
* Regularly clear old logs if not required for compliance:

```bash
php artisan activitylog:clean
```

---

## 📚 References

* [Spatie Laravel Activitylog Documentation](https://spatie.be/docs/laravel-activitylog)

