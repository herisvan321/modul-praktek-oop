# BAB 3: MANAJEMEN PRODUK DAN KATALOG GAME

---

**📚 Modul Pembelajaran Object-Oriented Programming dengan PHP**

**Penulis:** Herisvan Hendra, M.Pd.T  
**BAB:** 3 - Manajemen Produk  
**Estimasi Waktu:** 4-5 jam  
**Level:** Intermediate

---

## Deskripsi
Bab ini membahas implementasi sistem manajemen produk dalam aplikasi Game Store menggunakan konsep OOP. Mahasiswa akan mempelajari cara kerja ProductController dan Product model, serta implementasi CRUD (Create, Read, Update, Delete) operations untuk mengelola katalog game.

## Tujuan Pembelajaran
Setelah menyelesaikan bab ini, mahasiswa diharapkan dapat:
1. Memahami implementasi CRUD operations menggunakan OOP
2. Menganalisis ProductController dan Product model
3. Memahami konsep file upload dan image handling
4. Mengimplementasikan search dan filtering functionality
5. Membuat sistem kategori produk
6. Memahami stock management

## Materi

### 3.1 Arsitektur Manajemen Produk

#### Komponen Utama
1. **ProductController**: Mengelola logika bisnis produk
2. **Product Model**: Mengelola data produk dan database operations
3. **Admin Views**: Interface untuk mengelola produk (admin)
4. **User Views**: Interface untuk melihat katalog (user)
5. **Image Management**: Sistem upload dan penyimpanan gambar

#### Database Schema Produk
```sql
CREATE TABLE products (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    description TEXT,
    price DECIMAL(10,2) NOT NULL,
    category VARCHAR(100),
    image VARCHAR(255),
    stock INT DEFAULT 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

#### Flow Manajemen Produk
```
Admin Input → ProductController → Product Model → Database → Response
User Request → ProductController → Product Model → Database → View
```

### 3.2 Analisis Product Model

#### Struktur Class Product
```php
// models/Product.php
class Product {
    private $conn;
    private $table_name = "products";
    
    public $id;
    public $name;
    public $description;
    public $price;
    public $category;
    public $image;
    public $stock;
    public $created_at;
    
