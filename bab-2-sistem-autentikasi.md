# BAB 2: SISTEM AUTENTIKASI DAN MANAJEMEN USER

---

**📚 Modul Pembelajaran Object-Oriented Programming dengan PHP**

**Penulis:** Herisvan Hendra, M.Pd.T  
**BAB:** 2 - Sistem Autentikasi  
**Estimasi Waktu:** 3-4 jam  
**Level:** Beginner to Intermediate

---

## Deskripsi
Bab ini membahas implementasi sistem autentikasi (login/register) dan manajemen user menggunakan konsep Object-Oriented Programming dalam PHP. Mahasiswa akan mempelajari cara kerja AuthController dan User model, serta implementasi security best practices dalam pengembangan web application. Sistem autentikasi merupakan fondasi penting dalam aplikasi web yang memungkinkan pengelolaan akses user dan admin.

## Tujuan Pembelajaran
Setelah menyelesaikan bab ini, mahasiswa diharapkan dapat:
1. Memahami implementasi sistem autentikasi menggunakan OOP principles
2. Menganalisis dan mengimplementasikan AuthController dan User model
3. Memahami konsep session management dan state management
4. Mengimplementasikan validasi input dan security measures
5. Membuat fitur login, register, logout dengan error handling
6. Memahami role-based access control (RBAC)
7. Mengimplementasikan password hashing dan security best practices

## Materi

### 2.1 Arsitektur Sistem Autentikasi

#### Komponen Utama
1. **AuthController**: Mengelola logika autentikasi
2. **User Model**: Mengelola data user dan database operations
3. **Session Management**: Mengelola status login user
4. **View Files**: Interface untuk login dan register

#### Flow Autentikasi
```
User Input → Validation → AuthController → User Model → Database → Session → Response
     ↓           ↓            ↓            ↓          ↓         ↓        ↓
  Form Data → Sanitize → Business Logic → SQL Query → Storage → Memory → UI Update
```

#### Diagram Sequence Autentikasi
```
User        View        Controller      Model       Database    Session
 |           |              |            |            |           |
 |--login--> |              |            |            |           |
 |           |--validate--> |            |            |           |
 |           |              |--login()--> |            |           |
 |           |              |            |--query---> |           |
 |           |              |            |<--result-- |           |
 |           |              |<--boolean- |            |           |
 |           |              |--store-------------------> |           |
 |           |<--redirect-- |            |            |           |
 |<--page--- |              |            |            |           |
```

### 2.2 Analisis User Model

#### Struktur Class User
```php
// models/User.php
class User {
    private $conn;
    private $table_name = "users";
    
    public $id;
    public $username;
    public $email;
    public $password;
    public $role;
    public $created_at;
}
```

#### Properties dan Fungsinya
- **$conn**: Koneksi database
- **$table_name**: Nama tabel dalam database
- **Public properties**: Menyimpan data user

#### Method Utama User Model

1. **Constructor**
```php
public function __construct() {
    $database = new Database();
    $this->conn = $database->getConnection();
    
    // Validasi koneksi database
    if (!$this->conn) {
        throw new Exception("Database connection failed");
    }
}
```
**Fungsi**: 
- Inisialisasi koneksi database menggunakan PDO
- Validasi koneksi untuk memastikan database tersedia
- Menggunakan dependency injection pattern untuk Database class

2. **Register Method**
```php
public function register() {
    $query = "INSERT INTO " . $this->table_name . " 
             SET username=:username, email=:email, password=:password";
    
    $stmt = $this->conn->prepare($query);
    
    // Sanitasi input
    $this->username = htmlspecialchars(strip_tags($this->username));
    $this->email = htmlspecialchars(strip_tags($this->email));
    $this->password = password_hash($this->password, PASSWORD_DEFAULT);
    
    // Bind parameters
    $stmt->bindParam(":username", $this->username);
    $stmt->bindParam(":email", $this->email);
    $stmt->bindParam(":password", $this->password);
    
    return $stmt->execute();
}
```
**Fungsi**: 
- Mendaftarkan user baru dengan validasi input
- Hash password menggunakan algoritma bcrypt
- Sanitasi input untuk mencegah XSS attacks
- Menggunakan prepared statements untuk mencegah SQL injection
- Return boolean untuk status keberhasilan operasi

