# loginWithGoogle
شرح كيفية إنشاء موقع باستخدام فريمورك لارفل وحساب جوجل للتسجيل

##### تعديل جدول المستخدم وإضافة حقل google id
database/migrations/2014_10_12_000000_create_users_table.php
```
    public function up()
    {
        Schema::create('users', function (Blueprint $table) {
            $table->id();
            $table->string('name');
            $table->string('email')->unique();
            $table->timestamp('email_verified_at')->nullable();
            $table->string('password')->nullable();
            $table->string('google_id')->nullable();
            $table->rememberToken();
            $table->timestamps();
        });
    }
```
app/User.php
```
    protected $fillable = [
        'name', 'email', 'password', 'google_id'
    ];
    
```
```
php artisan migrate
 ```
##### إنشاء نظام التسجيل والدخول في لارفل
```
  composer require laravel/ui
```
```
  npm install 
```
```
  npm run dev
```

#### تثبيت مكتبة socialite
```
composer require laravel/socialite
```
#### إنشاء مشروع جديد في قوقل
https://developers.google.com/identity/sign-in/web/sign-in
#### إضافة بيانات مشروع قوقل إلى مشروع لارفل
config/services.php
```
    'google' => [
        'client_id' => 'xxx', // your client_id form google console
        'client_secret' => 'xxx', // your client_secret form google console
        'redirect' => 'http://127.0.0.1:8000/callback'],


];
```
#### إنشاء كونترولر وكتابة كود الاتصال بجوجل
```
php artisan make:controller LoginWithGoogleController
```
app/Http/Controllers/LoginWithGoogleController.php
```
use Illuminate\Http\Request;
use App\User;
use Socialite;
use Auth;
use Exception;
```
```
    public function redirect()
    {
        return Socialite::driver('google')->redirect();
    }
```
```
public function callback()
    {
        try {
            $user = Socialite::driver('google')->user();
        } catch (\Exception $e) {
            return redirect('/login');
        }
        // check if they're an existing user
        $existingUser = User::where('email', $user->email)->first();
        if($existingUser){
            // log them in
            auth()->login($existingUser, true);
        } else {
            // create a new user
            $newUser                  = new User;
            $newUser->name            = $user->name;
            $newUser->email           = $user->email;
            $newUser->google_id       = $user->id;
            $newUser->save();
            auth()->login($newUser, true);
        }
        return redirect()->to('/home');
    }
  ```
#### تعريف الروت route
routes/web.php
```
 Route::get('/redirect', 'LoginWithGoogleController@redirect');
Route::get('/callback', 'LoginWithGoogleController@callback');

 ```
 #### إضافة زر التسجيل باستخدام حساب جوجل
 resources/views/auth/login.blade.php
 resources/views/auth/register.blade.php
 
 ```
<a href="{{ url('/redirect') }}" class="btn btn-primary" > Login With Google </a>
 ```


#### كل شي جاهز للتجربة !
```
php artisan serve
```