    public function __construct() {
        $database = new Database();
        $this->conn = $database->getConnection();
    }
}
```

#### Method Utama Product Model

1. **getAllProducts() - Read All**
```php
public function getAllProducts() {
    $query = "SELECT * FROM " . $this->table_name . " WHERE stock > 0 ORDER BY created_at DESC";
    $stmt = $this->conn->prepare($query);
    $stmt->execute();
    
    return $stmt->fetchAll(PDO::FETCH_ASSOC);
}
```
**Fungsi**: Mengambil semua produk yang masih memiliki stok
**Fitur**: 
- Filter produk dengan stok > 0
- Sorting berdasarkan tanggal terbaru
- Return array associative

2. **getProductById() - Read Single**
```php
public function getProductById($id) {
    $query = "SELECT * FROM " . $this->table_name . " WHERE id = :id LIMIT 1";
    $stmt = $this->conn->prepare($query);
    $stmt->bindParam(":id", $id);
    $stmt->execute();
    
    if($stmt->rowCount() > 0) {
        return $stmt->fetch(PDO::FETCH_ASSOC);
    }
    return false;
}
```
**Fungsi**: Mengambil detail produk berdasarkan ID
**Fitur**:
- Parameter binding untuk security
- Return false jika tidak ditemukan
- Single record retrieval

3. **createProduct() - Create**
```php
public function createProduct() {
    $query = "INSERT INTO " . $this->table_name . " 
             SET name=:name, description=:description, price=:price, 
                 category=:category, image=:image, stock=:stock";
    
    $stmt = $this->conn->prepare($query);
    
    // Sanitasi input
    $this->name = htmlspecialchars(strip_tags($this->name));
    $this->description = htmlspecialchars(strip_tags($this->description));
    $this->price = htmlspecialchars(strip_tags($this->price));
    $this->category = htmlspecialchars(strip_tags($this->category));
    $this->image = htmlspecialchars(strip_tags($this->image));
    $this->stock = htmlspecialchars(strip_tags($this->stock));
    
    // Bind parameters
    $stmt->bindParam(":name", $this->name);
    $stmt->bindParam(":description", $this->description);
    $stmt->bindParam(":price", $this->price);
    $stmt->bindParam(":category", $this->category);
    $stmt->bindParam(":image", $this->image);
    $stmt->bindParam(":stock", $this->stock);
    
    return $stmt->execute();
}
```
**Fungsi**: Menambahkan produk baru ke database
**Fitur**:
- Input sanitization untuk security
- Parameter binding
- Validation data types

4. **updateProduct() - Update**
```php
public function updateProduct() {
    $query = "UPDATE " . $this->table_name . " 
             SET name=:name, description=:description, price=:price, 
                 category=:category, image=:image, stock=:stock 
             WHERE id=:id";
    
    $stmt = $this->conn->prepare($query);
    
    // Sanitasi input
    $this->name = htmlspecialchars(strip_tags($this->name));
    $this->description = htmlspecialchars(strip_tags($this->description));
    $this->price = htmlspecialchars(strip_tags($this->price));
    $this->category = htmlspecialchars(strip_tags($this->category));
    $this->image = htmlspecialchars(strip_tags($this->image));
    $this->stock = htmlspecialchars(strip_tags($this->stock));
    $this->id = htmlspecialchars(strip_tags($this->id));
    
    // Bind parameters
    $stmt->bindParam(":name", $this->name);
    $stmt->bindParam(":description", $this->description);
    $stmt->bindParam(":price", $this->price);
    $stmt->bindParam(":category", $this->category);
    $stmt->bindParam(":image", $this->image);
    $stmt->bindParam(":stock", $this->stock);
    $stmt->bindParam(":id", $this->id);
    
    return $stmt->execute();
}
```
**Fungsi**: Mengupdate data produk yang sudah ada
**Fitur**:
- Update berdasarkan ID
- Sanitization semua input
- Preserve data integrity

5. **deleteProduct() - Delete**
```php
public function deleteProduct($id) {
    $query = "DELETE FROM " . $this->table_name . " WHERE id = :id";
    $stmt = $this->conn->prepare($query);
    $stmt->bindParam(":id", $id);
    
    return $stmt->execute();
}
```
**Fungsi**: Menghapus produk dari database
**Fitur**:
- Hard delete (permanent)
- Parameter binding
- Simple and secure

6. **searchProducts() - Search Functionality**
```php
public function searchProducts($keyword) {
    $query = "SELECT * FROM " . $this->table_name . " 
             WHERE (name LIKE :keyword OR description LIKE :keyword OR category LIKE :keyword) 
             AND stock > 0 
             ORDER BY created_at DESC";
    
    $stmt = $this->conn->prepare($query);
    $searchTerm = "%" . $keyword . "%";
    $stmt->bindParam(":keyword", $searchTerm);
    $stmt->execute();
    
    return $stmt->fetchAll(PDO::FETCH_ASSOC);
}
```
**Fungsi**: Mencari produk berdasarkan keyword
**Fitur**:
- Search di multiple fields (name, description, category)
- Wildcard search dengan LIKE
- Filter stok > 0

### 3.3 Analisis ProductController

#### Struktur ProductController
```php
// controllers/ProductController.php
class ProductController {
    private $product;
    