**Security Features**:
- Password hashing dengan `PASSWORD_DEFAULT` (bcrypt)
- Input sanitization dengan `htmlspecialchars()` dan `strip_tags()`
- Prepared statements untuk database queries
- Validation sebelum insert ke database

3. **Login Method**
```php
public function login($username, $password) {
    $query = "SELECT id, username, email, password, role FROM " . $this->table_name . " 
             WHERE username = :username OR email = :username LIMIT 1";
    
    $stmt = $this->conn->prepare($query);
    $stmt->bindParam(":username", $username);
    $stmt->execute();
    
    if($stmt->rowCount() > 0) {
        $row = $stmt->fetch(PDO::FETCH_ASSOC);
        
        if(password_verify($password, $row['password'])) {
            $this->id = $row['id'];
            $this->username = $row['username'];
            $this->email = $row['email'];
            $this->role = $row['role'];
            return true;
        }
    }
    return false;
}
```
**Fungsi**: 
- Memverifikasi kredensial user (username/email dan password)
- Menggunakan `password_verify()` untuk validasi password hash
- Mengisi object properties dengan data user dari database
- Support login dengan username atau email
- Return boolean untuk status login

**Security Features**:
- Password verification menggunakan built-in PHP function
- Prepared statements untuk query database
- Limit query result untuk optimasi
- Tidak menyimpan password dalam object properties

4. **Username/Email Exists Methods**
```php
public function usernameExists() {
    $query = "SELECT id FROM " . $this->table_name . " WHERE username = :username LIMIT 1";
    $stmt = $this->conn->prepare($query);
    $stmt->bindParam(":username", $this->username);
    $stmt->execute();
    
    return $stmt->rowCount() > 0;
}

public function emailExists() {
    $query = "SELECT id FROM " . $this->table_name . " WHERE email = :email LIMIT 1";
    $stmt = $this->conn->prepare($query);
    $stmt->bindParam(":email", $this->email);
    $stmt->execute();
    
    return $stmt->rowCount() > 0;
}
```
**Fungsi**: 
- Mengecek apakah username atau email sudah terdaftar
- Mencegah duplikasi data user
- Menggunakan prepared statements untuk keamanan
- Return boolean untuk status keberadaan data

5. **Get User by ID Method**
```php
public function getUserById($id) {
    $query = "SELECT id, username, email, role, created_at FROM " . $this->table_name . " WHERE id = :id LIMIT 1";
    $stmt = $this->conn->prepare($query);
    $stmt->bindParam(":id", $id);
    $stmt->execute();
    
    if($stmt->rowCount() > 0) {
        $row = $stmt->fetch(PDO::FETCH_ASSOC);
        $this->id = $row['id'];
        $this->username = $row['username'];
        $this->email = $row['email'];
        $this->role = $row['role'];
        $this->created_at = $row['created_at'];
        return true;
    }
    return false;
}
```
**Fungsi**: 
- Mengambil data user berdasarkan ID
- Mengisi object properties dengan data dari database
- Tidak mengambil password untuk keamanan
- Return boolean untuk status keberhasilan

### 2.3 Analisis AuthController

#### Struktur AuthController
```php
// controllers/AuthController.php
class AuthController {
    private $user;
    private $errors = [];
    
    public function __construct() {
        $this->user = new User();
        
        // Start session jika belum dimulai
        if (session_status() == PHP_SESSION_NONE) {
            session_start();
        }
    }
    
    // Getter untuk errors
    public function getErrors() {
        return $this->errors;
    }
    
    // Method untuk menambah error
    private function addError($message) {
        $this->errors[] = $message;
    }
    
    // Method untuk clear errors
    public function clearErrors() {
        $this->errors = [];
    }
}
```

#### Method Utama AuthController

