# BAB 6: KEAMANAN DAN OPTIMASI APLIKASI

---

**📚 Modul Pembelajaran Object-Oriented Programming dengan PHP**

**Penulis:** Herisvan Hendra, M.Pd.T  
**BAB:** 6 - Keamanan dan Optimasi  
**Estimasi Waktu:** 3-4 jam  
**Level:** Advanced

---

## Deskripsi
Bab ini membahas aspek keamanan dan optimasi dalam aplikasi Game Store menggunakan konsep OOP. Mahasiswa akan mempelajari implementasi security best practices, optimasi database, caching, dan performance tuning untuk aplikasi e-commerce yang aman dan efisien.

## Tujuan Pembelajaran
Setelah menyelesaikan bab ini, mahasiswa diharapkan dapat:
1. Memahami konsep keamanan aplikasi web
2. Mengimplementasikan input validation dan sanitization
3. Memahami SQL injection prevention
4. Mengimplementasikan session security
5. Memahami teknik optimasi database
6. Mengimplementasikan caching strategies
7. Melakukan performance monitoring dan tuning

## Materi

### 6.1 Keamanan Aplikasi Web

#### Konsep Dasar Security
1. **Authentication**: Verifikasi identitas user
2. **Authorization**: Kontrol akses berdasarkan role
3. **Input Validation**: Validasi data input
4. **Output Encoding**: Encoding data output
5. **Session Management**: Pengelolaan session yang aman
6. **Error Handling**: Penanganan error yang aman

#### OWASP Top 10 Web Application Security Risks
1. **Injection** (SQL, NoSQL, OS, LDAP)
2. **Broken Authentication**
3. **Sensitive Data Exposure**
4. **XML External Entities (XXE)**
5. **Broken Access Control**
6. **Security Misconfiguration**
7. **Cross-Site Scripting (XSS)**
8. **Insecure Deserialization**
9. **Using Components with Known Vulnerabilities**
10. **Insufficient Logging & Monitoring**

### 6.2 Input Validation dan Sanitization

#### Implementasi Validation Class
```php
// utils/Validator.php
class Validator {
    private $errors = [];
    
    public function validateRequired($value, $fieldName) {
        if (empty(trim($value))) {
            $this->errors[$fieldName] = ucfirst($fieldName) . ' is required';
            return false;
        }
        return true;
    }
    
    public function validateEmail($email, $fieldName = 'email') {
        if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
            $this->errors[$fieldName] = 'Invalid email format';
            return false;
        }
        return true;
    }
    
    public function validateLength($value, $min, $max, $fieldName) {
        $length = strlen(trim($value));
        if ($length < $min || $length > $max) {
            $this->errors[$fieldName] = ucfirst($fieldName) . " must be between {$min} and {$max} characters";
            return false;
        }
        return true;
    }
    
    public function validateNumeric($value, $fieldName, $min = null, $max = null) {
        if (!is_numeric($value)) {
            $this->errors[$fieldName] = ucfirst($fieldName) . ' must be a number';
            return false;
        }
        
        $numValue = (float)$value;
        
        if ($min !== null && $numValue < $min) {
            $this->errors[$fieldName] = ucfirst($fieldName) . " must be at least {$min}";
            return false;
        }
        
        if ($max !== null && $numValue > $max) {
            $this->errors[$fieldName] = ucfirst($fieldName) . " must not exceed {$max}";
            return false;
        }
        
        return true;
    }
    
    public function validatePassword($password, $fieldName = 'password') {
        // Minimum 8 characters, at least one letter and one number
        if (!preg_match('/^(?=.*[A-Za-z])(?=.*\d)[A-Za-z\d@$!%*#?&]{8,}$/', $password)) {
            $this->errors[$fieldName] = 'Password must be at least 8 characters with letters and numbers';
            return false;
        }
        return true;
    }
    
    public function validateFileUpload($file, $allowedTypes, $maxSize, $fieldName = 'file') {
        if ($file['error'] !== UPLOAD_ERR_OK) {
            $this->errors[$fieldName] = 'File upload error';
            return false;
        }
        
        // Check file size
        if ($file['size'] > $maxSize) {
            $this->errors[$fieldName] = 'File size exceeds maximum allowed size';
            return false;
        }
        
        // Check file type
        $fileType = mime_content_type($file['tmp_name']);
        if (!in_array($fileType, $allowedTypes)) {
            $this->errors[$fieldName] = 'Invalid file type';
            return false;
        }
        
        return true;
    }
    
    public function sanitizeString($input) {
        return htmlspecialchars(trim($input), ENT_QUOTES, 'UTF-8');
    }
    
    public function sanitizeEmail($email) {
        return filter_var(trim($email), FILTER_SANITIZE_EMAIL);
    }
    
    public function sanitizeInt($input) {
        return filter_var($input, FILTER_SANITIZE_NUMBER_INT);
    }
    
    public function sanitizeFloat($input) {
        return filter_var($input, FILTER_SANITIZE_NUMBER_FLOAT, FILTER_FLAG_ALLOW_FRACTION);
    }
    
    public function getErrors() {
        return $this->errors;
    }
    
    public function hasErrors() {
        return !empty($this->errors);
    }
    
    public function clearErrors() {
        $this->errors = [];
    }
}
```

