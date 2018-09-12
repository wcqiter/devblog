# File Download Through AJAX request (Laravel/Lumen + Vue.js)

This tutorial does not cover upload of files. This tutorial only show how to download files from laravel/lumen server, which the files are located in `storage` folder.

Dependencies:
php: >=7.1.3
laravel/lumen: 5.6.*
vue: ^2.5.1,
axios: ^0.18.0

## Backend

Add a controller for file download request, i.e. `FileController.php` in `app/Http/Controllers`:
```
<?php

namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Storage;

class FileController extends Controller
{
    public function __construct()
    {

    }

    // Required a query string called path in get request to indicate the path of file in storage folder, for example "image/myPic.jpg"
    public function download(Request $request){
        $path = $request->query('path');
        $file = Storage::disk('public')->getDriver()->getAdapter()->applyPathPrefix($path);
        return response()->download($file);
    }
}

```

Add route for the controller method in `web.php` in `route`:
```
$router->get('/file', 'FileController@download');
```

## Frontend

Add a method in your component to be the event handler for download file:
```
    onClickDownload: function() {
        axios({
    		method: 'get',
    		url: apiUrl + '/file?path=' + path,  // apiUrl is the server api url and path is the path of file to download in server
    		responseType: 'arraybuffer' // Important attribute for response to create a blob for download
    	}).then(res => {
            let blob = new Blob([res.data], { type: mimetype }); // Create a blob, mimetype is a variable to indicate the type of file, such as "text/plain" for json files
			let link = document.createElement('a');
			link.href = window.URL.createObjectURL(blob);
			link.download = "myPic.jpg";
			link.click();
        })
        .catch(err => {
            console.log(err.message)
        });
    },

```
