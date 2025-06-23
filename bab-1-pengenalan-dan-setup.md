# BAB 1: PENGENALAN DAN SETUP PROJECT GAME STORE OOP

---

**📚 Modul Pembelajaran Object-Oriented Programming dengan PHP**

**Penulis:** Herisvan Hendra, M.Pd.T  
**BAB:** 1 - Pengenalan dan Setup Project  
**Estimasi Waktu:** 2-3 jam  
**Level:** Beginner

---

## Deskripsi
Bab ini membahas pengenalan konsep Object-Oriented Programming (OOP) dalam PHP dan setup project Game Store yang akan digunakan sebagai studi kasus pembelajaran. Project ini merupakan aplikasi e-commerce sederhana untuk penjualan game digital dengan fitur lengkap seperti manajemen produk, keranjang belanja, sistem pembayaran, dan dashboard admin.

## Tujuan Pembelajaran
Setelah menyelesaikan bab ini, mahasiswa diharapkan dapat:
1. Memahami konsep dasar OOP dalam PHP dan implementasinya
2. Menginstall dan mengkonfigurasi environment development yang tepat
3. Memahami struktur project Game Store dan arsitektur MVC
4. Menjalankan aplikasi Game Store dan melakukan testing dasar
5. Memahami best practices dalam pengembangan aplikasi web dengan OOP

## Materi

### 1.1 Konsep Dasar OOP

#### Pengertian OOP
Object-Oriented Programming (OOP) adalah paradigma pemrograman yang menggunakan objek dan kelas untuk mengorganisir kode. Konsep utama OOP meliputi:

- **Class**: Template atau blueprint untuk membuat objek
  ```php
  class User {
      private $name;
      private $email;
      
      public function __construct($name, $email) {
          $this->name = $name;
          $this->email = $email;
      }
  }
  ```

- **Object**: Instance dari sebuah class
  ```php
  $user = new User("John Doe", "john@example.com");
  ```

- **Encapsulation**: Menyembunyikan detail implementasi dengan access modifiers
  ```php
  private $password;    // Hanya bisa diakses dalam class
  protected $role;      // Bisa diakses dalam class dan subclass
  public $username;     // Bisa diakses dari mana saja
  ```

- **Inheritance**: Pewarisan sifat dari class parent
  ```php
  class Admin extends User {
      public function deleteUser($userId) {
          // Method khusus admin
      }
  }
  ```

- **Polymorphism**: Kemampuan objek untuk memiliki banyak bentuk
  ```php
  interface PaymentInterface {
      public function processPayment($amount);
  }
  
  class CreditCard implements PaymentInterface {
      public function processPayment($amount) {
          // Implementasi pembayaran kartu kredit
      }
  }
  ```

#### Keuntungan OOP
- **Maintainability**: Kode lebih terorganisir dan mudah dipelihara
- **Reusability**: Komponen dapat digunakan kembali di berbagai bagian aplikasi
- **Modularity**: Setiap class memiliki tanggung jawab yang spesifik
- **Scalability**: Mudah untuk menambah fitur baru tanpa mengubah kode existing
- **Debugging**: Error lebih mudah dilacak karena struktur yang jelas
- **Team Collaboration**: Multiple developer dapat bekerja pada class yang berbeda

### 1.2 Setup Environment Development

#### Kebutuhan Sistem
**Minimum Requirements:**
- PHP 7.4 atau lebih tinggi (Recommended: PHP 8.0+)
- MySQL 5.7 atau lebih tinggi (Recommended: MySQL 8.0+)
- Web Server (Apache/Nginx)
- RAM minimal 2GB
- Storage minimal 1GB free space

**Development Tools:**
- XAMPP/MAMP/WAMP (untuk development lokal)
- Text Editor/IDE (VS Code, PHPStorm, Sublime Text)
- Git (untuk version control)
- Browser modern (Chrome, Firefox, Safari)

**Optional Tools:**
- Composer (PHP dependency manager)
- Node.js dan NPM (untuk asset management)
- Postman (untuk API testing)

#### Langkah Instalasi

