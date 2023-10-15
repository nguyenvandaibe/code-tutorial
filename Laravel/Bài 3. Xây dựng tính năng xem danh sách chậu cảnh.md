# HƯỚNG DẪN VIẾT CODE LARAVEL
# BÀI 3. XÂY DỰNG TÍNH NĂNG XEM DANH SÁCH CHẬU CẢNH

## 1. Nghiệp vụ
Yêu cầu: Quản trị viên cần xem danh sách các chậu cảnh. Các thông tin của một chậu cảnh mới tạo gồm có:
- 1 hình ảnh
- Tên chậu
- Kích thước: chiều dài, chiều rộng, chiều cao
- Giá bán

Tiều điều kiện: Quản trị viên phải đăng nhập hệ thống mới tạo được chậu cảnh.\

Hậu điều kiện: Không có.\

Luồng nghiệp vụ:
| Bước | Tác nhân | Hành động |
|---|---|---|
| 1 | Quản trị viên | Click vào menu "Chậu cảnh" trên thanh menu |
| 2 | Hệ thống | Hiển thị màn hình "Danh sách các chậu cảnh" |
| 3 | Quản trị viên | Click nút "Tạo chậu" |
| 4 | Hệ thống | Hiển thị màn hình "Tạo chậu cảnh mới" |
| 5 | Quản trị viên | Upload ảnh, điền thông tin form tạo chậu cảnh và bấm nút "Lưu" |
| 6 | Hệ thống | Lưu thông tin chậu cảnh và quay về màn hình "Danh sách các chậu cảnh" |

Trong đó:
- Ở màn hình "Danh sách các chậu cảnh", Quản trị viên có thể nhìn được danh sách các chậu cảnh mà mình đã đăng thông tin lên. Màn hình này, mỗi chậu có các thông tin gồm: Hình ảnh chậu, Tên chậu, Kích thước (chiều dài, chiều rộng, chiều cao), Giá bán.


## 2. Các bước thực hiện
### Bước 1. Tạo bảng lưu thông tin chậu cảnh
Theo quy tắc đặt tên bảng, tôi đặt tên bảng là bảng `pots` (danh từ số nhiều).\
Mở Terminal trên Visual Studio Code, tạo migration cho bảng `pots`:
```
php artisan make:migration create_pots_table
```

File migration `***_create_pots_table.php` được tạo ra, sử method `up` để bổ sung các cột cho bảng này:
```php
    /**
     * Run the migrations.
     */
    public function up(): void
    {
        Schema::create('pots', function (Blueprint $table) {
            $table->id();
            $table->string('name', 255)->comment('Tên chậu');
            $table->text('image')->comment('Hình ảnh của chậu');
            $table->unsignedInteger('dimesion_length')->comment('Chiều dài');
            $table->unsignedInteger('dimesion_width')->comment('Chiều rộng');
            $table->unsignedInteger('dimesion_height')->comment('Chiều cao');
            $table->unsignedBigInteger('price')->comment('Giá bán VNĐ');
            $table->timestamps();
            $table->softDeletes();
        });
    }
```

Chạy migrate để tạo bảng trong database:
```
php artisan migrate
```

Tạo 2 bản ghi chậu cảnh mới bằng cách chạy các câu truy vấn sau (Laravel có công cụ tạo dữ liệu mẫu, sẽ trình bày sau):
```sql
INSERT INTO `hoacanh_online`.`pots` (`id`, `name`, `image`, `dimesion_length`, `dimesion_width`, `dimesion_height`, `price`, `created_at`, `updated_at`) VALUES ('1', 'Chậu hoa hình chữ nhật đẹp', 'https://s3-minio.vobaitap.online/hoacanh-online/pots/chau-1.jpg', '150', '150', '150', '1000000', now(), now());
```

```sql
INSERT INTO `hoacanh_online`.`pots` (`id`, `name`, `image`, `dimesion_length`, `dimesion_width`, `dimesion_height`, `price`, `created_at`, `updated_at`) VALUES ('2', 'Chậu hoa xyz', 'https://s3-minio.vobaitap.online/hoacanh-online/pots/chau-2.jpg', '100', '80', '20', '200000', now(), now());
```



### Bước 2. Tạo một Model đại diện cho "Chậu cảnh" để giao tiếp với cơ sở dữ liệu
**Tạo model**\
Mở Terminal và chạy lệnh sau để tạo ra một model mới. Theo quy tắt đặt tên, model này có tên là `Pot`:
```
php artisan make:model Pot
```

