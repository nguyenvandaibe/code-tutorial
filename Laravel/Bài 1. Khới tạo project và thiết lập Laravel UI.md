# HƯỚNG DẪN VIẾT CODE LARAVEL
# BÀI 1. KHỞI TẠO PROJECT VÀ THIẾT LẬP LARAVEL UI
Phiên bản Laravel: 10.x

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