1. **Install XAMPP/MAMP**
   ```bash
   # Download dari official website
   # XAMPP: https://www.apachefriends.org/
   # MAMP: https://www.mamp.info/
   
   # Untuk Windows: Download XAMPP installer
   # Untuk macOS: Download MAMP installer
   # Untuk Linux: 
   sudo apt update
   sudo apt install apache2 mysql-server php php-mysql
   ```

2. **Clone/Download Project**
   ```bash
   # Jika menggunakan Git
   git clone <repository-url>
   cd game-store-oop
   
   # Atau download ZIP dan extract ke folder htdocs/www
   # XAMPP: C:\xampp\htdocs\game-store-oop
   # MAMP: /Applications/MAMP/htdocs/game-store-oop
   ```

3. **Start Services**
   ```bash
   # XAMPP Control Panel: Start Apache dan MySQL
   # MAMP: Start Servers
   
   # Atau menggunakan command line (Linux/macOS)
   sudo systemctl start apache2
   sudo systemctl start mysql
   ```

4. **Setup Database**
   - Buka phpMyAdmin: `http://localhost/phpmyadmin`
   - Buat database baru: `game_store_oop`
   - Pilih database yang baru dibuat
   - Klik tab "Import"
   - Pilih file `database.sql` dari project
   - Klik "Go" untuk import

5. **Konfigurasi Database**
   ```php
   // config/database.php
   private $host = 'localhost';
   private $db_name = 'game_store_oop';
   private $username = 'root';
   private $password = ''; // XAMPP default kosong
   // private $password = 'root'; // MAMP default 'root'
   ```

6. **Set Permissions (Linux/macOS)**
   ```bash
   chmod -R 755 /path/to/game-store-oop
   chown -R www-data:www-data /path/to/game-store-oop
   ```

### 1.3 Struktur Project

```
game-store-oop/
├── config/
│   └── database.php          # Konfigurasi database
├── controllers/
│   ├── AdminController.php   # Controller untuk admin
│   ├── AuthController.php    # Controller untuk autentikasi
│   ├── CartController.php    # Controller untuk keranjang
│   ├── OrderController.php   # Controller untuk pesanan
│   └── ProductController.php # Controller untuk produk
├── models/
│   ├── Cart.php             # Model keranjang
│   ├── Order.php            # Model pesanan
│   ├── OrderItem.php        # Model item pesanan
│   ├── Product.php          # Model produk
│   └── User.php             # Model pengguna
├── views/
│   ├── admin/               # Views untuk admin
│   ├── layouts/             # Layout template
│   └── user/                # Views untuk user
├── assets/
│   ├── css/                 # File CSS
│   ├── js/                  # File JavaScript
│   └── images/              # File gambar
├── index.php                # Entry point aplikasi
└── database.sql             # Script database
```

### 1.4 Arsitektur MVC

Project ini menggunakan pola arsitektur **Model-View-Controller (MVC)**:

#### Model
- Mengelola data dan logika bisnis
- Berinteraksi dengan database
- Contoh: `User.php`, `Product.php`, `Order.php`

#### View
- Menampilkan data kepada user
- File HTML/PHP untuk interface
- Contoh: `home.php`, `dashboard.php`, `login.php`

#### Controller
- Menghubungkan Model dan View
- Menangani request dari user
- Contoh: `AuthController.php`, `ProductController.php`

### 1.5 Menjalankan Aplikasi

1. **Start Web Server**
   ```bash
   # Menggunakan XAMPP/MAMP
   # - Buka XAMPP/MAMP Control Panel
   # - Start Apache dan MySQL services
   
   # Atau menggunakan built-in PHP server:
   cd /path/to/game-store-oop
   php -S localhost:8000
   
   # Untuk development dengan custom port:
   php -S localhost:3000
   ```

2. **Akses Aplikasi**
   - **XAMPP/MAMP**: `http://localhost/game-store-oop`
   - **Built-in Server**: `http://localhost:8000`
   - **Custom Port**: `http://localhost:3000`