    public function __construct() {
        $this->product = new Product();
    }
}
```

#### Method Utama ProductController

1. **getAllProducts()**
```php
public function getAllProducts() {
    return $this->product->getAllProducts();
}
```
**Fungsi**: Wrapper method untuk mengambil semua produk

2. **getProductById()**
```php
public function getProductById($id) {
    return $this->product->getProductById($id);
}
```
**Fungsi**: Wrapper method untuk mengambil produk by ID

3. **createProduct() - dengan Validasi**
```php
public function createProduct($name, $description, $price, $category, $image, $stock) {
    // Validasi input
    if(empty($name) || empty($price) || empty($stock)) {
        return ['success' => false, 'message' => 'Field wajib tidak boleh kosong'];
    }
    
    if(!is_numeric($price) || $price <= 0) {
        return ['success' => false, 'message' => 'Harga harus berupa angka positif'];
    }
    
    if(!is_numeric($stock) || $stock < 0) {
        return ['success' => false, 'message' => 'Stok harus berupa angka non-negatif'];
    }
    
    // Set properties
    $this->product->name = $name;
    $this->product->description = $description;
    $this->product->price = $price;
    $this->product->category = $category;
    $this->product->image = $image;
    $this->product->stock = $stock;
    
    if($this->product->createProduct()) {
        return ['success' => true, 'message' => 'Produk berhasil ditambahkan'];
    }
    
    return ['success' => false, 'message' => 'Gagal menambahkan produk'];
}
```
**Fungsi**: Mengelola pembuatan produk dengan validasi
**Fitur**:
- Input validation
- Business logic validation
- Error handling
- Success/failure response

4. **updateProduct() - dengan Validasi**
```php
public function updateProduct($id, $name, $description, $price, $category, $image, $stock) {
    // Cek apakah produk exists
    if(!$this->product->getProductById($id)) {
        return ['success' => false, 'message' => 'Produk tidak ditemukan'];
    }
    
    // Validasi input (sama seperti create)
    if(empty($name) || empty($price) || empty($stock)) {
        return ['success' => false, 'message' => 'Field wajib tidak boleh kosong'];
    }
    
    if(!is_numeric($price) || $price <= 0) {
        return ['success' => false, 'message' => 'Harga harus berupa angka positif'];
    }
    
    if(!is_numeric($stock) || $stock < 0) {
        return ['success' => false, 'message' => 'Stok harus berupa angka non-negatif'];
    }
    
    // Set properties
    $this->product->id = $id;
    $this->product->name = $name;
    $this->product->description = $description;
    $this->product->price = $price;
    $this->product->category = $category;
    $this->product->image = $image;
    $this->product->stock = $stock;
    
    if($this->product->updateProduct()) {
        return ['success' => true, 'message' => 'Produk berhasil diupdate'];
    }
    
    return ['success' => false, 'message' => 'Gagal mengupdate produk'];
}
```
**Fungsi**: Mengelola update produk dengan validasi
**Fitur**:
- Existence check
- Input validation
- Business logic validation
- Error handling

5. **deleteProduct()**
```php
public function deleteProduct($id) {
    // Cek apakah produk exists
    if(!$this->product->getProductById($id)) {
        return ['success' => false, 'message' => 'Produk tidak ditemukan'];
    }
    
    if($this->product->deleteProduct($id)) {
        return ['success' => true, 'message' => 'Produk berhasil dihapus'];
    }
    
    return ['success' => false, 'message' => 'Gagal menghapus produk'];
}
```
**Fungsi**: Mengelola penghapusan produk
**Fitur**:
- Existence check
- Safe deletion
- Error handling

6. **searchProducts()**
```php
public function searchProducts($keyword) {
    if(empty($keyword)) {
        return $this->getAllProducts();
    }
    
    return $this->product->searchProducts($keyword);
}
```
**Fungsi**: Mengelola pencarian produk
**Fitur**:
- Empty keyword handling
- Fallback to all products

### 3.4 Image Management System

#### Upload Image Functionality
```php
// Dalam product_action.php
function uploadImage($file) {
    $targetDir = "../../assets/images/";
    $allowedTypes = ['jpg', 'jpeg', 'png', 'gif', 'svg'];
    $maxSize = 5 * 1024 * 1024; // 5MB
    
    // Validasi file
    if($file['error'] !== UPLOAD_ERR_OK) {
        return ['success' => false, 'message' => 'Error uploading file'];
    }
    
    // Validasi size
    if($file['size'] > $maxSize) {
        return ['success' => false, 'message' => 'File terlalu besar (max 5MB)'];
    }
    
    // Validasi type
    $fileExtension = strtolower(pathinfo($file['name'], PATHINFO_EXTENSION));
    if(!in_array($fileExtension, $allowedTypes)) {
        return ['success' => false, 'message' => 'Tipe file tidak diizinkan'];
    }
    
    // Generate unique filename
    $fileName = uniqid() . '.' . $fileExtension;
    $targetPath = $targetDir . $fileName;
    
    // Upload file
    if(move_uploaded_file($file['tmp_name'], $targetPath)) {
        return ['success' => true, 'filename' => $fileName];
    }
    
    return ['success' => false, 'message' => 'Gagal mengupload file'];
}
```

#### Image Display
```php
// Dalam view files
<img src="../../assets/images/<?php echo htmlspecialchars($product['image'] ?? 'default.svg'); ?>" 
     alt="<?php echo htmlspecialchars($product['name']); ?>" 
     class="product-image">
```

### 3.5 Category Management

#### Implementasi Kategori
```php
// Method untuk mendapatkan kategori unik
public function getCategories() {
    $query = "SELECT DISTINCT category FROM " . $this->table_name . " 
             WHERE category IS NOT NULL AND category != '' 
             ORDER BY category ASC";
    $stmt = $this->conn->prepare($query);
    $stmt->execute();
    
    return $stmt->fetchAll(PDO::FETCH_COLUMN);
}

