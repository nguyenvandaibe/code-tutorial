# HƯỚNG DẪN VIẾT CODE LARAVEL
# BÀI 4. XÂY DỰNG TÍNH NĂNG TẠO CHẬU CẢNH MỚI

## 1. Nghiệp vụ
Ở trong tài liệu hướng dẫn này, chúng ta thực hiện xây dựng tính năng tạo chậu cảnh mới. Theo nghiệp vụ ở `Bài 3`, chậu cảnh có các thông tin gồm:
- 1 hình ảnh
- Tên chậu
- Kích thước: chiều dài, chiều rộng, chiều cao
- Giá bán

Đối với các thông tin `Kích thước` (chiều dài, chiều rộng, chiều cao), `Giá bán`, ta có thể làm một thao tác đơn gián từ input vào controller và dùng model create để lưu. Vấn đề cần suy nghĩ là hình ảnh của chậu được lưu ở đâu và lưu như thế nào.

### Lưu ở đâu ?
Ở đây chúng ta sẽ sử dụng MinIO làm nơi lưu file. Chúng ta sẽ viết code upload file từ input lên MinIO và lưu lại đường dẫn của file này.


### Lưu như thế nào ?
Có 2 cách có thể thực hiện tính năng tạo chậu cảnh có lưu ảnh:
**Cách 1**\
Trên giao diện, trong form tạo chậu cảnh mới, ta làm một phần file input, upload file lên MinIO qua một controller riêng rồi trả về một đường dẫn file ảnh. Sau đó, dùng đường dẫn file ảnh này cùng với các thông tin `Kích thước`, `Giá bán` submit form tạo chậu cảnh và đưa lên controller.

**Cách 2**\
Ta submit cả file ảnh cùng các thông tin `Kích thước`, `Giá bán` submit form tạo chậu cảnh và đưa lên controller. Trong controller, ta upload file lên MinIO rồi lấy URL trả về để lưu vào database.

```
Ở trong hoàn cảnh hướng dẫn đơn giản này, tôi thực hiện theo cách 2 vì đánh giá rằng nó đơn giản hơn cách 1.
```

## 2. Các bước thực hiện
Chúng ta sẽ xây dựng theo các bước:\
Controller --> Route --> View
### Bước 1. Xây dựng một phương thức trong PotsController để trả ra view "Tạo chậu cảnh mới"
Bổ sung thêm phương thức `create` vào `PotsController`:
```php
    /**
     * Trả về view "Tạo chậu cảnh"
     *
     * @return View
     */
    public function create() : View {
        return view('pots.admin_create_pot');
    }
```

### Bước 2. Xây dựng view "Tạo chậu cảnh mới": `pots.admin_create_pot`
Tạo file mới: /resources/views/pots/admin_create_pot.blade.php\
View `pots.admin_create_pot` bao gồm một form tạo chậu mới:
```php
@extends('layouts.app')

@section('content')
<div class="container">
    <form method="POST" action="#" enctype="multipart/form-data">
        <div class="mb-3">
            <label for="name" class="form-label">Đặt tên cho chậu</label>
            <input type="text" class="form-control bg-white" id="name" placeholder="Nhập tên chậu">
        </div>
        <div class="mb-3">
            <label for="length" class="form-label">Chiều dài</label>
            <input type="number" class="form-control bg-white" name="length" id="length" value="0" min="1" max="10000"> cm
        </div>
        <div class="mb-3">
            <label for="width" class="form-label">Chiều rộng</label>
            <input type="number" class="form-control bg-white" name="width" id="width" value="0" min="1" max="10000"> cm
        </div>
        <div class="mb-3">
            <label for="height" class="form-label">Chiều cao</label>
            <input type="number" class="form-control bg-white" name="height" id="height" value="0" min="1" max="10000"> cm
        </div>
        <div class="mb-3">
            <label for="uploadedImage" class="form-label">Chọn ảnh</label>
            <input class="form-control bg-white" type="file" id="uploadedImage" name="uploadedImage" accept="image/*">
        </div>
        <div class="mb-3">
            <button type="submit" class="btn btn-primary px-5">
                Lưu
            </button>
        </div>
    </form>
</div>
@endsection
```