3. **Testing Koneksi Database**
   ```php
   // test_connection.php (buat file ini di root project)
   <?php
   require_once 'config/database.php';
   
   try {
       $database = new Database();
       $conn = $database->getConnection();
       
       if($conn) {
           echo "✅ Koneksi database berhasil!";
           echo "<br>Server info: " . $conn->getAttribute(PDO::ATTR_SERVER_VERSION);
       }
   } catch(Exception $e) {
       echo "❌ Koneksi database gagal: " . $e->getMessage();
   }
   ?>
   ```

4. **Login Admin Default**
   - Username: `admin`
   - Password: `admin123`
   - URL: `http://localhost/game-store-oop/views/user/login.php`

5. **Register User Baru**
   - Klik "Daftar" untuk membuat akun user
   - URL: `http://localhost/game-store-oop/views/user/register.php`

6. **Troubleshooting Common Issues**
   - **Error 404**: Pastikan path URL sesuai dengan struktur folder
   - **Database Error**: Cek konfigurasi di `config/database.php`
   - **Permission Error**: Set permission yang tepat untuk folder project
   - **PHP Error**: Pastikan PHP version minimal 7.4

## Praktikum

### Latihan 1: Eksplorasi Struktur Project
1. Buka project di text editor/IDE
2. Identifikasi file-file utama dalam setiap folder
3. Buat diagram struktur project

### Latihan 2: Analisis Class Database
1. Buka file `config/database.php`
2. Analisis method `getConnection()`
3. Jelaskan fungsi setiap baris kode

### Latihan 3: Testing Koneksi Database
1. Buat file test sederhana untuk mengecek koneksi
2. Jalankan dan pastikan tidak ada error
3. Test CRUD operations sederhana

```php
<?php
// test_connection.php
require_once 'config/database.php';

try {
    $database = new Database();
    $conn = $database->getConnection();
    
    if($conn) {
        echo "✅ Koneksi database berhasil!<br>";
        echo "Server info: " . $conn->getAttribute(PDO::ATTR_SERVER_VERSION) . "<br>";
        echo "Database name: " . $conn->query('SELECT DATABASE()')->fetchColumn() . "<br>";
        
        // Test query sederhana
        $stmt = $conn->query("SHOW TABLES");
        $tables = $stmt->fetchAll(PDO::FETCH_COLUMN);
        echo "Tables found: " . implode(', ', $tables) . "<br>";
        
        // Test count records
        $stmt = $conn->query("SELECT COUNT(*) FROM users");
        $userCount = $stmt->fetchColumn();
        echo "Total users: " . $userCount . "<br>";
        
    }
} catch(PDOException $e) {
    echo "❌ Database Error: " . $e->getMessage() . "<br>";
} catch(Exception $e) {
    echo "❌ General Error: " . $e->getMessage() . "<br>";
}
?>
```

### Latihan 4: Eksplorasi Fitur Aplikasi
1. Login sebagai admin dan jelajahi dashboard
2. Tambah produk baru melalui admin panel
3. Register sebagai user dan test shopping cart
4. Test proses checkout dan order management

```php
// test_features.php
<?php
session_start();
require_once 'controllers/ProductController.php';
require_once 'controllers/AuthController.php';

// Test ProductController
$productController = new ProductController();
$products = $productController->getAllProducts();
echo "Total products: " . count($products) . "<br>";

// Test AuthController
$authController = new AuthController();
echo "Auth system loaded successfully<br>";

// Test session
if(isset($_SESSION['logged_in'])) {
    echo "User logged in: " . $_SESSION['username'] . "<br>";
} else {
    echo "No user logged in<br>";
}
?>
```

## Tugas

### Tugas Individu
1. **Setup Environment**: 
   - Install dan konfigurasi environment development
   - Screenshot setiap langkah instalasi
   - Dokumentasikan masalah yang ditemui dan solusinya

2. **Dokumentasi Project**: 
   - Buat dokumentasi setup yang Anda lakukan
   - Buat diagram struktur folder project
   - Dokumentasikan setiap file dan fungsinya

3. **Eksplorasi Kode**: 
   - Jelajahi semua file dalam project
   - Buat catatan fungsi masing-masing class dan method
   - Identifikasi implementasi OOP concepts dalam project