#### Enhanced AuthController dengan Validation
```php
// controllers/AuthController.php (Enhanced)
class AuthController {
    private $user;
    private $validator;
    
    public function __construct() {
        $this->user = new User();
        $this->validator = new Validator();
    }
    
    public function register($username, $email, $password, $confirmPassword) {
        // Clear previous errors
        $this->validator->clearErrors();
        
        // Sanitize inputs
        $username = $this->validator->sanitizeString($username);
        $email = $this->validator->sanitizeEmail($email);
        
        // Validate inputs
        $this->validator->validateRequired($username, 'username');
        $this->validator->validateLength($username, 3, 50, 'username');
        
        $this->validator->validateRequired($email, 'email');
        $this->validator->validateEmail($email, 'email');
        
        $this->validator->validateRequired($password, 'password');
        $this->validator->validatePassword($password, 'password');
        
        if ($password !== $confirmPassword) {
            $this->validator->getErrors()['confirm_password'] = 'Passwords do not match';
        }
        
        // Check for validation errors
        if ($this->validator->hasErrors()) {
            return [
                'success' => false,
                'message' => 'Validation failed',
                'errors' => $this->validator->getErrors()
            ];
        }
        
        // Check if username or email already exists
        if ($this->user->usernameExists($username)) {
            return [
                'success' => false,
                'message' => 'Username already exists',
                'errors' => ['username' => 'Username already exists']
            ];
        }
        
        if ($this->user->emailExists($email)) {
            return [
                'success' => false,
                'message' => 'Email already exists',
                'errors' => ['email' => 'Email already exists']
            ];
        }
        
        // Hash password
        $hashedPassword = password_hash($password, PASSWORD_DEFAULT);
        
        // Register user
        if ($this->user->register($username, $email, $hashedPassword)) {
            return [
                'success' => true,
                'message' => 'Registration successful'
            ];
        }
        
        return [
            'success' => false,
            'message' => 'Registration failed'
        ];
    }
    
    public function login($username, $password) {
        // Rate limiting check
        if ($this->isRateLimited($username)) {
            return [
                'success' => false,
                'message' => 'Too many login attempts. Please try again later.'
            ];
        }
        
        // Sanitize inputs
        $username = $this->validator->sanitizeString($username);
        
        // Validate inputs
        $this->validator->clearErrors();
        $this->validator->validateRequired($username, 'username');
        $this->validator->validateRequired($password, 'password');
        
        if ($this->validator->hasErrors()) {
            return [
                'success' => false,
                'message' => 'Validation failed',
                'errors' => $this->validator->getErrors()
            ];
        }
        
        // Attempt login
        $user = $this->user->login($username, $password);
        
        if ($user) {
            // Clear failed attempts
            $this->clearFailedAttempts($username);
            
            // Regenerate session ID
            session_regenerate_id(true);
            
            // Set session variables
            $_SESSION['user_id'] = $user['id'];
            $_SESSION['username'] = $user['username'];
            $_SESSION['role'] = $user['role'];
            $_SESSION['login_time'] = time();
            $_SESSION['last_activity'] = time();
            
            // Log successful login
            $this->logActivity($user['id'], 'login', 'User logged in successfully');
            
            return [
                'success' => true,
                'message' => 'Login successful',
                'user' => $user
            ];
        }
        
        // Record failed attempt
        $this->recordFailedAttempt($username);
        
        return [
            'success' => false,
            'message' => 'Invalid username or password'
        ];
    }
    
    private function isRateLimited($username) {
        $maxAttempts = 5;
        $timeWindow = 900; // 15 minutes
        
        if (!isset($_SESSION['failed_attempts'])) {
            $_SESSION['failed_attempts'] = [];
        }
        
        $attempts = $_SESSION['failed_attempts'][$username] ?? [];
        $recentAttempts = array_filter($attempts, function($timestamp) use ($timeWindow) {
            return (time() - $timestamp) < $timeWindow;
        });
        
        return count($recentAttempts) >= $maxAttempts;
    }
    
    private function recordFailedAttempt($username) {
        if (!isset($_SESSION['failed_attempts'])) {
            $_SESSION['failed_attempts'] = [];
        }
        
        if (!isset($_SESSION['failed_attempts'][$username])) {
            $_SESSION['failed_attempts'][$username] = [];
        }
        
        $_SESSION['failed_attempts'][$username][] = time();
    }
    
    private function clearFailedAttempts($username) {
        if (isset($_SESSION['failed_attempts'][$username])) {
            unset($_SESSION['failed_attempts'][$username]);
        }
    }
    
    public function requireLogin() {
        if (!$this->isLoggedIn()) {
            header('Location: ../auth/login.php');
            exit();
        }
        
        // Check session timeout (30 minutes)
        $sessionTimeout = 1800;
        if (isset($_SESSION['last_activity']) && (time() - $_SESSION['last_activity']) > $sessionTimeout) {
            $this->logout();
            header('Location: ../auth/login.php?timeout=1');
            exit();
        }
        
        // Update last activity
        $_SESSION['last_activity'] = time();
    }
    
    public function logout() {
        if (isset($_SESSION['user_id'])) {
            $this->logActivity($_SESSION['user_id'], 'logout', 'User logged out');
        }
        
        // Clear session data
        $_SESSION = [];
        
        // Destroy session cookie
        if (ini_get("session.use_cookies")) {
            $params = session_get_cookie_params();
            setcookie(session_name(), '', time() - 42000,
                $params["path"], $params["domain"],
                $params["secure"], $params["httponly"]
            );
        }
        
        // Destroy session
        session_destroy();
    }
    
    private function logActivity($userId, $action, $description) {
        // Implementation for activity logging
        // This could be stored in database or log files
        error_log("[" . date('Y-m-d H:i:s') . "] User {$userId}: {$action} - {$description}");
    }
}
```

