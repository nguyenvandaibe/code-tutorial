# HƯỚNG DẪN VIẾT CODE LARAVEL
## 1. Khởi tạo project
Mở command line tại thư mục dự định chứa source code và chạy lệnh sau:
```
larvel new my-project
```

## 2. Thiết lập Laravel UI
Hướng dẫn: https://packagist.org/packages/laravel/ui
Mở project bằng Visual Studio Code, mở Terminal và chạy các câu lệnh như sau:
- Cài đặt Laravel UI
```
composer require laravel/ui
```

- Frontend scaffolding
```
php artisan ui bootstrap --auth
```

- Compile source frontend:
```
npm install
npm run build
```


## 3. Thiết lập đăng nhập bằng username thay vì email

**Gợi ý:** [Tham khảo các file thay đổi thực hiện](https://github.com/nguyenvandaibe/hoacanh-online/commit/d57f335c87fff9ef0a7080072f36e87ccb505087)

### Bước 1. Bổ sung cột `username` vào bảng `users`**
Chạy command line sau để tạo migration:
```
php artisan make:migration add_username_column_to_users_table
```


Mở file database/migrations/***_add_username_column_to_users_table.php, sửa function `up` như sau:
```
/**
 * Run the migrations.
 */
public function up(): void
{
    Schema::table('users', function (Blueprint $table) {
        $table->string('username', 255)->after()->unique()->comment('Ten dang nhap');
    });
}
```
Ý nghĩa: Bổ sung cột `username` vào bảng `users`, vị trí là phía sau cột `email_verified_at`. Các giá trị của `username` là duy nhất (unique()).


Sửa lại function `down`:
```
/**
 * Reverse the migrations.
 */
public function down(): void
{
    Schema::table('users', function (Blueprint $table) {
        $table->dropColumn('username');
    });
}
```
Ý nghĩa: Khi revert migration này thì xoá cột `username`

Chạy migration:
```
php artisan migrate
```

Kiểm tra lại thông tin cột trong cơ sở dữ liệu.


### Bước 2. Sửa model User
Mở file /app/Models/User.php, cập nhật $fillables:
```
    /**
     * The attributes that are mass assignable.
     *
     * @var array<int, string>
     */
    protected $fillable = [
        ...
        'username'
    ];
```

Ý nghĩa: Bổ sung thêm thuộc tính `username` để sau này create đổi tượng của model User.

### Bước 3. Cập nhật LoginController
Trong class `LoginController` có sử dụng một trait là `AuthenticatesUsers`, có thể hiểu đơn giản là `LoginController` thực tế có các phương thức y hệt như trong trait `AuthenticatesUsers`. Ta thực hiện ghi đè phương thức `username()` vào `LoginController` bằng cách bổ sung code như sau:
```
class LoginController extends Controller
{
    // Các dòng code cũ, không sửa đến
    ...

    // Phẩn bổ sung:
    /**
     * Get the login username to be used by the controller.
     *
     * @return string
     */
    public function username()
    {
        return 'username';
    }
}

```
Ý nghĩa: Trong trait `AuthenticatesUsers`, phương thức `username()` trả về giá trị 'email'. Điều này có nghĩa là mặc định, Laravel sử dụng giá trị `email` của model `User` để login. Nay ta chuyển về return 'username' thì sẽ dùng được `username` để login.

### Bước 4. Cập nhật RegisterController
- Sửa phương thức là `validator`:\
Phương thức này nhận đầu vào là mảng `$data` truyền từ form lên, có vai trò kiểm tra tính đúng đắn của dữ liệu được submit từ form. Ta sẽ thêm quy tắc (rule) validate cho thông tin `username`:
```
    /**
     * Get a validator for an incoming registration request.
     *
     * @param  array  $data
     * @return \Illuminate\Contracts\Validation\Validator
     */
    protected function validator(array $data)
    {
        return Validator::make($data, [
            'name' => ['required', 'string', 'max:255'],
            'email' => ['required', 'string', 'email', 'max:255', 'unique:users'],
            'password' => ['required', 'string', 'min:8', 'confirmed'],
            'username' => ['required', 'string', 'max:255', 'unique:users'],
        ]);
    }
```
Về các quy tắc validate, xem thêm tại: https://laravel.com/docs/10.x/validation#available-validation-rules

- Sửa phương thức `create`:\
Phương thức này chính là nơi để tạo ra đối tượng của model User và lưu đối tượng vào database:
```
    /**
     * Create a new user instance after a valid registration.
     *
     * @param  array  $data
     * @return \App\Models\User
     */
    protected function create(array $data)
    {
        return User::create([
            'name' => $data['name'],
            'email' => $data['email'],
            'password' => Hash::make($data['password']),
            'username' => $data['username'],
        ]);
    }
```

### Bước 5. Sửa view login và register
Vị trí file:
- Trang login: /resources/views/auth/login.blade.php
- Trang register: /resources/views/auth/register.blade.php


Trang login: Sửa phần input của `Email Address`:
```
...
                    <form method="POST" action="{{ route('login') }}">
                        @csrf

                        <div class="row mb-3">
                            <label for="username" class="col-md-4 col-form-label text-md-end">{{ __('Tên đăng nhập') }}</label>

                            <div class="col-md-6">
                                <input id="username" type="text" class="form-control @error('username') is-invalid @enderror" name="username" value="{{ old('username') }}" required autocomplete="username" autofocus>

                                @error('username')
                                    <span class="invalid-feedback" role="alert">
                                        <strong>{{ $message }}</strong>
                                    </span>
                                @enderror
                            </div>
                        </div>

...
```


Trang register: Sao chép phần input của `Email Address`, đổi giá trị của `type="email"` thành `type="text"` và các property `email` thành `username`:
```
```