1. **Login Method**
```php
public function login($username, $password) {
    // Clear previous errors
    $this->clearErrors();
    
    // Validasi input
    if (empty($username)) {
        $this->addError("Username atau email tidak boleh kosong");
        return false;
    }
    
    if (empty($password)) {
        $this->addError("Password tidak boleh kosong");
        return false;
    }
    
    // Sanitasi input
    $username = htmlspecialchars(strip_tags(trim($username)));
    
    // Attempt login
    if($this->user->login($username, $password)) {
        // Regenerate session ID untuk keamanan
        session_regenerate_id(true);
        
        // Set session variables
        $_SESSION['user_id'] = $this->user->id;
        $_SESSION['username'] = $this->user->username;
        $_SESSION['email'] = $this->user->email;
        $_SESSION['role'] = $this->user->role;
        $_SESSION['logged_in'] = true;
        $_SESSION['login_time'] = time();
        
        // Log successful login (optional)
        error_log("User login successful: " . $this->user->username);
        
        return true;
    } else {
        $this->addError("Username/email atau password salah");
        
        // Log failed login attempt
        error_log("Failed login attempt for: " . $username);
        
        return false;
    }
}
```
**Fungsi**: 
- Mengelola proses login dengan validasi input lengkap
- Sanitasi input untuk mencegah XSS
- Session management dengan regeneration untuk keamanan
- Error handling dan logging
- Return boolean dengan error messages

2. **Register Method**
```php
public function register($username, $email, $password, $confirmPassword = null) {
    // Clear previous errors
    $this->clearErrors();
    
    // Validasi input
    if (!$this->validateRegistrationInput($username, $email, $password, $confirmPassword)) {
        return ['success' => false, 'errors' => $this->getErrors()];
    }
    
    // Sanitasi input
    $username = htmlspecialchars(strip_tags(trim($username)));
    $email = htmlspecialchars(strip_tags(trim($email)));
    
    // Set user properties
    $this->user->username = $username;
    $this->user->email = $email;
    $this->user->password = $password;
    
    // Cek duplikasi username
    if($this->user->usernameExists()) {
        $this->addError('Username sudah digunakan');
        return ['success' => false, 'errors' => $this->getErrors()];
    }
    
    // Cek duplikasi email
    if($this->user->emailExists()) {
        $this->addError('Email sudah digunakan');
        return ['success' => false, 'errors' => $this->getErrors()];
    }
    
    // Attempt registration
    if($this->user->register()) {
        // Log successful registration
        error_log("User registration successful: " . $username);
        
        return ['success' => true, 'message' => 'Registrasi berhasil! Silakan login.'];
    } else {
        $this->addError('Registrasi gagal. Silakan coba lagi.');
        
        // Log failed registration
        error_log("User registration failed for: " . $username);
        
        return ['success' => false, 'errors' => $this->getErrors()];
    }
}

// Method helper untuk validasi input registrasi
private function validateRegistrationInput($username, $email, $password, $confirmPassword) {
    $isValid = true;
    
    // Validasi username
    if (empty($username)) {
        $this->addError("Username tidak boleh kosong");
        $isValid = false;
    } elseif (strlen($username) < 3) {
        $this->addError("Username minimal 3 karakter");
        $isValid = false;
    } elseif (strlen($username) > 50) {
        $this->addError("Username maksimal 50 karakter");
        $isValid = false;
    } elseif (!preg_match('/^[a-zA-Z0-9_]+$/', $username)) {
        $this->addError("Username hanya boleh mengandung huruf, angka, dan underscore");
        $isValid = false;
    }
    
    // Validasi email
    if (empty($email)) {
        $this->addError("Email tidak boleh kosong");
        $isValid = false;
    } elseif (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
        $this->addError("Format email tidak valid");
        $isValid = false;
    }
    
    // Validasi password
    if (empty($password)) {
        $this->addError("Password tidak boleh kosong");
        $isValid = false;
    } elseif (strlen($password) < 6) {
        $this->addError("Password minimal 6 karakter");
        $isValid = false;
    } elseif (strlen($password) > 255) {
        $this->addError("Password terlalu panjang");
        $isValid = false;
    }
    
    // Validasi konfirmasi password
    if ($confirmPassword !== null && $password !== $confirmPassword) {
        $this->addError("Konfirmasi password tidak cocok");
        $isValid = false;
    }
    
    return $isValid;
}
```
**Fungsi**: 
- Mengelola proses registrasi dengan validasi input komprehensif
- Validasi format username, email, dan password
- Sanitasi input untuk keamanan
- Error handling dengan multiple error messages
- Logging untuk monitoring
- Return array dengan status dan error details