Ta bổ sung tiếp vào form này các thành phần khác như là `@csrf` (dùng để bảo mật), giá trị old(), phần hiển thị lỗi validate trả ra. Các thành phần này có thể tham khảo từ view `login.blade.php` và `register.blade.php`. Kết quả như sau:

```php
@extends('layouts.app')

@section('content')
<div class="container">
    <form action="POST" action="#" enctype="multipart/form-data">

        @csrf

        <div class="mb-3">
            <label for="name" class="form-label">Đặt tên cho chậu</label>
            <input type="text" class="form-control bg-white @error('name') is-invalid @enderror" id="name" value="{{ old('name') }}" placeholder="Nhập tên chậu">
            @error('name')
                <span class="invalid-feedback" role="alert">
                    <strong>{{ $message }}</strong>
                </span>
            @enderror
        </div>
        
        <div class="mb-3">
            <label for="length" class="form-label">Chiều dài (cm)</label>
            <input type="number" class="form-control bg-white @error('length') is-invalid @enderror" name="length" id="length" value="{{ old('length') !== null ? old('length') : 0 }}" min="1" max="10000"> cm
            @error('length')
                <span class="invalid-feedback" role="alert">
                    <strong>{{ $message }}</strong>
                </span>
            @enderror
        </div>
        
        <div class="mb-3">
            <label for="width" class="form-label">Chiều rộng (cm)</label>
            <input type="number" class="form-control bg-white @error('width') is-invalid @enderror" name="width" id="width" value="{{ old('width') !== null ? old('width') : 0 }}" min="1" max="10000">
            @error('width')
                <span class="invalid-feedback" role="alert">
                    <strong>{{ $message }}</strong>
                </span>
            @enderror
        </div>
        
        <div class="mb-3">
            <label for="height" class="form-label">Chiều cao (cm)</label>
            <input type="number" class="form-control bg-white @error('height') is-invalid @enderror" name="height" id="height" value="{{ old('height') !== null ? old('height') : 0 }}" min="1" max="10000">
            @error('height')
                <span class="invalid-feedback" role="alert">
                    <strong>{{ $message }}</strong>
                </span>
            @enderror
        </div>
        
        <div class="mb-3">
            <label for="uploadedImage" class="form-label">Chọn ảnh</label>
            <input class="form-control bg-white @error('uploadedImage') is-invalid @enderror" type="file" id="uploadedImage" name="uploadedImage" accept="image/*">
            @error('uploadedImage')
                <span class="invalid-feedback" role="alert">
                    <strong>{{ $message }}</strong>
                </span>
            @enderror
        </div>
        
        <div class="mb-3">
            <button type="submit" class="btn btn-primary px-5">
                Lưu
            </button>
        </div>
    </form>
</div>
@endsection
```

### Bước 3. Bổ sung route dẫn đến view "Tạo chậu cảnh mới"
Bổ sung vào cuối file `routes/web.php`
```php
Route::get('/admin/pots/create', [PotsController::class, 'create'])->middleware('auth')->name('admin.pots.create');
```
Khi truy cập đường dẫn http://localhost:8000/admin/pots/create, ta được kết quả như sau:\
![](/Laravel/Images/Bai%204/form-tao-chau-canh-moi.png "Màn hình tạo chậu cảnh mới")


### Bước 4. Xây dựng phương thức `store` trong `PotsController` để đón request từ form submit lên
**Bổ sung phương thức `store` trong `PotsController`:**

Ở đây, ta sẽ xây dựng `khung` của phương thức `store` để xem thông tin request gửi từ view "Tạo chậu cảnh mới" lên `PotsController` là gì. Sau khi đã xác định rõ các thành phần, ta sẽ hoàn thiện phương thức `store` sau.

