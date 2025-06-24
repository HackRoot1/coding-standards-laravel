# Laravel CRUD Application with File Upload

This guide walks you through building a complete **CRUD (Create, Read, Update, Delete)** application in Laravel with image/file uploading support.

## 📋 Requirements

- PHP >= 8.1
- Composer
- Laravel >= 10
- MySQL
- Node.js & NPM (optional for frontend assets)

## ⚙️ Step 1: Create Laravel Project

```bash
composer create-project laravel/laravel laravel-crud
cd laravel-crud
php artisan serve
```

## 🔧 Step 2: Configure Database

Update `.env` file:

```env
DB_DATABASE=crud_db
DB_USERNAME=root
DB_PASSWORD=
```

Then run:

```bash
php artisan migrate
```

## 🧾 Step 3: Create Model, Migration, and Controller

```bash
php artisan make:model Product -mc
```

### Migration Example

Edit `database/migrations/xxxx_create_products_table.php`:

```php
public function up()
{
    Schema::create('products', function (Blueprint $table) {
        $table->id();
        $table->string('name');
        $table->text('description');
        $table->decimal('price', 8, 2);
        $table->string('image')->nullable();
        $table->timestamps();
    });
}
```

Then migrate:

```bash
php artisan migrate
```

## 🧠 Step 4: ProductController Logic

```php
namespace App\Http\Controllers;

use App\Models\Product;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Storage;

class ProductController extends Controller
{
    public function index()
    {
        $products = Product::all();
        return view('products.index', compact('products'));
    }

    public function create()
    {
        return view('products.create');
    }

    public function store(Request $request)
    {
        $request->validate([
            'name' => 'required',
            'description' => 'required',
            'price' => 'required|numeric',
            'image' => 'nullable|image|mimes:jpg,jpeg,png|max:2048'
        ]);

        $path = $request->file('image')?->store('products', 'public');

        Product::create([
            'name' => $request->name,
            'description' => $request->description,
            'price' => $request->price,
            'image' => $path
        ]);

        return redirect()->route('products.index')->with('success', 'Product created successfully.');
    }

    public function edit(Product $product)
    {
        return view('products.edit', compact('product'));
    }

    public function update(Request $request, Product $product)
    {
        $request->validate([
            'name' => 'required',
            'description' => 'required',
            'price' => 'required|numeric',
            'image' => 'nullable|image|mimes:jpg,jpeg,png|max:2048'
        ]);

        if ($request->hasFile('image')) {
            Storage::disk('public')->delete($product->image);
            $product->image = $request->file('image')->store('products', 'public');
        }

        $product->update($request->only(['name', 'description', 'price', 'image']));

        return redirect()->route('products.index')->with('success', 'Product updated successfully.');
    }

    public function destroy(Product $product)
    {
        Storage::disk('public')->delete($product->image);
        $product->delete();

        return redirect()->route('products.index')->with('success', 'Product deleted successfully.');
    }
}
```

## 📁 Step 5: Add Routes

Update `routes/web.php`:

```php
use App\Http\Controllers\ProductController;

Route::resource('products', ProductController::class);
```

## 🖼️ Step 6: Views (Blade Templates)

### `resources/views/products/index.blade.php`

```blade
<a href="{{ route('products.create') }}">Add New Product</a>

@foreach ($products as $product)
    <div>
        <h3>{{ $product->name }}</h3>
        <p>{{ $product->description }}</p>
        <p>${{ $product->price }}</p>
        @if ($product->image)
            <img src="{{ asset('storage/' . $product->image) }}" width="100" />
        @endif
        <a href="{{ route('products.edit', $product->id) }}">Edit</a>
        <form action="{{ route('products.destroy', $product->id) }}" method="POST">
            @csrf @method('DELETE')
            <button type="submit">Delete</button>
        </form>
    </div>
@endforeach
```

### `resources/views/products/create.blade.php`

```blade
<form method="POST" action="{{ route('products.store') }}" enctype="multipart/form-data">
    @csrf
    <input type="text" name="name" placeholder="Name" required>
    <textarea name="description" placeholder="Description" required></textarea>
    <input type="number" step="0.01" name="price" placeholder="Price" required>
    <input type="file" name="image">
    <button type="submit">Create</button>
</form>
```

### `resources/views/products/edit.blade.php`

```blade
<form method="POST" action="{{ route('products.update', $product->id) }}" enctype="multipart/form-data">
    @csrf @method('PUT')
    <input type="text" name="name" value="{{ $product->name }}" required>
    <textarea name="description" required>{{ $product->description }}</textarea>
    <input type="number" step="0.01" name="price" value="{{ $product->price }}" required>
    <input type="file" name="image">
    <button type="submit">Update</button>
</form>
```

## 📦 Step 7: File Storage Setup

Link the storage:

```bash
php artisan storage:link
```

## ✅ Best Practices

- PSR-12 Compliant
- Input validation using `validate()`
- File handling via Laravel Storage
- RESTful routes and controller structure

## 📂 Folder Structure

```
resources/views/products/
├── index.blade.php
├── create.blade.php
└── edit.blade.php

routes/web.php
app/Http/Controllers/ProductController.php
```

---

This completes your basic CRUD app with image upload. You can enhance it by adding pagination, soft deletes, search filters, etc.