### 6.3 SQL Injection Prevention

#### Prepared Statements Implementation
```php
// models/User.php (Enhanced)
class User {
    private $conn;
    private $table_name = "users";
    
    public function __construct() {
        $database = new Database();
        $this->conn = $database->getConnection();
    }
    
    // Safe user registration
    public function register($username, $email, $password) {
        try {
            // Use prepared statement to prevent SQL injection
            $query = "INSERT INTO " . $this->table_name . " 
                     (username, email, password, role, created_at) 
                     VALUES (:username, :email, :password, 'user', NOW())";
            
            $stmt = $this->conn->prepare($query);
            
            // Bind parameters
            $stmt->bindParam(':username', $username, PDO::PARAM_STR);
            $stmt->bindParam(':email', $email, PDO::PARAM_STR);
            $stmt->bindParam(':password', $password, PDO::PARAM_STR);
            
            return $stmt->execute();
        } catch (PDOException $e) {
            error_log("Registration error: " . $e->getMessage());
            return false;
        }
    }
    
    // Safe user login
    public function login($username, $password) {
        try {
            $query = "SELECT id, username, email, password, role 
                     FROM " . $this->table_name . " 
                     WHERE username = :username OR email = :username 
                     LIMIT 1";
            
            $stmt = $this->conn->prepare($query);
            $stmt->bindParam(':username', $username, PDO::PARAM_STR);
            $stmt->execute();
            
            if ($stmt->rowCount() > 0) {
                $user = $stmt->fetch(PDO::FETCH_ASSOC);
                
                if (password_verify($password, $user['password'])) {
                    // Remove password from returned data
                    unset($user['password']);
                    return $user;
                }
            }
            
            return false;
        } catch (PDOException $e) {
            error_log("Login error: " . $e->getMessage());
            return false;
        }
    }
    
    // Safe user search with pagination
    public function searchUsers($searchTerm, $limit = 10, $offset = 0) {
        try {
            $searchTerm = "%{$searchTerm}%";
            
            $query = "SELECT id, username, email, role, created_at 
                     FROM " . $this->table_name . " 
                     WHERE username LIKE :search OR email LIKE :search 
                     ORDER BY created_at DESC 
                     LIMIT :limit OFFSET :offset";
            
            $stmt = $this->conn->prepare($query);
            $stmt->bindParam(':search', $searchTerm, PDO::PARAM_STR);
            $stmt->bindParam(':limit', $limit, PDO::PARAM_INT);
            $stmt->bindParam(':offset', $offset, PDO::PARAM_INT);
            $stmt->execute();
            
            return $stmt->fetchAll(PDO::FETCH_ASSOC);
        } catch (PDOException $e) {
            error_log("Search users error: " . $e->getMessage());
            return [];
        }
    }
    
    // Safe user update
    public function updateUser($userId, $data) {
        try {
            $allowedFields = ['username', 'email', 'role'];
            $updateFields = [];
            $params = [':id' => $userId];
            
            foreach ($data as $field => $value) {
                if (in_array($field, $allowedFields)) {
                    $updateFields[] = "{$field} = :{$field}";
                    $params[":{$field}"] = $value;
                }
            }
            
            if (empty($updateFields)) {
                return false;
            }
            
            $query = "UPDATE " . $this->table_name . " 
                     SET " . implode(', ', $updateFields) . " 
                     WHERE id = :id";
            
            $stmt = $this->conn->prepare($query);
            
            foreach ($params as $param => $value) {
                $stmt->bindValue($param, $value);
            }
            
            return $stmt->execute();
        } catch (PDOException $e) {
            error_log("Update user error: " . $e->getMessage());
            return false;
        }
    }
}
```

### 6.4 Cross-Site Scripting (XSS) Prevention