```php
    /**
     * Lưu thông tin chậu cảnh mới
     *
     * @param Request $request
     * @return View
     */
    public function store(Request $request) {
        dd($request); // Hiển thị toàn bộ thông tin request và dừng chương trình
    }
```

**Bổ sung route đến phương thức `store` vào `routes/web.php`:**
```php
Route::post('/admin/pots/store', [PotsController::class, 'store'])->middleware('auth')->name('admin.pots.store');
```

**Đặt tên route `admin.pots.store` vào form trong view `pots.admin_create_pot` bằng cách sửa lại thẻ <form>**
```php
<form method="POST" action="{{ route('admin.pots.store') }}" enctype="multipart/form-data">
```

**Kiểm tra**\
Điền một vài thông tin vào form:\
![](/Laravel/Images/Bai%204/vi-du-dien-form-tao-chau-canh-moi.png "Điền thông tin vào form")

Kết quả:\
![](/Laravel/Images/Bai%204/noi-dung-request.png "Nội dung của request mới submit lên")

Ta sẽ tập trung vào 2 thông tin là `files` và `request`:
![](/Laravel/Images/Bai%204/thong-tin-request-can-chu-y.png "Nội dung files và request")

Trong đó:
- `request` là một mảng (#parmeter: array:5 - mảng có 5 phần tử), có các `key` là `name`, `length`, `width`, `height`. Key thì tương ứng với property `name` ở trong thẻ <input> ở form. Ví dụ nếu thẻ input là <input name="studentName" ...> thì lúc submit form sẽ nhận được `key` là `studentName`.
- `files` cũng là một mảng (#parameters: array:1 - mảng có 1 phần tử, chính là file `chau-1.jpg` được chọn)

Như vậy, ta có thể lấy ra các giá trị `files` và các `key` của `request` như sau:
```php
    /**
     * Lưu thông tin chậu cảnh mới
     *
     * @param Request $request
     * @return View
     */
    public function store(Request $request)
    {
        $uploadedImage = $request->file('uploadedImage');

        // Tải $uploadedImage lên lưu ở MinIO
        // rồi thu về URL của ảnh
        // dùng URL này làm giá trị của thuộc tính 'image' của Pot mới
        $uploadedImageURL = '';

        // Lưu vào cơ sở dữ liệu
        Pot::create([
            'name' => $request->input('name'),
            'image' => $uploadedImageURL,
            'dimesion_length' => $request->input('dimesion_length'),
            'dimesion_width' => $request->input('dimesion_width'),
            'dimesion_height' => $request->input('dimesion_height'),
            'price' => $request->input('price')
        ]);
    }
```

### Bước 5. Xây dựng tính năng upload file lên MinIO
#### Phân tích
Chúng ta sẽ tư duy rằng việc upload file này không chỉ cần cho việc tạo chậu cảnh mới mà có thể còn dùng cho nhiều chức năng khác. Vì thế chúng ta viết một class riêng để xử lý nghiệp vụ này.\
Nghiệp vụ này liên quan đến file storage. Trong đó MinIO là hệ thống file storage tương tự Amazon S3. Tham khảo [Laravel File storage](https://laravel.com/docs/10.x/filesystem#amazon-s3-compatible-filesystems) để tìm hiểu thêm.

#### Cài đặt thư viện
Mở Terminal, cài đặt thư viện `league/flysystem-aws-s3-v3`
```sh
composer require league/flysystem-aws-s3-v3
```

#### Cấu hình MinIO
Bổ sung các thông tin sau vào cuối file `.env`:
```
FILESYSTEM_CLOUD=s3
AWS_ACCESS_KEY_ID=***
AWS_SECRET_ACCESS_KEY=***
AWS_DEFAULT_REGION=us-east-1
AWS_BUCKET=hoacanh-online
AWS_URL="https://s3-minio.vobaitap.online"
AWS_ENDPOINT="https://s3-minio.vobaitap.online"
AWS_USE_PATH_STYLE_ENDPOINT=true
```

Cập nhật file `config/filesystems.php`, bổ sung dòng `'cloud' => env(...)`:
```php
<?php

return [

    /*
    |--------------------------------------------------------------------------
    | Default Filesystem Disk
    |--------------------------------------------------------------------------
    |
    | Here you may specify the default filesystem disk that should be used
    | by the framework. The "local" disk, as well as a variety of cloud
    | based disks are available to your application. Just store away!
    |
    */

    'default' => env('FILESYSTEM_DISK', 'local'),

    'cloud' => env('FILESYSTEM_CLOUD', 's3'),
    // ...
```

**Chú ý:** Các thay đổi ở file `.env` không nhìn thấy trên Github.\

#### Sửa phương thức `store`, đổi tên file
```php
    /**
     * Lưu thông tin chậu cảnh mới
     *
     * @param Request $request
     * @return View
     */
    public function store(Request $request)
    {
        $uploadedImage = $request->file('uploadedImage');

        #region Đổi tên file ảnh để tránh trùng lặp
        // Lấy ra tên file
        // Ví dụ: my-image.jpg
        $imageOriginalName = $uploadedImage->getClientOriginalName();

        // Lấy ra file extension
        // Ví dụ: my-image.jpg ==> jpg
        $imageOriginalExtension = $uploadedImage->getClientOriginalExtension();

        // Lấy ra chuỗi chỉ có tên file không có extension
        // Ví dụ: my-image.jpg ==> my-imge
        $imageOriginalNameWithoutExtension = trim($imageOriginalName, '.' . $imageOriginalExtension);

        $imageName = $imageOriginalNameWithoutExtension . '-' . Carbon::now()->timestamp . '.' . $imageOriginalExtension;
        #endregion
        
        #region Upload file lên MinIO
        
        #endregion
        
        $uploadedImageURL = '';
        
        // Lưu vào cơ sở dữ liệu
        Pot::create([
            'name' => $request->input('name'),
            'image' => $uploadedImageURL,
            'dimesion_length' => $request->input('dimesion_length'),
            'dimesion_width' => $request->input('dimesion_width'),
            'dimesion_height' => $request->input('dimesion_height'),
            'price' => $request->input('price')
        ]);
    }
```

#### Sửa phương thức `store`, sửa tiếp code `Upload file lên MinIO`
```php
    /**
     * Lưu thông tin chậu cảnh mới
     *
     * @param Request $request
     * @return View
     */
    public function store(Request $request)
    {
        $uploadedImage = $request->file('uploadedImage');

        #region Đổi tên file ảnh để tránh trùng lặp
        // Lấy ra tên file
        // Ví dụ: my-image.jpg
        $imageOriginalName = $uploadedImage->getClientOriginalName();

        // Lấy ra file extension
        // Ví dụ: my-image.jpg ==> jpg
        $imageOriginalExtension = $uploadedImage->getClientOriginalExtension();

        // Lấy ra chuỗi chỉ có tên file không có extension
        // Ví dụ: my-image.jpg ==> my-imge
        $imageOriginalNameWithoutExtension = trim($imageOriginalName, '.' . $imageOriginalExtension);

        $imageName = $imageOriginalNameWithoutExtension . '-' . Carbon::now()->timestamp . '.' . $imageOriginalExtension;

        $imagePath = 'pots/' . $imageName;
        #endregion

        // Upload file lên MinIO
        // Trả về true: Thành công
        // Trả về false: Thất bại
        $hasSuccessfullyUpload = Storage::cloud()->put($imagePath, $uploadedImage->getContent());

        // Nếu upload lên MinIO thành công
        if ($hasSuccessfullyUpload) {

            // Lấy đầy đủ URL của ảnh
            $uploadedImageURL = config('filesystems.disks.s3.endpoint') . config('filesystems.disks.s3.bucket') . '/' . $imagePath;

            // Lưu vào cơ sở dữ liệu
            Pot::create([
                'name' => $request->input('name'),
                'image' => $uploadedImageURL,
                'dimesion_length' => $request->input('dimesion_length'),
                'dimesion_width' => $request->input('dimesion_width'),
                'dimesion_height' => $request->input('dimesion_height'),
                'price' => $request->input('price')
            ]);
            
            // Trả về kết quả là đã tạo thành công
            // return
        } else {
            // Trả về kết quả là không tạo thành công
            // return
        }
    }
```

#### Trả về kết quả
Bây giờ ta lại muốn sau khi tạo xong một chậu cảnh thì trả về màn hình danh sách chậu cảnh. Ta sẽ sửa tiếp phần `return` đang bị comment:
```php
        // Nếu upload lên MinIO thành công
        if ($hasSuccessfullyUpload) {

            // Lấy đầy đủ URL của ảnh
            $uploadedImageURL = config('filesystems.disks.s3.endpoint') . config('filesystems.disks.s3.bucket') . '/' . $imagePath;

            // Lưu vào cơ sở dữ liệu
            Pot::create([
                'name' => $request->input('name'),
                'image' => $uploadedImageURL,
                'dimesion_length' => $request->input('dimesion_length'),
                'dimesion_width' => $request->input('dimesion_width'),
                'dimesion_height' => $request->input('dimesion_height'),
                'price' => $request->input('price')
            ]);
            
            // Trả về kết quả là đã tạo thành công
            return redirect()->route('admin.pots.index')->with('status', [
                'isSuccess' => true,
                'message' => 'Tạo chậu ' . $request->input('name') . ' thành công.'
            ]);
        } else {
            // Trả về kết quả là không tạo thành công
            redirect()->route('admin.pots.index')->with('status', [
                'isSuccess' => false,
                'message' => 'Tạo chậu ' . $request->input('name') . ' thất bại.'
            ]);
        }
```

Cập nhật lại view `pots.admin_all_pots`:
```php
@extends('layouts.app')

@section('content')
<div class="container">
    @if(session('status'))
        @if (session('status')['isSuccess'])
            <div class="alert alert-success mb-3">
                {{ session('status')['message'] }}
            </div>
        @else
            <div class="alert alert-alert mb-3">
                {{ session('status')['message'] }}
            </div>
        @endif
    @endif
    
    <div class="mb-3">
        <a href="{{ route('admin.pots.create') }}" class="btn btn-primary">
            Tạo chậu cảnh mới
        </a>
    </div>
    
    @foreach ($pots as $pot)
    <div>
        <img src="{{url($pot->image)}}"/>
        <div>
            <div>{{$pot->name}}</div>
            <div>{{$pot->price}} VNĐ</div>
            <div>{{$pot->dimesion_length}} cm x {{$pot->dimesion_width}} cm x {{$pot->dimesion_height}} cm</div>
        </div>
    </div>
    @endforeach
</div>
@endsection

```

Ta cũng cập nhật lại `PotsController`, method `index` để lấy các chậu được tạo ra từ mới nhất đến cũ nhất cho dễ nhìn kết quả:
```php
    /**
     * Liệt kê danh sách chậu cảnh
     *
     * @return View
     */
    public function index(): View
    {

        $pots = Pot::orderByDesc('created_at')->get(); // lấy toàn bộ chậu cảnh trong database

        return view('pots.admin_all_pots', ['pots' => $pots]); // trả ra view pots.admin_all_pots với dữ liệu là $pots
    }
```

#### Kết quả đạt được
![](/Laravel/Images/Bai%204/tao-chau-canh-moi-thanh-cong.png "Tạo chậu cảnh mới thành công")

## 3. Tham khảo
[Tham khảo các file thay đổi](https://github.com/nguyenvandaibe/hoacanh-online/commit/668c84bb7b6abf4d5b1d1f1e721024d781c9d9dc)