Model `Pot.php` mới được tạo ra ở thư mục `app/Models`:
```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

class Pot extends Model
{
    use HasFactory;
}
```

**Sửa model**\
Ta cập nhật thông tin `$fillables` cho model `Pot`:
```php
    /**
     * The attributes that are mass assignable.
     *
     * @var array<int, string>
     */
    protected $fillable = [
        'name',
        'image',
        'dimesion_length',
        'dimesion_width',
        'dimesion_height',
        'price',
    ];

```
Chú ý, các item trong mảng `$fillables` phải khớp với tên cột đã tạo ở migration `***_create_pots_table.php`.



### Bước 3. Tạo một controller xử lý logic cho các chức năng liên quan đến chậu cảnh
Theo quy tắt đặt tên Laravel, tôi sẽ đặt tên cho controller này là `PotsController`. Mở Terminal và chạy lệnh sau để tạo controller:
```
php artisan make:controller PotsController
```

File `PotsController.php` mới sẽ được tạo ra ở trong thư mục `app/Http/Controllers`.\

Nội dung ban đầu của file như sau:
```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

class PotsController extends Controller
{
    //
}
```

### Bước 4. Tạo một method để hiển thị danh sách
```php
<?php

namespace App\Http\Controllers;

use App\Models\Pot;
use Illuminate\Http\Request;
use Illuminate\View\View;

class PotsController extends Controller
{
    /*
    |--------------------------------------------------------------------------
    | Pots Controller
    |--------------------------------------------------------------------------
    |
    | Controller này xử lý các request liên quan đến đối tượng "Chậu cảnh".
    |
    */

    /**
     * Liệt kê danh sách chậu cảnh
     *
     * @return View
     */
    public function index() : View {
        
        $pots = Pot::all(); // lấy toàn bộ chậu cảnh trong database

        return view('pots.admin_all_pots', ['pots' => $pots]); // trả ra view pots.admin_all_pots với dữ liệu là $pots
    }
}
```

### Bước 5. Tạo view "Danh sách các chậu cảnh"
Đối với view `pots.admin_all_pots` thì ta tạo một thư mục `pots` bên trong thư mục `resources/views` và tạo file `admin_all_pots.blade.php` trong thư mục `pots`:
![](/Laravel/Images/Bai%203/vi-tri%20view-pots.admin_all_pots.png "Vị trí view pots.admin_all_pots")

Để chứng minh view đã nhận được dữ liệu, ta sẽ sửa view `admin_all_pots.blade.php` để hiển thị được danh sách chậu cảnh:
```php
@extends('layouts.app')

@section('content')
<div class="container">
    @foreach ($pots as $pot)
    <div>
        <img src="{{$pot->src}}"/>
        <div>
            <div>{{$pot->name}}</div>
            <div>{{$pot->price}}</div>
            <div>{{$pot->dimesion_length}} cm x {{$pot->dimesion_width}} cm x {{$pot->dimesion_height}} cm</div>
        </div>
    </div>
    @endforeach
</div>
@endsection
```

### Bước 5. Tạo một route dẫn sang view "Danh sách các chậu cảnh"
Sửa file `routes/web.php` như sau:
```php
<?php

use Illuminate\Support\Facades\Route;
use App\Http\Controllers\HomeController;
use App\Http\Controllers\PotsController;

/*
|--------------------------------------------------------------------------
| Web Routes
|--------------------------------------------------------------------------
|
| Here is where you can register web routes for your application. These
| routes are loaded by the RouteServiceProvider and all of them will
| be assigned to the "web" middleware group. Make something great!
|
*/

Route::get('/', function () {
    return view('welcome');
});

Auth::routes();

Route::get('/home', [HomeController::class, 'index'])->name('home');

Route::get('/admin/pots', [PotsController::class, 'index']);

```

Ta sẽ đặt tên route `/admin/pots` là `admin.pots.index`. Thêm vào đó, nghiệp vụ này yêu cầu phải login thì mới tạo `Pot` được. Ta sẽ sửa route thêm như thế này:
```php
...
Route::get('/admin/pots', [PotsController::class, 'index'])->middleware('auth')->name('admin.pots.index');
```

Kiểm tra: chạy `php artisan serve` ở Terminal và gõ vào trình duyệt http://localhost:8000/admin/pots
Kết quả:
![](/Laravel/Images/Bai%203/Ket%20qua.png "Giao diện")

## 3. Tham khảo
[Tham khảo các file thay đổi](https://github.com/nguyenvandaibe/hoacanh-online/commit/935836de513782bb14f3bb1bfd66fe1852674cdd)