#### Output Encoding Functions
```php
// utils/Security.php
class Security {
    
    // Escape HTML output
    public static function escapeHtml($string) {
        return htmlspecialchars($string, ENT_QUOTES | ENT_HTML5, 'UTF-8');
    }
    
    // Escape for HTML attributes
    public static function escapeHtmlAttr($string) {
        return htmlspecialchars($string, ENT_QUOTES | ENT_HTML5, 'UTF-8');
    }
    
    // Escape for JavaScript
    public static function escapeJs($string) {
        return json_encode($string, JSON_HEX_TAG | JSON_HEX_APOS | JSON_HEX_QUOT | JSON_HEX_AMP);
    }
    
    // Escape for URL
    public static function escapeUrl($string) {
        return urlencode($string);
    }
    
    // Clean HTML with allowed tags
    public static function cleanHtml($html, $allowedTags = '<p><br><strong><em><ul><ol><li>') {
        return strip_tags($html, $allowedTags);
    }
    
    // Generate CSRF token
    public static function generateCsrfToken() {
        if (!isset($_SESSION['csrf_token'])) {
            $_SESSION['csrf_token'] = bin2hex(random_bytes(32));
        }
        return $_SESSION['csrf_token'];
    }
    
    // Verify CSRF token
    public static function verifyCsrfToken($token) {
        return isset($_SESSION['csrf_token']) && hash_equals($_SESSION['csrf_token'], $token);
    }
    
    // Generate secure random string
    public static function generateRandomString($length = 32) {
        return bin2hex(random_bytes($length / 2));
    }
    
    // Hash sensitive data
    public static function hashData($data, $salt = '') {
        return hash('sha256', $data . $salt);
    }
    
    // Secure file upload
    public static function secureFileUpload($file, $uploadDir, $allowedTypes, $maxSize) {
        $validator = new Validator();
        
        if (!$validator->validateFileUpload($file, $allowedTypes, $maxSize)) {
            return ['success' => false, 'errors' => $validator->getErrors()];
        }
        
        // Generate secure filename
        $extension = pathinfo($file['name'], PATHINFO_EXTENSION);
        $filename = self::generateRandomString(16) . '.' . $extension;
        $filepath = $uploadDir . '/' . $filename;
        
        // Create upload directory if it doesn't exist
        if (!is_dir($uploadDir)) {
            mkdir($uploadDir, 0755, true);
        }
        
        // Move uploaded file
        if (move_uploaded_file($file['tmp_name'], $filepath)) {
            return ['success' => true, 'filename' => $filename, 'filepath' => $filepath];
        }
        
        return ['success' => false, 'message' => 'Failed to upload file'];
    }
}
```

#### Safe Output in Views
```php
// Example in product listing
<?php foreach($products as $product): ?>
<div class="product-card">
    <img src="../assets/images/<?php echo Security::escapeHtmlAttr($product['image']); ?>" 
         alt="<?php echo Security::escapeHtmlAttr($product['name']); ?>">
    <h3><?php echo Security::escapeHtml($product['name']); ?></h3>
    <p class="price">Rp <?php echo number_format($product['price'], 0, ',', '.'); ?></p>
    <p class="description"><?php echo Security::cleanHtml($product['description']); ?></p>
    
    <form method="POST" action="../controllers/cart_action.php">
        <input type="hidden" name="csrf_token" value="<?php echo Security::generateCsrfToken(); ?>">
        <input type="hidden" name="action" value="add">
        <input type="hidden" name="product_id" value="<?php echo (int)$product['id']; ?>">
        <input type="number" name="quantity" value="1" min="1" max="<?php echo (int)$product['stock']; ?>">
        <button type="submit" class="btn btn-primary">Add to Cart</button>
    </form>
</div>
<?php endforeach; ?>
```

### 6.5 Session Security

#### Secure Session Configuration
```php
// config/session_config.php
class SessionConfig {
    public static function init() {
        // Prevent session fixation
        ini_set('session.use_strict_mode', 1);
        
        // Use secure cookies
        ini_set('session.cookie_secure', 1);
        ini_set('session.cookie_httponly', 1);
        ini_set('session.cookie_samesite', 'Strict');
        
        // Set session lifetime
        ini_set('session.gc_maxlifetime', 1800); // 30 minutes
        ini_set('session.cookie_lifetime', 0); // Until browser closes
        
        // Use strong session ID
        ini_set('session.entropy_length', 32);
        ini_set('session.hash_function', 'sha256');
        
        // Start session
        if (session_status() === PHP_SESSION_NONE) {
            session_start();
        }
        
        // Regenerate session ID periodically
        if (!isset($_SESSION['created'])) {
            $_SESSION['created'] = time();
        } else if (time() - $_SESSION['created'] > 1800) {
            session_regenerate_id(true);
            $_SESSION['created'] = time();
        }
    }
}
```

### 6.6 Database Optimization