3. **Logout Method**
```php
public function logout() {
    // Log logout activity
    if (isset($_SESSION['username'])) {
        error_log("User logout: " . $_SESSION['username']);
    }
    
    // Clear all session variables
    $_SESSION = array();
    
    // Delete session cookie if exists
    if (ini_get("session.use_cookies")) {
        $params = session_get_cookie_params();
        setcookie(session_name(), '', time() - 42000,
            $params["path"], $params["domain"],
            $params["secure"], $params["httponly"]
        );
    }
    
    // Destroy session
    session_destroy();
    
    return true;
}
```
**Fungsi**: 
- Menghapus session dan logout user dengan aman
- Logging aktivitas logout
- Menghapus session cookie untuk keamanan
- Complete session cleanup

4. **Session Management Methods**
```php
public function isLoggedIn() {
    return isset($_SESSION['logged_in']) && $_SESSION['logged_in'] === true;
}

public function requireLogin() {
    if (!$this->isLoggedIn()) {
        header('Location: login.php?redirect=' . urlencode($_SERVER['REQUEST_URI']));
        exit();
    }
}

public function requireAdmin() {
    $this->requireLogin();
    if (!$this->isAdmin()) {
        header('Location: ../user/home.php?error=access_denied');
        exit();
    }
}

public function isAdmin() {
    return $this->isLoggedIn() && isset($_SESSION['role']) && $_SESSION['role'] === 'admin';
}

public function getCurrentUser() {
    if (!$this->isLoggedIn()) {
        return null;
    }
    
    return [
        'id' => $_SESSION['user_id'] ?? null,
        'username' => $_SESSION['username'] ?? null,
        'email' => $_SESSION['email'] ?? null,
        'role' => $_SESSION['role'] ?? null,
        'login_time' => $_SESSION['login_time'] ?? null
    ];
}

public function getSessionDuration() {
    if (!$this->isLoggedIn() || !isset($_SESSION['login_time'])) {
        return 0;
    }
    
    return time() - $_SESSION['login_time'];
}
```
**Fungsi**: 
- Mengecek status login user
- Memaksa login untuk halaman yang memerlukan autentikasi
- Role-based access control
- Mendapatkan informasi user yang sedang login
- Monitoring durasi session

### 2.4 Session Management

#### Konsep Session
- Session menyimpan data user selama browsing
- Data tersimpan di server, bukan di browser
- Digunakan untuk tracking status login

#### Session Variables
```php
$_SESSION['user_id']    // ID user
$_SESSION['username']   // Username
$_SESSION['role']       // Role (admin/user)
$_SESSION['logged_in']  // Status login
```

#### Checking Login Status
```php
public function isLoggedIn() {
    return isset($_SESSION['logged_in']) && $_SESSION['logged_in'] === true;
}

public function requireLogin() {
    if (!$this->isLoggedIn()) {
        header('Location: login.php');
        exit();
    }
}

public function requireAdmin() {
    $this->requireLogin();
    if ($_SESSION['role'] !== 'admin') {
        header('Location: ../user/home.php');
        exit();
    }
}
```

### 2.5 Security Implementation

#### Password Hashing
```php
// Saat register
$this->password = password_hash($this->password, PASSWORD_DEFAULT);

// Saat login
if(password_verify($password, $row['password'])) {
    // Login berhasil
}
```

#### Input Sanitization
```php
$this->username = htmlspecialchars(strip_tags($this->username));
$this->email = htmlspecialchars(strip_tags($this->email));
```

#### SQL Injection Prevention
```php
$stmt = $this->conn->prepare($query);
$stmt->bindParam(":username", $username);
```

### 2.6 View Implementation

