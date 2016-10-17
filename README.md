# laravel_upload_multiple_image_with_thumb
實現多個檔案上傳處理兼縮圖，且自動移除 temp file

本程式碼使用 intervention/image 套件

安裝步驟：

在安裝目錄輸入：
```
composer require intervention/image
php artisan cache:clear
```

在 config/app.php 的 $providers 下增加：

```
Intervention\Image\ImageServiceProvider::class,
```
在 config/app.php 的 $aliases 下增加：

```
'Image' => Intervention\Image\Facades\Image::class,
```

在 Controller 下 ```use Image;```

參考我的 Controller 範例去更改成自己需要的

範例 HTML：
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>
<body>
    <form action="/test" method="POST" enctype="multipart/form-data">
        Send these files:<br /><br />
        <input name="photofile[]" type="file" /><br /><br />
        <input name="photofile[]" type="file" /><br /><br />
        <input name="photofile[]" type="file" /><br /><br />
        {{ csrf_field() }}
        <input type="submit" value="Send files" />
    </form>
</body>
</html>
```

範例 Controller：
```
<?php
namespace App\Http\Controllers;
use Illuminate\Http\Request;
use App\Http\Requests;
use Input;
use Image;

class PhotoController extends Controller
{
    public function getUploadPage()
    {
        return view('test.test');
    }
    public function uploadImage(Request $request)
    {
        $maxHeight = 480;
        $maxWidth = 640;
        $maxThumbHeight = 75;
        $maxThumbWidth = 100;
        if (!file_exists(public_path('uploads'))) {
            mkdir(public_path('uploads'), 0777);
            mkdir(public_path('uploads/thumb'), 0777);
        }

        $fileNameArray = [];

        foreach (Input::file('photofile') as $file) {
            $newFileName = md5(strval(time()).str_random(5).$file->getClientOriginalName()).'.'.$file->getClientOriginalExtension();
            $fileNameArray[] = $newFileName;

            $location = $file->getPath().'/'.$file->getFileName();

            // 製作大圖
            $image = Image::make($file);
            if ($image->width() > $maxWidth || $image->height() > $maxHeight) {
                $image->resize($maxWidth, null, function ($constraint) {
                    $constraint->aspectRatio();
                });
            }
            if ($image->height() > $maxWidth) {
                $image->resize(null, $maxHeight, function ($constraint) {
                    $constraint->aspectRatio();
                });
            }
            $image->save(public_path('uploads/') . $newFileName);
            
            // 製作縮圖
            $image = Image::make($file);
            if ($image->width() > $maxThumbWidth || $image->height() > $maxThumbHeight) {
                $image->resize($maxThumbWidth, null, function ($constraint) {
                    $constraint->aspectRatio();
                });
            }
            if ($image->height() > $maxThumbWidth) {
                $image->resize(null, $maxThumbHeight, function ($constraint) {
                    $constraint->aspectRatio();
                });
            }
            $image->save(public_path('uploads/thumb/') . $newFileName);

            \File::delete($location);
        }

        return $fileNameArray;
    }
}

```