#### Query Optimization
```php
// models/Product.php (Optimized)
class Product {
    private $conn;
    private $table_name = "products";
    
    // Optimized product search with indexing
    public function searchProducts($searchTerm, $category = null, $minPrice = null, $maxPrice = null, $limit = 12, $offset = 0) {
        try {
            $conditions = [];
            $params = [];
            
            // Build WHERE conditions
            if (!empty($searchTerm)) {
                $conditions[] = "(name LIKE :search OR description LIKE :search)";
                $params[':search'] = "%{$searchTerm}%";
            }
            
            if (!empty($category)) {
                $conditions[] = "category = :category";
                $params[':category'] = $category;
            }
            
            if ($minPrice !== null) {
                $conditions[] = "price >= :min_price";
                $params[':min_price'] = $minPrice;
            }
            
            if ($maxPrice !== null) {
                $conditions[] = "price <= :max_price";
                $params[':max_price'] = $maxPrice;
            }
            
            $whereClause = !empty($conditions) ? 'WHERE ' . implode(' AND ', $conditions) : '';
            
            // Use index on (category, price, name) for better performance
            $query = "SELECT id, name, description, price, image, category, stock 
                     FROM " . $this->table_name . " 
                     {$whereClause} 
                     ORDER BY created_at DESC 
                     LIMIT :limit OFFSET :offset";
            
            $stmt = $this->conn->prepare($query);
            
            // Bind parameters
            foreach ($params as $param => $value) {
                $stmt->bindValue($param, $value);
            }
            $stmt->bindValue(':limit', $limit, PDO::PARAM_INT);
            $stmt->bindValue(':offset', $offset, PDO::PARAM_INT);
            
            $stmt->execute();
            return $stmt->fetchAll(PDO::FETCH_ASSOC);
        } catch (PDOException $e) {
            error_log("Search products error: " . $e->getMessage());
            return [];
        }
    }
    
    // Optimized related products query
    public function getRelatedProducts($productId, $category, $limit = 4) {
        try {
            $query = "SELECT id, name, price, image 
                     FROM " . $this->table_name . " 
                     WHERE category = :category AND id != :product_id AND stock > 0 
                     ORDER BY RAND() 
                     LIMIT :limit";
            
            $stmt = $this->conn->prepare($query);
            $stmt->bindParam(':category', $category, PDO::PARAM_STR);
            $stmt->bindParam(':product_id', $productId, PDO::PARAM_INT);
            $stmt->bindParam(':limit', $limit, PDO::PARAM_INT);
            $stmt->execute();
            
            return $stmt->fetchAll(PDO::FETCH_ASSOC);
        } catch (PDOException $e) {
            error_log("Get related products error: " . $e->getMessage());
            return [];
        }
    }
    
    // Batch update for better performance
    public function updateMultipleProducts($updates) {
        try {
            $this->conn->beginTransaction();
            
            $query = "UPDATE " . $this->table_name . " 
                     SET stock = :stock, price = :price 
                     WHERE id = :id";
            $stmt = $this->conn->prepare($query);
            
            foreach ($updates as $update) {
                $stmt->bindParam(':stock', $update['stock'], PDO::PARAM_INT);
                $stmt->bindParam(':price', $update['price'], PDO::PARAM_STR);
                $stmt->bindParam(':id', $update['id'], PDO::PARAM_INT);
                $stmt->execute();
            }
            
            $this->conn->commit();
            return true;
        } catch (PDOException $e) {
            $this->conn->rollback();
            error_log("Batch update error: " . $e->getMessage());
            return false;
        }
    }
}
```

#### Database Indexing Strategy
```sql
-- Recommended indexes for better performance

-- Users table indexes
CREATE INDEX idx_users_username ON users(username);
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_role ON users(role);

-- Products table indexes
CREATE INDEX idx_products_category ON products(category);
CREATE INDEX idx_products_price ON products(price);
CREATE INDEX idx_products_stock ON products(stock);
CREATE INDEX idx_products_name ON products(name);
CREATE FULLTEXT INDEX idx_products_search ON products(name, description);
CREATE INDEX idx_products_category_price ON products(category, price);

-- Orders table indexes
CREATE INDEX idx_orders_user_id ON orders(user_id);
CREATE INDEX idx_orders_status ON orders(status);
CREATE INDEX idx_orders_created_at ON orders(created_at);
CREATE INDEX idx_orders_user_status ON orders(user_id, status);

-- Order items table indexes
CREATE INDEX idx_order_items_order_id ON order_items(order_id);
CREATE INDEX idx_order_items_product_id ON order_items(product_id);
CREATE INDEX idx_order_items_order_product ON order_items(order_id, product_id);
```

### 6.7 Caching Implementation

#### Simple File-based Cache
```php
// utils/Cache.php
class Cache {
    private $cacheDir;
    private $defaultTtl;
    
    public function __construct($cacheDir = '../cache', $defaultTtl = 3600) {
        $this->cacheDir = $cacheDir;
        $this->defaultTtl = $defaultTtl;
        
        // Create cache directory if it doesn't exist
        if (!is_dir($this->cacheDir)) {
            mkdir($this->cacheDir, 0755, true);
        }
    }
    
    public function get($key) {
        $filename = $this->getCacheFilename($key);
        
        if (!file_exists($filename)) {
            return null;
        }
        
        $data = file_get_contents($filename);
        $cache = json_decode($data, true);
        
        if ($cache === null || $cache['expires'] < time()) {
            $this->delete($key);
            return null;
        }
        
        return $cache['data'];
    }
    
    public function set($key, $data, $ttl = null) {
        $ttl = $ttl ?? $this->defaultTtl;
        $filename = $this->getCacheFilename($key);
        
        $cache = [
            'data' => $data,
            'expires' => time() + $ttl,
            'created' => time()
        ];
        
        return file_put_contents($filename, json_encode($cache)) !== false;
    }
    
    public function delete($key) {
        $filename = $this->getCacheFilename($key);
        
        if (file_exists($filename)) {
            return unlink($filename);
        }
        
        return true;
    }
    
    public function clear() {
        $files = glob($this->cacheDir . '/*.cache');
        
        foreach ($files as $file) {
            unlink($file);
        }
        
        return true;
    }
    
    public function remember($key, $callback, $ttl = null) {
        $data = $this->get($key);
        
        if ($data === null) {
            $data = $callback();
            $this->set($key, $data, $ttl);
        }
        
        return $data;
    }
    
    private function getCacheFilename($key) {
        return $this->cacheDir . '/' . md5($key) . '.cache';
    }
    
    public function cleanup() {
        $files = glob($this->cacheDir . '/*.cache');
        $cleaned = 0;
        
        foreach ($files as $file) {
            $data = file_get_contents($file);
            $cache = json_decode($data, true);
            
            if ($cache && $cache['expires'] < time()) {
                unlink($file);
                $cleaned++;
            }
        }
        
        return $cleaned;
    }
}
```