#### Login Form
```php
// views/user/login.php
<form method="POST" action="">
    <input type="text" name="username" placeholder="Username atau Email" required>
    <input type="password" name="password" placeholder="Password" required>
    <button type="submit" name="login">Login</button>
</form>

<?php
if(isset($_POST['login'])) {
    $auth = new AuthController();
    if($auth->login($_POST['username'], $_POST['password'])) {
        header('Location: home.php');
    } else {
        echo "Login gagal!";
    }
}
?>
```

#### Register Form
```php
// views/user/register.php
<form method="POST" action="">
    <input type="text" name="username" placeholder="Username" required>
    <input type="email" name="email" placeholder="Email" required>
    <input type="password" name="password" placeholder="Password" required>
    <button type="submit" name="register">Daftar</button>
</form>

<?php
if(isset($_POST['register'])) {
    $auth = new AuthController();
    $result = $auth->register($_POST['username'], $_POST['email'], $_POST['password']);
    echo $result['message'];
}
?>
```

## Praktikum

### Latihan 1: Analisis User Model
1. Buka file `models/User.php`
2. Identifikasi semua method yang ada
3. Jelaskan fungsi masing-masing method
4. Buat diagram class User

### Latihan 2: Testing Autentikasi
1. Buat user baru melalui form register
2. Login menggunakan user yang baru dibuat
3. Cek session yang tersimpan
4. Test logout functionality

### Latihan 3: Implementasi Validasi Lanjutan
1. Tambahkan validasi password yang lebih kuat
2. Implementasikan rate limiting untuk login
3. Tambahkan validasi CSRF token

```php
// Enhanced password validation
public function validateStrongPassword($password) {
    $errors = [];
    
    if (strlen($password) < 8) {
        $errors[] = 'Password minimal 8 karakter';
    }
    
    if (!preg_match('/[A-Z]/', $password)) {
        $errors[] = 'Password harus mengandung huruf besar';
    }
    
    if (!preg_match('/[a-z]/', $password)) {
        $errors[] = 'Password harus mengandung huruf kecil';
    }
    
    if (!preg_match('/[0-9]/', $password)) {
        $errors[] = 'Password harus mengandung angka';
    }
    
    if (!preg_match('/[^A-Za-z0-9]/', $password)) {
        $errors[] = 'Password harus mengandung karakter khusus';
    }
    
    return empty($errors) ? ['valid' => true] : ['valid' => false, 'errors' => $errors];
}

// Rate limiting implementation
public function checkLoginAttempts($username) {
    $maxAttempts = 5;
    $timeWindow = 900; // 15 minutes
    
    // Check attempts from session or database
    $attempts = $_SESSION['login_attempts'][$username] ?? [];
    
    // Remove old attempts
    $attempts = array_filter($attempts, function($time) use ($timeWindow) {
        return (time() - $time) < $timeWindow;
    });
    
    if (count($attempts) >= $maxAttempts) {
        return ['allowed' => false, 'message' => 'Terlalu banyak percobaan login. Coba lagi dalam 15 menit.'];
    }
    
    return ['allowed' => true];
}

public function recordLoginAttempt($username) {
    $_SESSION['login_attempts'][$username][] = time();
}

// CSRF Token implementation
public function generateCSRFToken() {
    if (!isset($_SESSION['csrf_token'])) {
        $_SESSION['csrf_token'] = bin2hex(random_bytes(32));
    }
    return $_SESSION['csrf_token'];
}

public function validateCSRFToken($token) {
    return isset($_SESSION['csrf_token']) && hash_equals($_SESSION['csrf_token'], $token);
}
```

### Latihan 4: Advanced Security Implementation
1. Implementasikan password strength meter
2. Tambahkan two-factor authentication (2FA) simulation
3. Buat audit log untuk aktivitas user

