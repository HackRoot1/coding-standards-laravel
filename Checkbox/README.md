# How to Store and Retrieve Checkbox Values in Laravel

This guide explains how to store multiple checkbox values in the database as an array (JSON column) and retrieve them for display and editing.

---

## 1️⃣ Database Setup

Make sure the column where you want to store the checkbox values can handle JSON.  
Example migration:

```php
Schema::table('users', function (Blueprint $table) {
    $table->json('skills')->nullable();
});
````

---

## 2️⃣ Model Configuration

In your `User` model, cast the column to an array:

```php
protected function casts(): array
{
    return [
        'skills' => 'array',
    ];
}
```

---

## 3️⃣ HTML Form (Create View)

Example: Suppose `skills` array can contain `html`, `css`, and `js`.

```blade
@foreach (['html', 'css', 'js'] as $skill)
    <div class="col-4">
        <label class="form-check">
            <input name="skills[]" value="{{ $skill }}" type="checkbox"
                class="form-check-input @error('skills') is-invalid @enderror"
                {{ is_array(old('skills')) && in_array($skill, old('skills')) ? 'checked' : '' }}>
            <span class="form-check-label">{{ strtoupper($skill) }}</span>
        </label>
    </div>
@endforeach
```

---

## 4️⃣ Controller (Store Data)

After form submission, store the data as normal:

```php
$user = User::create([
    'skills' => $validated['skills'],
]);
```

---

## 5️⃣ Display Skills (Show View)

Example: `$staff` contains a `User` instance.

```blade
@forelse ($staff->skills as $skill)
    <span class="badge bg-secondary">{{ strtoupper($skill) }}</span>
@empty
    <span>No skills listed</span>
@endforelse
```

---

## 6️⃣ Edit Form (Retrieve Data)

When editing, pre-check the selected checkboxes:

```blade
@foreach (['html', 'css', 'js'] as $skill)
    <div class="col-4">
        <label class="form-check">
            <input name="skills[]" value="{{ $skill }}" type="checkbox"
                class="form-check-input @error('skills') is-invalid @enderror"
                {{ in_array($skill, $staff->skills, true) ? 'checked' : '' }}>
            <span class="form-check-label">{{ strtoupper($skill) }}</span>
        </label>
    </div>
@endforeach
```

---

✅ **Summary**

* Use **`json` column** in the DB to store checkbox values.
* Cast to `array` in the model for automatic serialization/deserialization.
* Use `old()` for create forms, and array checks for edit forms.
* Display values as badges or any style you prefer.
