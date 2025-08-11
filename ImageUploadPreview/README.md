# Profile Image Preview on File Upload

This feature allows users to preview their profile image immediately after selecting a file, without needing to submit the form.  
It works using HTML, JavaScript, and Blade (Laravel).

---

## Features
- Instantly shows the uploaded image before form submission.
- Works with any image format (`jpg`, `jpeg`, `png`, `gif`, etc.).
- Falls back to a default image if no file is selected.
- Memory-efficient — temporary object URLs are revoked after loading.

---

## Usage

### 1. Blade/HTML Code
Add the following HTML snippet to your form:

```html
<div class="mb-3">
    <label for="profile" class="form-label">Profile Image</label>
    <input type="file" name="profile" id="profile" class="form-control" accept="image/*" onchange="previewProfileImage(event)">
</div>

<div class="mb-3">
    <img id="profile-preview" 
         src="{{ asset('uploads/profile/default.png') }}" 
         alt="Profile Preview" 
         style="max-width: 150px; display: none; border-radius: 8px; margin-top: 10px;">
</div>
````

---

### 2. JavaScript Code

Add the following script to your page (before the closing `</body>` tag or in a JS file):

```html
<script>
function previewProfileImage(event) {
    const preview = document.getElementById('profile-preview');
    const file = event.target.files[0];

    if (file) {
        preview.src = URL.createObjectURL(file);
        preview.style.display = 'block';

        // Free memory after image loads
        preview.onload = function() {
            URL.revokeObjectURL(preview.src);
        }
    } else {
        preview.style.display = 'none';
    }
}
</script>
```

---

## How It Works

1. The `<input type="file">` lets users choose an image.
2. The `onchange` event calls the `previewProfileImage` function.
3. JavaScript creates a temporary URL for the file using `URL.createObjectURL(file)`.
4. The preview `<img>` tag updates instantly with the selected image.
5. Once the image loads, the temporary URL is revoked to save memory.

---

## Notes

* Ensure your file input has `accept="image/*"` to limit selection to images.
* Replace `default.png` with your preferred placeholder image.
* Works in all modern browsers.

---

## Example Output

When a user selects an image, it will be shown below the file input:

```
[ Choose File ]   →    [Preview of Selected Image]
```

---

## License

You are free to use and modify this code in your projects.