```php
// Password strength meter
public function calculatePasswordStrength($password) {
    $score = 0;
    $feedback = [];
    
    // Length check
    if (strlen($password) >= 8) $score += 25;
    else $feedback[] = 'Tambahkan lebih banyak karakter';
    
    // Character variety
    if (preg_match('/[a-z]/', $password)) $score += 25;
    if (preg_match('/[A-Z]/', $password)) $score += 25;
    if (preg_match('/[0-9]/', $password)) $score += 25;
    if (preg_match('/[^A-Za-z0-9]/', $password)) $score += 25;
    
    // Bonus for length
    if (strlen($password) >= 12) $score += 10;
    
    $strength = 'Weak';
    if ($score >= 60) $strength = 'Medium';
    if ($score >= 80) $strength = 'Strong';
    if ($score >= 100) $strength = 'Very Strong';
    
    return [
        'score' => min($score, 100),
        'strength' => $strength,
        'feedback' => $feedback
    ];
}

// Audit logging
public function logUserActivity($action, $details = []) {
    $logData = [
        'timestamp' => date('Y-m-d H:i:s'),
        'user_id' => $_SESSION['user_id'] ?? null,
        'username' => $_SESSION['username'] ?? 'guest',
        'action' => $action,
        'ip_address' => $_SERVER['REMOTE_ADDR'] ?? 'unknown',
        'user_agent' => $_SERVER['HTTP_USER_AGENT'] ?? 'unknown',
        'details' => $details
    ];
    
    // Log to file or database
    error_log('USER_ACTIVITY: ' . json_encode($logData));
}

// Session security
public function validateSession() {
    // Check session timeout (30 minutes)
    if (isset($_SESSION['login_time']) && (time() - $_SESSION['login_time']) > 1800) {
        $this->logout();
        return false;
    }
    
    // Check IP address consistency
    if (isset($_SESSION['ip_address']) && $_SESSION['ip_address'] !== $_SERVER['REMOTE_ADDR']) {
        $this->logout();
        return false;
    }
    
    // Update last activity
    $_SESSION['last_activity'] = time();
    
    return true;
}
```

## Tugas

### Tugas Individu

1. **Analisis Mendalam Sistem Autentikasi**:
   - Buat dokumentasi lengkap tentang flow autentikasi dari input user hingga response
   - Analisis setiap method dalam User model dan AuthController
   - Buat diagram sequence untuk proses login dan register
   - Identifikasi potential security vulnerabilities

2. **Implementation Enhancement**:
   - Tambahkan fitur "Remember Me" pada login dengan secure cookie
   - Implementasikan password reset functionality
   - Buat sistem notifikasi email untuk aktivitas login
   - Tambahkan profile management untuk user

3. **Security Hardening**:
   - Implementasikan rate limiting untuk login attempts
   - Tambahkan CAPTCHA simulation untuk multiple failed attempts
   - Buat audit log system untuk tracking user activities
   - Implementasikan session timeout dan concurrent session control

4. **Advanced Validation System**:
   - Buat comprehensive input validation dengan custom rules
   - Implementasikan real-time password strength indicator
   - Tambahkan email verification untuk registration
   - Buat blacklist system untuk common passwords

### Tugas Kelompok

1. **Security Assessment**:
   - Lakukan penetration testing pada sistem autentikasi
   - Buat laporan vulnerability assessment
   - Diskusikan OWASP Top 10 dalam konteks aplikasi ini
   - Presentasikan findings dan recommendations

2. **Performance Optimization**:
   - Analisis performance bottlenecks dalam sistem autentikasi
   - Implementasikan caching strategy untuk user sessions
   - Optimize database queries untuk user operations
   - Buat load testing scenario

### Mini Project: Advanced Authentication System

Buat sistem autentikasi yang lebih advanced dengan fitur:
- Multi-factor authentication (2FA)
- Social login integration simulation
- Role-based permissions dengan granular control
- API authentication dengan JWT tokens
- Real-time session monitoring dashboard

## Evaluasi

### Checklist Penyelesaian

**Pemahaman Konsep:**
- [ ] Memahami struktur dan fungsi User model
- [ ] Memahami implementasi AuthController
- [ ] Memahami flow autentikasi lengkap
- [ ] Memahami session management dan security
- [ ] Memahami role-based access control
- [ ] Memahami password hashing dan verification

**Implementasi Praktis:**
- [ ] Dapat mengimplementasikan login/register dengan validasi
- [ ] Dapat membuat sistem error handling
- [ ] Dapat implementasi CSRF protection
- [ ] Dapat membuat rate limiting system
- [ ] Dapat implementasi audit logging
- [ ] Dapat membuat session security measures