#### Cached Product Controller
```php
// controllers/ProductController.php (with caching)
class ProductController {
    private $product;
    private $cache;
    
    public function __construct() {
        $this->product = new Product();
        $this->cache = new Cache();
    }
    
    public function getAllProducts($limit = 12, $offset = 0) {
        $cacheKey = "products_all_{$limit}_{$offset}";
        
        return $this->cache->remember($cacheKey, function() use ($limit, $offset) {
            return $this->product->getAllProducts($limit, $offset);
        }, 1800); // Cache for 30 minutes
    }
    
    public function getProductsByCategory($category, $limit = 12, $offset = 0) {
        $cacheKey = "products_category_{$category}_{$limit}_{$offset}";
        
        return $this->cache->remember($cacheKey, function() use ($category, $limit, $offset) {
            return $this->product->getProductsByCategory($category, $limit, $offset);
        }, 1800);
    }
    
    public function searchProducts($searchTerm, $category = null, $minPrice = null, $maxPrice = null, $limit = 12, $offset = 0) {
        // Don't cache search results as they're dynamic
        return $this->product->searchProducts($searchTerm, $category, $minPrice, $maxPrice, $limit, $offset);
    }
    
    public function getProductById($id) {
        $cacheKey = "product_{$id}";
        
        return $this->cache->remember($cacheKey, function() use ($id) {
            return $this->product->getProductById($id);
        }, 3600); // Cache for 1 hour
    }
    
    public function createProduct($data) {
        $result = $this->product->createProduct($data);
        
        if ($result) {
            // Clear related caches
            $this->clearProductCaches();
        }
        
        return $result;
    }
    
    public function updateProduct($id, $data) {
        $result = $this->product->updateProduct($id, $data);
        
        if ($result) {
            // Clear specific product cache and related caches
            $this->cache->delete("product_{$id}");
            $this->clearProductCaches();
        }
        
        return $result;
    }
    
    public function deleteProduct($id) {
        $result = $this->product->deleteProduct($id);
        
        if ($result) {
            // Clear specific product cache and related caches
            $this->cache->delete("product_{$id}");
            $this->clearProductCaches();
        }
        
        return $result;
    }
    
    private function clearProductCaches() {
        // Clear category and listing caches
        $categories = ['action', 'adventure', 'rpg', 'strategy', 'sports', 'racing'];
        
        foreach ($categories as $category) {
            // Clear multiple pages
            for ($page = 0; $page < 10; $page++) {
                $offset = $page * 12;
                $this->cache->delete("products_category_{$category}_12_{$offset}");
                $this->cache->delete("products_all_12_{$offset}");
            }
        }
    }
}
```

### 6.8 Performance Monitoring

#### Performance Logger
```php
// utils/PerformanceLogger.php
class PerformanceLogger {
    private $startTime;
    private $startMemory;
    private $queries = [];
    
    public function __construct() {
        $this->startTime = microtime(true);
        $this->startMemory = memory_get_usage();
    }
    
    public function logQuery($query, $executionTime, $params = []) {
        $this->queries[] = [
            'query' => $query,
            'execution_time' => $executionTime,
            'params' => $params,
            'timestamp' => microtime(true)
        ];
    }
    
    public function getStats() {
        $endTime = microtime(true);
        $endMemory = memory_get_usage();
        
        return [
            'execution_time' => $endTime - $this->startTime,
            'memory_usage' => $endMemory - $this->startMemory,
            'peak_memory' => memory_get_peak_usage(),
            'query_count' => count($this->queries),
            'total_query_time' => array_sum(array_column($this->queries, 'execution_time')),
            'queries' => $this->queries
        ];
    }
    
    public function logToFile($filename = 'performance.log') {
        $stats = $this->getStats();
        $logEntry = [
            'timestamp' => date('Y-m-d H:i:s'),
            'url' => $_SERVER['REQUEST_URI'] ?? 'CLI',
            'method' => $_SERVER['REQUEST_METHOD'] ?? 'CLI',
            'stats' => $stats
        ];
        
        file_put_contents($filename, json_encode($logEntry) . "\n", FILE_APPEND | LOCK_EX);
    }
    
    public function shouldAlert() {
        $stats = $this->getStats();
        
        // Alert if execution time > 2 seconds or memory usage > 50MB
        return $stats['execution_time'] > 2.0 || $stats['memory_usage'] > 50 * 1024 * 1024;
    }
}
```