// Method untuk filter berdasarkan kategori
public function getProductsByCategory($category) {
    $query = "SELECT * FROM " . $this->table_name . " 
             WHERE category = :category AND stock > 0 
             ORDER BY created_at DESC";
    $stmt = $this->conn->prepare($query);
    $stmt->bindParam(":category", $category);
    $stmt->execute();
    
    return $stmt->fetchAll(PDO::FETCH_ASSOC);
}
```

### 3.6 Stock Management

#### Update Stock saat Pembelian
```php
public function updateStock($productId, $quantity) {
    // Cek stok saat ini
    $product = $this->getProductById($productId);
    if(!$product) {
        return ['success' => false, 'message' => 'Produk tidak ditemukan'];
    }
    
    $newStock = $product['stock'] - $quantity;
    if($newStock < 0) {
        return ['success' => false, 'message' => 'Stok tidak mencukupi'];
    }
    
    // Update stok
    $query = "UPDATE " . $this->table_name . " SET stock = :stock WHERE id = :id";
    $stmt = $this->conn->prepare($query);
    $stmt->bindParam(":stock", $newStock);
    $stmt->bindParam(":id", $productId);
    
    if($stmt->execute()) {
        return ['success' => true, 'new_stock' => $newStock];
    }
    
    return ['success' => false, 'message' => 'Gagal mengupdate stok'];
}

// Method untuk cek ketersediaan stok
public function checkStock($productId, $quantity) {
    $product = $this->getProductById($productId);
    if(!$product) {
        return false;
    }
    
    return $product['stock'] >= $quantity;
}
```

### 3.7 View Implementation

#### Admin Product Management
```php
// views/admin/kelola_produk.php
<div class="product-management">
    <h2>Kelola Produk</h2>
    
    <!-- Add Product Button -->
    <a href="tambah_produk.php" class="btn btn-primary">Tambah Produk</a>
    
    <!-- Product List -->
    <div class="product-list">
        <?php foreach($products as $product): ?>
        <div class="product-item">
            <img src="../../assets/images/<?php echo $product['image']; ?>" alt="<?php echo $product['name']; ?>">
            <div class="product-info">
                <h3><?php echo htmlspecialchars($product['name']); ?></h3>
                <p>Kategori: <?php echo htmlspecialchars($product['category'] ?? 'Tidak ada'); ?></p>
                <p>Harga: Rp <?php echo number_format($product['price'], 0, ',', '.'); ?></p>
                <p>Stok: <?php echo $product['stock']; ?></p>
            </div>
            <div class="product-actions">
                <a href="edit_produk.php?id=<?php echo $product['id']; ?>" class="btn btn-warning">Edit</a>
                <a href="product_action.php?action=delete&id=<?php echo $product['id']; ?>" 
                   class="btn btn-danger" 
                   onclick="return confirm('Yakin ingin menghapus produk ini?')">Hapus</a>
            </div>
        </div>
        <?php endforeach; ?>
    </div>
</div>
```

#### User Product Catalog
```php
// views/user/home.php
<div class="product-catalog">
    <!-- Search Form -->
    <form method="GET" class="search-form">
        <input type="text" name="search" placeholder="Cari game..." value="<?php echo htmlspecialchars($searchKeyword); ?>">
        <button type="submit">Cari</button>
    </form>
    
    <!-- Category Filter -->
    <div class="category-filter">
        <a href="home.php" class="<?php echo empty($_GET['category']) ? 'active' : ''; ?>">Semua</a>
        <?php foreach($categories as $category): ?>
        <a href="home.php?category=<?php echo urlencode($category); ?>" 
           class="<?php echo ($_GET['category'] ?? '') === $category ? 'active' : ''; ?>">
            <?php echo htmlspecialchars($category); ?>
        </a>
        <?php endforeach; ?>
    </div>
    
    <!-- Product Grid -->
    <div class="product-grid">
        <?php foreach($products as $product): ?>
        <div class="product-card">
            <div class="product-image">
                <img src="../../assets/images/<?php echo $product['image']; ?>" alt="<?php echo $product['name']; ?>">
                <?php if($product['stock'] <= 0): ?>
                <div class="out-of-stock">Stok Habis</div>
                <?php endif; ?>
            </div>
            <div class="product-info">
                <h3><?php echo htmlspecialchars($product['name']); ?></h3>
                <p class="category"><?php echo htmlspecialchars($product['category'] ?? 'Game'); ?></p>
                <p class="price">Rp <?php echo number_format($product['price'], 0, ',', '.'); ?></p>
                <p class="stock">Stok: <?php echo $product['stock']; ?></p>
            </div>
            <div class="product-actions">
                <a href="detail_produk.php?id=<?php echo $product['id']; ?>" class="btn btn-info">Detail</a>
                <?php if($product['stock'] > 0): ?>
                <form method="POST" action="cart_action.php" class="add-to-cart-form">
                    <input type="hidden" name="product_id" value="<?php echo $product['id']; ?>">
                    <input type="hidden" name="action" value="add">
                    <button type="submit" class="btn btn-primary">Tambah ke Keranjang</button>
                </form>
                <?php else: ?>
                <button class="btn btn-secondary" disabled>Stok Habis</button>
                <?php endif; ?>
            </div>
        </div>
        <?php endforeach; ?>
    </div>
