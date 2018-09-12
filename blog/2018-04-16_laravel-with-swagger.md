為了方便查看RESTful API的文檔，這篇文章將會使用Laravel搭配Swagger，創建一個根據comment自動產生API文檔的頁面。

我們將會使用Swagger-php和swagger-ui的集合，適用於Laravel 5的[DarkaOnLine/L5-Swagger](//github.com/DarkaOnLine/L5-Swagger)。

## 匯入`DarkaOnLine/L5-Swagger`

首先在Laravel專案裡面匯入DarkaOnLine/L5-Swagger：
```
 $ composer require "darkaonline/l5-swagger:5.6.*"
```

然後發佈L5-Swagger的設定檔案和View檔案：
```
 $ php artisan vendor:publish --provider "L5Swagger\L5SwaggerServiceProvider"
```

## 建立API資訊

現在`app/`裡面隨便一個檔案建立以下comment，用來建立API的資訊：
```
/**
* @SWG\Swagger(
*     schemes={"http","https"},
*     host="http://localhost",
*     basePath="/api",
*     @SWG\Info(
*         version="1.0.0",
*         title="API for TestProject",
*         description="",
*         termsOfService="",
*         @SWG\Contact(
*             email="admin@test.com"
*         )
*     ),
*     @SWG\ExternalDocumentation(
*         description="More document",
*         url="some_url_for_external_document"
*     ),
*     @SWG\Definition(
*    definition="User",
*    type="object",
*       required={"id", "username"},
*       @SWG\Property(property="id", type="string", example="5ab153327b06e20257156b2f"),
*    @SWG\Property(property="name", type="string", example="Chan Tai Man"),
*    @SWG\Property(property="username", type="string", example="chantm2018"),
*    @SWG\Property(property="email", type="string", example="chantm2018@test.com"),
*    @SWG\Property(property="password", type="string", example="********"),
*   ),
* )
*/
```

然後舉例在Controller裡面有著名為`hi`的method：
```
  /**
  * @SWG\Get(
  *   path="/hi",
  *   summary="Get a hi from server",
  *   @SWG\Response(
  *     response=200,
  *     description="Hi!",
  *     @SWG\Schema(
  *          type="string",
  *          ="test user, hi!",
  *     )
  *   ),
  public function hi(Request $request) {
    $user = Auth::user();
    return $user->username . ", hi!";
  }
```

有關以上Annotation的語法，可以參考[zircote/swagger-php](//github.com/zircote/swagger-php)的文檔

最後編寫完成文檔後，需要利用以下命令在`storage/api-docs`生成`api-docs.json`:
``` $ php artisan l5-swagger:generate
```

然後前往`http://host_url/api/documentation`，就能看到API文檔。

有關文檔的設定，可以去`config/l5-swagger.php`進行修改。
