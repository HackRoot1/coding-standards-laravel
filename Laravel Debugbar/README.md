# Laravel Debugbar

The **[Laravel Debugbar](https://github.com/barryvdh/laravel-debugbar)** adds a debug toolbar to your application for performance monitoring, query analysis, and debugging during development.

---

## **Installation**

Install the package (development only):

```bash
composer require barryvdh/laravel-debugbar --dev
````

---

## **Configuration**

Publish the configuration file (optional):

```bash
php artisan vendor:publish --provider="Barryvdh\Debugbar\ServiceProvider"
```

This will create:

```
config/debugbar.php
```

---

## **Enable / Disable Debugbar**

Debugbar is controlled via the `.env` file:

```env
DEBUGBAR_ENABLED=true
```

Disable in production:

```env
DEBUGBAR_ENABLED=false
```

---

## **Features**

Once installed, you will see a toolbar at the bottom of your pages showing:

* Execution time & memory usage
* Database queries (with bindings & execution time)
* Route information
* Log messages
* Authentication details
* Dumps & exceptions

---

## **Logging Custom Messages**

You can send your own data to the Debugbar:

```php
\Debugbar::info($user);
\Debugbar::error('Something went wrong!');
\Debugbar::warning('Heads up!');
\Debugbar::addMessage('Custom message', 'label');
```

---

## **Performance Monitoring**

Use Debugbar to identify:

* Slow queries
* High memory usage
* Extra or unnecessary database calls

---

## **Uninstalling**

If you no longer need Debugbar:

```bash
composer remove barryvdh/laravel-debugbar
```

---

## **Best Practice**

* Always keep Debugbar **disabled in production**.
* Combine it with [Laravel Query Detector] for N+1 query detection.

