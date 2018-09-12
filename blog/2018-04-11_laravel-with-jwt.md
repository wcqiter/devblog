基於最近開發的系統中捨棄了session這種傳統的authentication，使用了我沒有接觸過的JWT進行認證，因此在這兒記錄下來。

JWT全名是JSON Web Token，是一個基於JSON的公開標準，用來創建訪問的token。

我們使用[firebase/php-jwt](//github.com/firebase/php-jwt)作為JWT的底層，這兒就使用Laravel作為框架。

<h3>
匯入`firebase/php-jwt`</h3>

首先在Laravel專案裡面匯入firebase/php-jwt：
```
composer require firebase/php-jwt
```

然後在`config/app.php`裡面加上JWT的aliases：
```
'aliases' => [
     ...
     'JWT' => Firebase\JWT\JWT::class,
   ],
```

## 建立JWT Auth類別

建立自己的Class來統一整個專案的JWT authentication，先在Laravel的`.env`裡面創建好環境變數：
```
JWT_AUTH_SECRET=some_secret_code
  JWT_ISSUE=testproject
```

然後建立一個名為`JWTAuth.php`的檔案，放在`app/`裡面：
```
namespace App;

  use Illuminate\Http\Request;
  use DB;
  use JWT;
  use Carbon\Carbon;

  class JWTAuth
  {

    /*
    * Getter function for key of JWT
    * Return(string): key of JWT
    */
    static protected function key() {
      return env('JWT_AUTH_SECRET', "");
    }

    /*
    * Getter function for issuer name of JWT
    * Return(string): issuer name of JWT
    */
    static protected function issue() {
      return env('JWT_ISSUE', "");
    }

    /*
    * Verify token based on username and time
    * Input Parameter (required*):
    * - jwt(string*): token
    * Return:(array)
    * - result(boolean): boolean for indicating whether the token is valid or not
    * - message(string): response message of the request
    * - data(array): the decoded token
    *   - uuid(string): unique id of this token
    *   - issue(string): issuer name of this token
    *   - username(string): account name
    *   - issue_at(time): time of this token issued
    *   - expired_at(time): time of this token expired
    * - token(string): token string

    * Token may be expired even the time now is not exceed expired_at in the decoded array of token.
    * This is because the user may logout to the system and only the expired_at in the database changed.
    * The expired_at in the decoded array of token cannot be changed since the value of the token string will change if the content is changed.
    * Therefore we need to examine both the expired_at value in decoded token array and that in database.
    */
    static public function verify($jwt) {
      try {
        $decode = JWT::decode($jwt, self::key(), array('HS256'));
        $uuid = $decode->uuid;
        $issue = $decode->issue;
        $username = $decode->username;
        $issue_at = Carbon::parse($decode->issue_at->date, $decode->issue_at->timezone);
        $expired_at = Carbon::parse($decode->expired_at->date, $decode->issue_at->timezone);
        $now_time = Carbon::now(env("TIMEZONE"));
        $token = DB::collection('token')->where("uuid", $uuid)->first();
        $server_expired_at = Carbon::parse($token["expired_at"]['date'], $token["expired_at"]['timezone']);
        if($now_time->gt($expired_at) || $now_time->gt($server_expired_at)) {
          return array('result' => false, 'message' => "Token Expired");
        } else {
          return array('result' => true, 'message' => "Verified", 'data' => $decode);
        }
      } catch (\Exception $e){
        return array('result' => false, 'message' => 'Invalid Token', 'exception' => $e->getMessage());
      }
    }

    /*
    * Generate token based on username and time
    * Input Parameter (required*):
    * - auth(array*)
    *   - username(string): account name
    * Return(string): generated token
    */
    static public function generate($auth) {
      $data = array(
        "uuid" => uniqid(),
        "issue" => self::issue(),
        "username" => $auth["username"],
        "issue_at" => Carbon::now(env("TIMEZONE")),
        "expired_at" => Carbon::now(env("TIMEZONE"))->addDay()
      );
      $token = JWT::encode($data, self::key());
      $data["token"] = $token;
      $id = DB::collection('token')->insertGetId($data);
      return $token;
    }

    /*
    * Check Login Username and Password
    * Input Parameter (required*):
    * - auth(array*)
    *   - username(string): account name
    *   - password: password of account
    * Return(boolean): boolean for indicating whether the username password pair is valid or not
    */
    static public function valid($auth) {
      $username = $auth["username"];
      $password = $auth["password"];
      $count = DB::table('users')->where('username', $username)->where('password', $password)->count();
      if($count > 0) {
        return true;
      } else {
        return false;
      }
    }

  }
  ?>

```

在`config/app.php`裡面加上上面自訂的Class JWTAuth的aliases：
```
'aliases' => [
     ...
     'JWTAuth' => App\JWTAuth::class,
   ],
```

<h3>
測試JWT認證方法</h3>

然後就可以利用這個自訂的Class來進行JWT認證了，假設是利用RESTful的POST auth這個作為routing的例子，先建立好route，在`routes/api.php`加上：
```
Route::post('auth', 'AuthController@post_index');
```

然後建立AuthController，作為發行JWT token的入口，也就是說用戶端會透過POST auth這個url來取得JWT的token；把`AuthController.php`放在`app/Http/Contrllers`裡面：
```
namespace App\Http\Controllers;

  use App\Http\Controllers\Controller;
  use Illuminate\Http\Request;
  use DB;
  use CASAuth;
  use Carbon\Carbon;
  use ResponseHelper;
  use ParameterValidator;

  class AuthController extends Controller
  {
    /**
    * Create a new controller instance.
    *
    * @return void
    */
    public function __construct()
    {

    }

    /*
    * POST /auth
    * Usage: Login
    * Input Parameter (required*):
    * - username(string*)(form data)
    * - password(string*)(form data)
    * Response:
    * - 200: Login success; token
    * - 400: Wrong parameter
    * - 401: Invalid username or password
    */
    public function post_index(Request $request) {
      $username = $request->input('username');
      $password = $request->input('password');
      $auth = array('username' => $username, 'password' => $password);
      if(JWTAuth::valid($auth)) {
        return JWTAuth::generate($auth); // Return generated token
      } else {
        return "Invalid username or password";
      }
    }

  }

```

然後就可以在資料庫裡面新增一個users的table，當中要有項目username和password，就可以透過這個賬號來測試JWT了。

在訪問POST auth這個路徑時，記得把username和password放在formdata裡面，而HTTP的method記得選用post，然後就可以獲得JWT的token了。

## 利用JWT token驗證用戶

建立一個名為`JWT.php`的Middleware，方便在Controller使用，放在`app/Http/Middleware`裡面：
```
namespace App\Http\Middleware;

  use Closure;
  use Illuminate\Support\Facades\Auth;
  use JWTAuth;
  use ResponseHelper;
  use ParameterValidator;

  class JWT
  {
    public function handle($request, Closure $next, $guard = null)
    {
      $token = $request->header('token');
      $result = JWTAuth::verify($token);
      if($result['result']) {
        $request['username'] = $result['data']->username;
        return $next($request);
      } else {
        return $result['message'];
      }
    }
  }

```

在`app/Http/Kernel.php`裡面加上Middleware：
```
protected $routeMiddleware = [
       ...
       'jwt' => \App\Http\Middleware\JWT::class,
   ];
```

然後就可以在各個需要認證的Controller裡面使用這個Middleware作為認證手段了，例子：
```
namespace App\Http\Controllers;

  use App\Http\Controllers\Controller;
  use Illuminate\Http\Request;
  use DB;

  class TestController extends Controller
  {
    public function __construct()
    {
      $this->middleware('jwt');
    }

    public function some_method() {
        // something to do after jwt auth
    }
  }
```