**Security Understanding:**
- [ ] Memahami common web vulnerabilities
- [ ] Dapat implementasi input sanitization
- [ ] Memahami secure session management
- [ ] Dapat implementasi password security
- [ ] Memahami OWASP security principles

### Pertanyaan Review

**Konsep Dasar:**
1. Mengapa password perlu di-hash? Jelaskan perbedaan antara hashing dan encryption!
2. Apa perbedaan session dan cookie? Kapan menggunakan masing-masing?
3. Bagaimana cara kerja `password_verify()` dan mengapa lebih aman dari comparison biasa?
4. Jelaskan konsep salt dalam password hashing!

**Security Implementation:**
5. Bagaimana cara mencegah SQL injection? Berikan 3 metode berbeda!
6. Apa itu XSS attack dan bagaimana cara mencegahnya?
7. Jelaskan implementasi CSRF protection dalam aplikasi web!
8. Mengapa perlu session regeneration setelah login?

**Advanced Topics:**
9. Bagaimana implementasi rate limiting untuk mencegah brute force attack?
10. Apa itu timing attack dan bagaimana cara mencegahnya?
11. Jelaskan konsep principle of least privilege dalam role-based access!
12. Bagaimana cara implementasi secure "Remember Me" functionality?

**Troubleshooting:**
13. Apa yang harus dilakukan jika user lupa password?
14. Bagaimana menangani concurrent login sessions?
15. Apa yang harus dilakukan jika terdeteksi suspicious login activity?

### Kriteria Penilaian

**Excellent (90-100):**
- Implementasi autentikasi sempurna dengan semua security measures
- Dapat menjelaskan semua konsep security dengan detail
- Code quality tinggi dengan proper error handling
- Dapat mengidentifikasi dan mengatasi security vulnerabilities
- Dokumentasi lengkap dan comprehensive

**Good (80-89):**
- Implementasi autentikasi berfungsi dengan sebagian besar security measures
- Memahami konsep security utama
- Code quality baik dengan basic error handling
- Dapat mengidentifikasi common vulnerabilities
- Dokumentasi cukup lengkap

**Satisfactory (70-79):**
- Implementasi autentikasi basic berfungsi
- Memahami konsep dasar security
- Code berfungsi dengan minimal error handling
- Memahami beberapa security issues
- Dokumentasi basic

**Needs Improvement (<70):**
- Implementasi autentikasi tidak lengkap atau bermasalah
- Pemahaman security kurang
- Code quality rendah dengan banyak issues
- Tidak dapat mengidentifikasi security problems
- Dokumentasi tidak memadai

### Assessment Methods

1. **Code Review (40%)**
   - Quality dan security dari implementasi
   - Proper error handling dan validation
   - Code organization dan documentation

2. **Security Analysis (30%)**
   - Identification of vulnerabilities
   - Implementation of security measures
   - Understanding of security principles

3. **Practical Testing (20%)**
   - Functionality testing
   - Security testing
   - Performance testing

4. **Documentation & Presentation (10%)**
   - Quality of documentation
   - Clarity of explanation
   - Demonstration of understanding

### Resources Tambahan

**Security Resources:**
- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [PHP Security Guide](https://phpsec.org/)
- [Session Security](https://www.php.net/manual/en/session.security.php)

**Best Practices:**
- [PHP Password Hashing](https://www.php.net/manual/en/faq.passwords.php)
- [Input Validation](https://cheatsheetseries.owasp.org/cheatsheets/Input_Validation_Cheat_Sheet.html)
- [Authentication Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html)

**Tools untuk Testing:**
- [Burp Suite Community](https://portswigger.net/burp/communitydownload)
- [OWASP ZAP](https://www.zaproxy.org/)
- [SQLMap](http://sqlmap.org/)

---

**Catatan**: Sistem autentikasi adalah fondasi keamanan aplikasi web. Pastikan memahami semua konsep security dan best practices sebelum melanjutkan ke bab selanjutnya. Jangan ragu untuk melakukan testing security secara menyeluruh pada implementasi Anda.