4. **Testing dan Debugging**: 
   - Pastikan aplikasi dapat berjalan tanpa error
   - Test semua fitur utama (login, register, CRUD produk)
   - Buat laporan testing dengan screenshot

### Tugas Kelompok
1. **Analisis Arsitektur**: 
   - Diskusikan implementasi MVC dalam project
   - Bandingkan dengan framework PHP lainnya
   - Presentasikan hasil analisis

2. **Enhancement Ideas**: 
   - Brainstorm fitur tambahan yang bisa ditambahkan
   - Buat proposal improvement untuk project
   - Diskusikan implementasi yang mungkin

## Evaluasi

### Checklist Penyelesaian
**Setup dan Konfigurasi:**
- [ ] Environment development terinstall (XAMPP/MAMP)
- [ ] PHP dan MySQL berjalan dengan baik
- [ ] Database terkonfigurasi dengan benar
- [ ] Project files tersimpan di lokasi yang tepat
- [ ] Permissions folder sudah diset (Linux/macOS)

**Testing Aplikasi:**
- [ ] Aplikasi dapat dijalankan tanpa error
- [ ] Koneksi database berhasil
- [ ] Dapat mengakses halaman utama
- [ ] Dapat login sebagai admin
- [ ] Dapat register user baru
- [ ] Dashboard admin dapat diakses

**Pemahaman Konsep:**
- [ ] Memahami struktur project dan MVC
- [ ] Memahami konsep OOP dasar
- [ ] Dapat menjelaskan fungsi setiap folder
- [ ] Memahami flow aplikasi

### Pertanyaan Review

**Konsep OOP:**
1. Apa perbedaan antara class dan object? Berikan contoh implementasinya!
2. Jelaskan 4 pilar OOP dan berikan contoh dalam project Game Store!
3. Apa keuntungan menggunakan encapsulation dalam programming?
4. Bagaimana inheritance diterapkan dalam project ini?

**Arsitektur dan Setup:**
5. Mengapa menggunakan pola MVC? Apa keuntungannya?
6. Apa fungsi file `index.php` dalam project ini?
7. Bagaimana cara kerja koneksi database menggunakan PDO?
8. Jelaskan perbedaan antara XAMPP dan MAMP!

**Troubleshooting:**
9. Apa yang harus dilakukan jika muncul error "Database connection failed"?
10. Bagaimana cara mengatasi error 404 saat mengakses aplikasi?

### Kriteria Penilaian

**Excellent (90-100):**
- Setup environment sempurna tanpa error
- Aplikasi berjalan dengan semua fitur berfungsi
- Dokumentasi lengkap dan detail
- Dapat menjelaskan semua konsep dengan baik
- Mampu troubleshooting masalah sendiri

**Good (80-89):**
- Setup berhasil dengan sedikit bantuan
- Aplikasi berjalan dengan minor issues
- Dokumentasi cukup lengkap
- Memahami sebagian besar konsep
- Dapat mengatasi masalah dengan guidance

**Satisfactory (70-79):**
- Setup berhasil dengan bantuan signifikan
- Aplikasi berjalan dengan beberapa error
- Dokumentasi basic
- Memahami konsep dasar
- Memerlukan bantuan untuk troubleshooting

**Needs Improvement (<70):**
- Setup tidak berhasil atau banyak error
- Aplikasi tidak berjalan dengan baik
- Dokumentasi tidak lengkap
- Pemahaman konsep kurang
- Tidak dapat mengatasi masalah

### Resources Tambahan
- [PHP OOP Documentation](https://www.php.net/manual/en/language.oop5.php)
- [MVC Pattern Explained](https://www.tutorialspoint.com/mvc_framework/mvc_framework_introduction.htm)
- [PDO Tutorial](https://phpdelusions.net/pdo)
- [XAMPP Documentation](https://www.apachefriends.org/docs/)

---

**Catatan**: Pastikan semua langkah setup telah dilakukan dengan benar dan aplikasi berjalan sempurna sebelum melanjutkan ke bab selanjutnya. Jika mengalami kesulitan, jangan ragu untuk bertanya kepada instruktur atau diskusi dengan teman sekelas.