#### Enhanced Database Class with Monitoring
```php
// config/database.php (Enhanced)
class Database {
    private $host = "localhost";
    private $db_name = "game_store_oop";
    private $username = "root";
    private $password = "";
    private $conn;
    private $performanceLogger;
    
    public function __construct() {
        $this->performanceLogger = new PerformanceLogger();
    }
    
    public function getConnection() {
        $this->conn = null;
        
        try {
            $dsn = "mysql:host=" . $this->host . ";dbname=" . $this->db_name . ";charset=utf8mb4";
            $options = [
                PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
                PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
                PDO::ATTR_EMULATE_PREPARES => false,
                PDO::MYSQL_ATTR_INIT_COMMAND => "SET NAMES utf8mb4"
            ];
            
            $this->conn = new PDO($dsn, $this->username, $this->password, $options);
            
            // Enable query logging in development
            if (defined('DEBUG') && DEBUG) {
                $this->conn = new MonitoredPDO($this->conn, $this->performanceLogger);
            }
            
        } catch(PDOException $exception) {
            error_log("Connection error: " . $exception->getMessage());
            throw new Exception("Database connection failed");
        }
        
        return $this->conn;
    }
    
    public function getPerformanceLogger() {
        return $this->performanceLogger;
    }
}

// Wrapper class for monitoring PDO queries
class MonitoredPDO {
    private $pdo;
    private $logger;
    
    public function __construct(PDO $pdo, PerformanceLogger $logger) {
        $this->pdo = $pdo;
        $this->logger = $logger;
    }
    
    public function prepare($statement, $driver_options = []) {
        return new MonitoredPDOStatement($this->pdo->prepare($statement, $driver_options), $this->logger, $statement);
    }
    
    public function __call($method, $args) {
        return call_user_func_array([$this->pdo, $method], $args);
    }
}

class MonitoredPDOStatement {
    private $stmt;
    private $logger;
    private $query;
    
    public function __construct(PDOStatement $stmt, PerformanceLogger $logger, $query) {
        $this->stmt = $stmt;
        $this->logger = $logger;
        $this->query = $query;
    }
    
    public function execute($input_parameters = null) {
        $startTime = microtime(true);
        $result = $this->stmt->execute($input_parameters);
        $executionTime = microtime(true) - $startTime;
        
        $this->logger->logQuery($this->query, $executionTime, $input_parameters);
        
        return $result;
    }
    
    public function __call($method, $args) {
        return call_user_func_array([$this->stmt, $method], $args);
    }
}
```

### 6.9 Error Handling dan Logging

#### Custom Error Handler
```php
// utils/ErrorHandler.php
class ErrorHandler {
    private $logFile;
    private $displayErrors;
    
    public function __construct($logFile = 'error.log', $displayErrors = false) {
        $this->logFile = $logFile;
        $this->displayErrors = $displayErrors;
        
        // Set custom error handler
        set_error_handler([$this, 'handleError']);
        set_exception_handler([$this, 'handleException']);
        register_shutdown_function([$this, 'handleFatalError']);
    }
    
    public function handleError($severity, $message, $file, $line) {
        if (!(error_reporting() & $severity)) {
            return false;
        }
        
        $errorInfo = [
            'type' => 'Error',
            'severity' => $this->getSeverityName($severity),
            'message' => $message,
            'file' => $file,
            'line' => $line,
            'timestamp' => date('Y-m-d H:i:s'),
            'url' => $_SERVER['REQUEST_URI'] ?? 'CLI',
            'user_agent' => $_SERVER['HTTP_USER_AGENT'] ?? 'CLI',
            'ip' => $_SERVER['REMOTE_ADDR'] ?? 'CLI'
        ];
        
        $this->logError($errorInfo);
        
        if ($this->displayErrors) {
            $this->displayError($errorInfo);
        }
        
        return true;
    }
    
    public function handleException($exception) {
        $errorInfo = [
            'type' => 'Exception',
            'class' => get_class($exception),
            'message' => $exception->getMessage(),
            'file' => $exception->getFile(),
            'line' => $exception->getLine(),
            'trace' => $exception->getTraceAsString(),
            'timestamp' => date('Y-m-d H:i:s'),
            'url' => $_SERVER['REQUEST_URI'] ?? 'CLI',
            'user_agent' => $_SERVER['HTTP_USER_AGENT'] ?? 'CLI',
            'ip' => $_SERVER['REMOTE_ADDR'] ?? 'CLI'
        ];
        
        $this->logError($errorInfo);
        
        if ($this->displayErrors) {
            $this->displayError($errorInfo);
        } else {
            $this->displayGenericError();
        }
    }
    
    public function handleFatalError() {
        $error = error_get_last();
        
        if ($error && in_array($error['type'], [E_ERROR, E_PARSE, E_CORE_ERROR, E_COMPILE_ERROR])) {
            $errorInfo = [
                'type' => 'Fatal Error',
                'message' => $error['message'],
                'file' => $error['file'],
                'line' => $error['line'],
                'timestamp' => date('Y-m-d H:i:s'),
                'url' => $_SERVER['REQUEST_URI'] ?? 'CLI',
                'user_agent' => $_SERVER['HTTP_USER_AGENT'] ?? 'CLI',
                'ip' => $_SERVER['REMOTE_ADDR'] ?? 'CLI'
            ];
            
            $this->logError($errorInfo);
            
            if (!$this->displayErrors) {
                $this->displayGenericError();
            }
        }
    }
    
    private function logError($errorInfo) {
        $logEntry = json_encode($errorInfo) . "\n";
        file_put_contents($this->logFile, $logEntry, FILE_APPEND | LOCK_EX);
        
        // Send email notification for critical errors
        if (in_array($errorInfo['type'], ['Fatal Error', 'Exception'])) {
            $this->sendErrorNotification($errorInfo);
        }
    }
    
    private function displayError($errorInfo) {
        echo "<div style='background: #f8d7da; color: #721c24; padding: 10px; margin: 10px; border: 1px solid #f5c6cb; border-radius: 4px;'>";
        echo "<strong>{$errorInfo['type']}:</strong> {$errorInfo['message']}<br>";
        echo "<strong>File:</strong> {$errorInfo['file']} on line {$errorInfo['line']}<br>";
        echo "<strong>Time:</strong> {$errorInfo['timestamp']}";
        if (isset($errorInfo['trace'])) {
            echo "<br><strong>Stack Trace:</strong><pre>{$errorInfo['trace']}</pre>";
        }
        echo "</div>";
    }
    
    private function displayGenericError() {
        if (!headers_sent()) {
            http_response_code(500);
        }
        
        echo "<div style='text-align: center; padding: 50px;'>";
        echo "<h1>Oops! Something went wrong</h1>";
        echo "<p>We're sorry, but something went wrong. Please try again later.</p>";
        echo "</div>";
    }
    
    private function getSeverityName($severity) {
        $severities = [
            E_ERROR => 'Error',
            E_WARNING => 'Warning',
            E_PARSE => 'Parse Error',
            E_NOTICE => 'Notice',
            E_CORE_ERROR => 'Core Error',
            E_CORE_WARNING => 'Core Warning',
            E_COMPILE_ERROR => 'Compile Error',
            E_COMPILE_WARNING => 'Compile Warning',
            E_USER_ERROR => 'User Error',
            E_USER_WARNING => 'User Warning',
            E_USER_NOTICE => 'User Notice',
            E_STRICT => 'Strict Standards',
            E_RECOVERABLE_ERROR => 'Recoverable Error',
            E_DEPRECATED => 'Deprecated',
            E_USER_DEPRECATED => 'User Deprecated'
        ];
        
        return $severities[$severity] ?? 'Unknown';
    }
    
    private function sendErrorNotification($errorInfo) {
        // Implementation for sending error notifications
        // This could be email, Slack, or other notification systems
        error_log("Critical error occurred: " . json_encode($errorInfo));
    }
}
```

