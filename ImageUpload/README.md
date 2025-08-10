# 📸 Updating Profile Image in Laravel

This guide explains how to handle **profile image uploads**, **resizing**, and **saving** using [Intervention Image](https://image.intervention.io/) in Laravel.

---

## 1️⃣ Requirements

Install **Intervention Image** package:

```bash
composer require intervention/image
```

If you are using **Gd driver**:

```php
use Intervention\Image\ImageManager;
use Intervention\Image\Drivers\Gd\Driver;
```

If you prefer **Imagick driver**:

```php
use Intervention\Image\Drivers\Imagick\Driver;
```

---

## 2️⃣ Controller Logic

```php
use Illuminate\Support\Facades\File;
use Intervention\Image\ImageManager;
use Intervention\Image\Drivers\Gd\Driver;


if ($request->hasFile('profile')) {
    // Only if updating profile 
    if ($staff->profile != '') {
        if (File::exists(public_path('uploads/profile/' . $staff->profile))) {
            File::delete(public_path('uploads/profile/' . $staff->profile));
        }

        if (File::exists(public_path('uploads/profile/small/' . $staff->profile))) {
            File::delete(public_path('uploads/profile/small/' . $staff->profile));
        }
    }
    // updating profile end

    $profile = $data['profile'];
    $ext = $profile->getClientOriginalExtension();
    $profileName = time() . '.' . $ext;

    // Store original image
    $profile->move(public_path('uploads/profile'), $profileName);

    // Resize image
    $manager = new ImageManager(Driver::class);
    $image = $manager->read(public_path('uploads/profile/' . $profileName));
    $image->resize(400, 500);

    // Ensure "small" directory exists
    $smallDir = public_path('uploads/profile/small');
    if (!File::exists($smallDir)) {
        File::makeDirectory($smallDir, 0755, true);
    }

    // Save resized image
    $image->save($smallDir . '/' . $profileName);
    $image->save();

    // Save filename in DB
    $data['profile'] = $profileName;
} else {
    // Prevent overwriting profile field with null
    unset($data['profile']);
}
```

---

## 3️⃣ Key Points

* **`hasFile('profile')`**: Checks if a file is uploaded.
* **Unique filename**: Using `time()` to avoid overwriting.
* **Original storage**: First save in `public/uploads/profile/`.
* **Resizing**: Set desired dimensions (`400x500` in this case).
* **Directory check**: Ensure target directory exists before saving resized image.
* **Prevent overwriting**: `unset($data['profile'])` if no new image uploaded.

---

## 4️⃣ Example Form

```html
<form action="{{ route('user.update', $user->id) }}" method="POST" enctype="multipart/form-data">
    @csrf
    @method('PUT')

    <input type="file" name="profile" class="form-control">
    <button type="submit" class="btn btn-primary">Update Profile</button>
</form>
```

---

## 5️⃣ Notes

* Store only **file name** in DB, not the full path.
* Use `asset('uploads/profile/'.$user->profile)` to display images.
* Consider deleting **old images** when replacing with a new one to save storage.
