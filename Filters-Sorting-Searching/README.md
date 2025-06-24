# Laravel Data Listing with Filters, Sorting, and Pagination

This guide shows how to implement data fetching features like **search**, **filtering**, **sorting**, and **pagination** using Laravel’s Eloquent and query builder.

## 📋 Requirements

- PHP >= 8.1
- Laravel >= 10
- MySQL
- Basic CRUD functionality set up

---

## 🎯 Goal

We’ll build a product listing with:

- Text-based search (name/description)
- Price range filter
- Category filter
- Sorting (name, price)
- Pagination

---

## 🧾 Step 1: Database Setup

Assuming `products` table has:

- id
- name
- description
- price
- category
- image
- created_at

If not, update your migration like:

```php
$table->string('category')->nullable();
```

Then run:

```bash
php artisan migrate
```

---

## ⚙️ Step 2: Update Controller Logic

### Update `ProductController@index`

```php
use Illuminate\Http\Request;

public function index(Request $request)
{
    $query = Product::query();

    // Search
    if ($request->filled('search')) {
        $query->where(function ($q) use ($request) {
            $q->where('name', 'like', '%' . $request->search . '%')
              ->orWhere('description', 'like', '%' . $request->search . '%');
        });
    }

    // Price Range Filter
    if ($request->filled('min_price')) {
        $query->where('price', '>=', $request->min_price);
    }

    if ($request->filled('max_price')) {
        $query->where('price', '<=', $request->max_price);
    }

    // Category Filter
    if ($request->filled('category')) {
        $query->where('category', $request->category);
    }

    // Sorting
    $sortField = $request->get('sort_by', 'created_at');
    $sortOrder = $request->get('order', 'desc');

    $query->orderBy($sortField, $sortOrder);

    // Pagination
    $products = $query->paginate(10)->withQueryString();

    // For filters
    $categories = Product::select('category')->distinct()->pluck('category');

    return view('products.index', compact('products', 'categories'));
}
```

---

## 🧾 Step 3: Blade View

### `resources/views/products/index.blade.php`

```blade
<form method="GET" action="{{ route('products.index') }}">
    <input type="text" name="search" placeholder="Search..." value="{{ request('search') }}">

    <input type="number" name="min_price" placeholder="Min Price" value="{{ request('min_price') }}">
    <input type="number" name="max_price" placeholder="Max Price" value="{{ request('max_price') }}">

    <select name="category">
        <option value="">All Categories</option>
        @foreach ($categories as $category)
            <option value="{{ $category }}" {{ request('category') == $category ? 'selected' : '' }}>{{ $category }}</option>
        @endforeach
    </select>

    <select name="sort_by">
        <option value="created_at" {{ request('sort_by') == 'created_at' ? 'selected' : '' }}>Newest</option>
        <option value="name" {{ request('sort_by') == 'name' ? 'selected' : '' }}>Name</option>
        <option value="price" {{ request('sort_by') == 'price' ? 'selected' : '' }}>Price</option>
    </select>

    <select name="order">
        <option value="asc" {{ request('order') == 'asc' ? 'selected' : '' }}>Asc</option>
        <option value="desc" {{ request('order') == 'desc' ? 'selected' : '' }}>Desc</option>
    </select>

    <button type="submit">Filter</button>
    <a href="{{ route('products.index') }}">Reset</a>
</form>

<hr>

@foreach ($products as $product)
    <div>
        <h4>{{ $product->name }}</h4>
        <p>{{ $product->description }}</p>
        <p>₹{{ $product->price }}</p>
        <p>{{ $product->category }}</p>
        @if ($product->image)
            <img src="{{ asset('storage/' . $product->image) }}" width="100" />
        @endif
    </div>
@endforeach

<div>
    {{ $products->links() }}
</div>
```

---

## ✅ Best Practices

- Use `withQueryString()` on pagination to persist filters.
- Sanitize and validate inputs for more robust production usage.
- Use Laravel Form Requests for complex validation.

---

## 📂 Folder Snapshot

```
resources/views/products/
├── index.blade.php

routes/web.php
app/Http/Controllers/ProductController.php
```

---

With this structure, you now have an interactive, user-friendly product listing interface with complete filtering and sorting functionality.
