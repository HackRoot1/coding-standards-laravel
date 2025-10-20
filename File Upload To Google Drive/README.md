# 📁 File Uploads to Google Drive in Laravel

This guide explains how to upload files to **Google Drive** using the `yaza/laravel-google-drive-storage` package in a Laravel project.

---

## ✅ Step 1: Install the Package

Run the following command:

```bash
composer require yaza/laravel-google-drive-storage
```

---

## ✅ Step 2: Add Credentials to `.env`

Add the following environment variables to your `.env` file:

```env
FILESYSTEM_CLOUD=google

GOOGLE_DRIVE_CLIENT_ID=your-google-client-id
GOOGLE_DRIVE_CLIENT_SECRET=your-google-client-secret
GOOGLE_DRIVE_REFRESH_TOKEN=your-refresh-token
GOOGLE_DRIVE_ACCESS_TOKEN=your-access-token   # optional
GOOGLE_DRIVE_FOLDER=your-folder-name-in-drive
```

> ⚠️ **Important:** Never commit real credentials to GitHub or share them publicly.

---

## ✅ Step 3: Configure the Disk in `config/filesystems.php`

Add the following configuration under the `disks` array:

```php
'disks' => [

    // ... other disks

    'google' => [
        'driver' => 'google',
        'clientId' => env('GOOGLE_DRIVE_CLIENT_ID'),
        'clientSecret' => env('GOOGLE_DRIVE_CLIENT_SECRET'),
        'accessToken' => env('GOOGLE_DRIVE_ACCESS_TOKEN'), // optional
        'refreshToken' => env('GOOGLE_DRIVE_REFRESH_TOKEN'),
        'folder' => env('GOOGLE_DRIVE_FOLDER'),
    ],

],
```

---

## ✅ Step 4: Generate Client ID & Refresh Token (GCP Setup)

You must create OAuth credentials in **Google Cloud Platform** and retrieve a refresh token.

### 📌 Required Guides:

* **Create Client ID & Secret**
  [https://github.com/ivanvermeyen/laravel-google-drive-demo/blob/master/README/1-getting-your-dlient-id-and-secret.md](https://github.com/ivanvermeyen/laravel-google-drive-demo/blob/master/README/1-getting-your-dlient-id-and-secret.md)

* **Generate Refresh Token**
  [https://github.com/ivanvermeyen/laravel-google-drive-demo/blob/master/README/2-getting-your-refresh-token.md](https://github.com/ivanvermeyen/laravel-google-drive-demo/blob/master/README/2-getting-your-refresh-token.md)

---

## ✅ Step 5: Upload File to Google Drive

Create a function in your controller to store the file:

```php
/**
 * Function: uploadFileToDrive
 * Description: Upload a file to Google Drive
 */
public function uploadFileToDrive(Request $request)
{
    $file = $request->file('file');
    $fileName = $file->getClientOriginalName();

    $response = Storage::disk('google')->put($fileName, File::get($file));

    dd($response); // returns true if file uploaded successfully
}
```

### ✅ Example Route

```php
Route::post('/upload-to-drive', [YourController::class, 'uploadFileToDrive']);
```

### ✅ Example HTML Form

```html
<form action="/upload-to-drive" method="POST" enctype="multipart/form-data">
    @csrf
    <input type="file" name="file" required>
    <button type="submit">Upload to Drive</button>
</form>
```

---

## ✅ Package Reference

Refer to the full package documentation:
[https://github.com/yaza-putu/laravel-google-drive-storage](https://github.com/yaza-putu/laravel-google-drive-storage)

---

## ✅ Notes

* Use `FILESYSTEM_CLOUD=google` to make it your default cloud storage.
* Make sure the Google API credentials have access to the Drive.
* Use the correct folder name or leave `GOOGLE_DRIVE_FOLDER` empty to upload to the root.
