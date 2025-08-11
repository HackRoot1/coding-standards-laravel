# Laravel N+1 Query Detector

This package helps detect **N+1 query issues** in your Laravel application during development.  
It alerts you when your code executes unnecessary repeated database queries.

---

## **Installation**

Run the following command to install the package (for development only):

```bash
composer require beyondcode/laravel-query-detector --dev
````

---

## **Configuration**

Publish the configuration file:

```bash
php artisan vendor:publish --provider="BeyondCode\QueryDetector\QueryDetectorServiceProvider"
```

This will create a config file at:

```
config/querydetector.php
```

---

## **Usage**

The N+1 Query Detector runs automatically when `APP_DEBUG=true` in your `.env` file.
When it detects an N+1 query, it will log or display a warning (depending on your config).

---

## **Configuration Options**

Inside `config/querydetector.php`, you can set:

* **Enabled**: Turn the detector on/off.
* **Output**: Decide where to show alerts (`debugbar`, `log`, `alert`, etc.).
* **Except**: Ignore specific classes or methods.

Example:

```php
'enabled' => env('APP_DEBUG', false),

'output' => [
    BeyondCode\QueryDetector\Outputs\Alert::class,
    BeyondCode\QueryDetector\Outputs\Log::class,
],

'ignore' => [
    // Add classes or methods to ignore
],
```

---

## **Tips to Avoid N+1 Queries**

* Use **Eager Loading**:

```php
$users = User::with('posts')->get();
```

instead of:

```php
$users = User::all();
foreach ($users as $user) {
    echo $user->posts;
}
```

* Monitor the Laravel Debugbar or logs for query counts.
* Refactor code when you see repeated queries.

---

## **Uninstalling**

If you no longer need it:

```bash
composer remove beyondcode/laravel-query-detector
```