</div>
```

## Praktikum

### Latihan 1: Analisis CRUD Operations
1. Buka file `models/Product.php`
2. Identifikasi semua method CRUD
3. Analisis query SQL yang digunakan
4. Buat diagram flow untuk setiap operation

### Latihan 2: Testing Product Management
1. Login sebagai admin
2. Tambah produk baru dengan gambar
3. Edit produk yang sudah ada
4. Test search functionality
5. Test category filtering
6. Hapus produk

### Latihan 3: Implementasi Fitur Baru
1. Tambahkan field "rating" pada tabel products
2. Update model dan controller untuk handle rating
3. Implementasikan sorting berdasarkan rating
4. Buat fitur "featured products"

```sql
-- Tambah kolom rating
ALTER TABLE products ADD COLUMN rating DECIMAL(2,1) DEFAULT 0.0;
ALTER TABLE products ADD COLUMN is_featured BOOLEAN DEFAULT FALSE;
```

```php
// Method untuk featured products
public function getFeaturedProducts($limit = 6) {
    $query = "SELECT * FROM " . $this->table_name . " 
             WHERE is_featured = 1 AND stock > 0 
             ORDER BY rating DESC, created_at DESC 
             LIMIT :limit";
    $stmt = $this->conn->prepare($query);
    $stmt->bindParam(":limit", $limit, PDO::PARAM_INT);
    $stmt->execute();
    
    return $stmt->fetchAll(PDO::FETCH_ASSOC);
}
```

### Latihan 4: Advanced Search
1. Implementasikan price range filter
2. Tambahkan sorting options (price, name, date)
3. Buat pagination untuk hasil yang banyak

```php
// Advanced search dengan filter
public function advancedSearch($keyword = '', $category = '', $minPrice = 0, $maxPrice = 999999, $sortBy = 'created_at', $sortOrder = 'DESC') {
    $query = "SELECT * FROM " . $this->table_name . " WHERE stock > 0";
    $params = [];
    
    if(!empty($keyword)) {
        $query .= " AND (name LIKE :keyword OR description LIKE :keyword)";
        $params[':keyword'] = "%" . $keyword . "%";
    }
    
    if(!empty($category)) {
        $query .= " AND category = :category";
        $params[':category'] = $category;
    }
    
    $query .= " AND price BETWEEN :minPrice AND :maxPrice";
    $params[':minPrice'] = $minPrice;
    $params[':maxPrice'] = $maxPrice;
    
    $allowedSorts = ['name', 'price', 'created_at', 'rating'];
    $allowedOrders = ['ASC', 'DESC'];
    
    if(in_array($sortBy, $allowedSorts) && in_array($sortOrder, $allowedOrders)) {
        $query .= " ORDER BY " . $sortBy . " " . $sortOrder;
    }
    
    $stmt = $this->conn->prepare($query);
    foreach($params as $key => $value) {
        $stmt->bindValue($key, $value);
    }
    $stmt->execute();
    
    return $stmt->fetchAll(PDO::FETCH_ASSOC);
}
```

## Tugas

1. **Enhancement**: Implementasikan sistem review dan rating produk
2. **Optimization**: Buat sistem caching untuk produk populer
3. **Analytics**: Tambahkan tracking views produk
4. **Inventory**: Buat sistem notifikasi stok rendah

## Evaluasi

### Checklist Penyelesaian
- [ ] Memahami CRUD operations
- [ ] Dapat mengelola produk sebagai admin
- [ ] Memahami image upload system
- [ ] Dapat menggunakan search dan filter
- [ ] Memahami stock management
- [ ] Dapat mengimplementasikan fitur baru
- [ ] Memahami security practices

### Pertanyaan Review
1. Apa perbedaan antara hard delete dan soft delete?
2. Mengapa perlu validasi input di controller?
3. Bagaimana cara mengamankan file upload?
4. Apa kegunaan prepared statements?
5. Bagaimana cara mengoptimalkan query search?
6. Mengapa perlu sanitasi output di view?
7. Bagaimana cara handle concurrent stock updates?

### Mini Project
Buat sistem manajemen produk dengan fitur:
- CRUD lengkap dengan validasi
- Upload multiple images
- Advanced search dan filter
- Stock management dengan notifikasi
- Product analytics dashboard
- Bulk operations (import/export)

---

**Catatan**: Pastikan memahami konsep CRUD dan best practices dalam manajemen data sebelum melanjutkan ke bab selanjutnya.