## Praktikum

### Latihan 1: Security Audit
1. Review kode untuk vulnerability
2. Test SQL injection pada form input
3. Test XSS pada output data
4. Verify CSRF protection
5. Check session security

### Latihan 2: Performance Testing
1. Implement performance monitoring
2. Test dengan data besar
3. Analyze slow queries
4. Implement caching
5. Measure improvement

### Latihan 3: Security Implementation
1. Implement input validation pada semua form
2. Add CSRF protection
3. Implement rate limiting
4. Add security headers
5. Test security measures

```php
// Security headers implementation
header('X-Content-Type-Options: nosniff');
header('X-Frame-Options: DENY');
header('X-XSS-Protection: 1; mode=block');
header('Strict-Transport-Security: max-age=31536000; includeSubDomains');
header('Content-Security-Policy: default-src \'self\'; script-src \'self\' \'unsafe-inline\'; style-src \'self\' \'unsafe-inline\';');
```

### Latihan 4: Optimization Implementation
1. Add database indexes
2. Implement query optimization
3. Add caching layer
4. Optimize image loading
5. Implement lazy loading

## Tugas

1. **Security Enhancement**: Implement comprehensive security measures
2. **Performance Optimization**: Optimize database queries dan implement caching
3. **Monitoring System**: Create monitoring dashboard
4. **Error Handling**: Implement robust error handling

## Evaluasi

### Checklist Penyelesaian
- [ ] Memahami web application security
- [ ] Dapat mengimplementasikan input validation
- [ ] Memahami SQL injection prevention
- [ ] Dapat mengimplementasikan XSS protection
- [ ] Memahami session security
- [ ] Dapat mengoptimalkan database queries
- [ ] Memahami caching strategies
- [ ] Dapat mengimplementasikan performance monitoring

### Pertanyaan Review
1. Apa saja OWASP Top 10 vulnerabilities?
2. Bagaimana cara mencegah SQL injection?
3. Apa perbedaan antara validation dan sanitization?
4. Bagaimana cara mengimplementasikan CSRF protection?
5. Apa keuntungan menggunakan prepared statements?
6. Bagaimana cara mengoptimalkan database queries?
7. Apa saja jenis-jenis caching?
8. Bagaimana cara monitoring performance aplikasi?

### Final Project
Buat aplikasi e-commerce yang secure dan optimal dengan fitur:
- Comprehensive security measures
- Input validation dan sanitization
- SQL injection prevention
- XSS protection
- CSRF protection
- Session security
- Database optimization
- Caching implementation
- Performance monitoring
- Error handling dan logging
- Security audit report
- Performance benchmark report

---

**Catatan**: Keamanan dan optimasi adalah aspek yang sangat penting dalam pengembangan aplikasi web. Pastikan untuk selalu mengikuti security best practices dan melakukan testing secara berkala.