# BAB 4: SISTEM KERANJANG BELANJA DAN MANAJEMEN PESANAN

---

**📚 Modul Pembelajaran Object-Oriented Programming dengan PHP**

**Penulis:** Herisvan Hendra, M.Pd.T  
**BAB:** 4 - Sistem Keranjang dan Pesanan  
**Estimasi Waktu:** 4-5 jam  
**Level:** Intermediate

---

## Deskripsi
Bab ini membahas implementasi sistem keranjang belanja (shopping cart) dan manajemen pesanan dalam aplikasi Game Store menggunakan konsep OOP. Mahasiswa akan mempelajari cara kerja CartController, OrderController, serta model Cart, Order, dan OrderItem untuk mengelola proses pembelian dari awal hingga selesai.

## Tujuan Pembelajaran
Setelah menyelesaikan bab ini, mahasiswa diharapkan dapat:
1. Memahami implementasi shopping cart menggunakan session
2. Menganalisis CartController dan Cart model
3. Memahami proses checkout dan order management
4. Mengimplementasikan OrderController dan Order model
5. Memahami relasi antar tabel dalam sistem pesanan
6. Membuat sistem notifikasi dan status tracking

## Materi

### 4.1 Arsitektur Sistem Keranjang dan Pesanan

#### Komponen Utama
1. **CartController**: Mengelola operasi keranjang belanja
2. **Cart Model**: Mengelola data keranjang dalam session
3. **OrderController**: Mengelola proses pemesanan
4. **Order Model**: Mengelola data pesanan di database
5. **OrderItem Model**: Mengelola item-item dalam pesanan
6. **Notification System**: Sistem notifikasi real-time

#### Database Schema Pesanan
```sql
-- Tabel orders
CREATE TABLE orders (
    id INT AUTO_INCREMENT PRIMARY KEY,
    user_id INT NOT NULL,
    total_amount DECIMAL(10,2) NOT NULL,
    status ENUM('pending', 'processing', 'shipped', 'delivered', 'cancelled') DEFAULT 'pending',
    shipping_address TEXT NOT NULL,
    payment_method VARCHAR(50) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);

-- Tabel order_items
CREATE TABLE order_items (
    id INT AUTO_INCREMENT PRIMARY KEY,
    order_id INT NOT NULL,
    product_id INT NOT NULL,
    quantity INT NOT NULL,
    price DECIMAL(10,2) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (order_id) REFERENCES orders(id) ON DELETE CASCADE,
    FOREIGN KEY (product_id) REFERENCES products(id) ON DELETE CASCADE
);
```

#### Flow Sistem Keranjang dan Pesanan
```
Add to Cart → Cart Session → Checkout → Order Creation → Payment → Order Processing → Delivery
```

### 4.2 Analisis Cart Model

#### Struktur Class Cart
```php
// models/Cart.php
class Cart {
    private $items;
    
    public function __construct() {
        if (!isset($_SESSION['cart'])) {
            $_SESSION['cart'] = [];
        }
        $this->items = &$_SESSION['cart'];
    }
}
```

#### Method Utama Cart Model

1. **addItem() - Menambah Item ke Keranjang**
```php
public function addItem($productId, $quantity = 1) {
    $productId = (int)$productId;
    $quantity = (int)$quantity;
    
    if ($quantity <= 0) {
        return false;
    }
    
    if (isset($this->items[$productId])) {
        $this->items[$productId]['quantity'] += $quantity;
    } else {
        // Get product details
        $product = new Product();
        $productData = $product->getProductById($productId);
        
        if (!$productData) {
            return false;
        }
        
        $this->items[$productId] = [
            'product_id' => $productId,
            'name' => $productData['name'],
            'price' => $productData['price'],
            'image' => $productData['image'],
            'quantity' => $quantity,
            'added_at' => time()
        ];
    }
    
    return true;
}
```
**Fungsi**: Menambahkan produk ke keranjang atau menambah quantity jika sudah ada
**Fitur**:
- Validasi input
- Increment quantity untuk item yang sudah ada
- Menyimpan detail produk untuk performa
- Timestamp untuk tracking

2. **updateItem() - Update Quantity Item**
```php
public function updateItem($productId, $quantity) {
    $productId = (int)$productId;
    $quantity = (int)$quantity;
    
    if (!isset($this->items[$productId])) {
        return false;
    }
    
    if ($quantity <= 0) {
        return $this->removeItem($productId);
    }
    
    $this->items[$productId]['quantity'] = $quantity;
    return true;
}
```
**Fungsi**: Mengupdate quantity item dalam keranjang
**Fitur**:
- Auto remove jika quantity <= 0
- Validasi existence
- Simple quantity update

3. **removeItem() - Hapus Item dari Keranjang**
```php
public function removeItem($productId) {
    $productId = (int)$productId;
    
    if (isset($this->items[$productId])) {
        unset($this->items[$productId]);
        return true;
    }
    
    return false;
}
```
**Fungsi**: Menghapus item dari keranjang
**Fitur**:
- Safe removal
- Return status

4. **getItems() - Ambil Semua Item**
```php
public function getItems() {
    return $this->items;
}
```
**Fungsi**: Mengambil semua item dalam keranjang

5. **getTotalItems() - Hitung Total Item**
```php
public function getTotalItems() {
    $total = 0;
    foreach ($this->items as $item) {
        $total += $item['quantity'];
    }
    return $total;
}
```
**Fungsi**: Menghitung total quantity semua item

6. **getTotalPrice() - Hitung Total Harga**
```php
public function getTotalPrice() {
    $total = 0;
    foreach ($this->items as $item) {
        $total += $item['price'] * $item['quantity'];
    }
    return $total;
}
```
**Fungsi**: Menghitung total harga semua item

7. **clearCart() - Kosongkan Keranjang**
```php
public function clearCart() {
    $this->items = [];
    $_SESSION['cart'] = [];
    return true;
}
```
**Fungsi**: Mengosongkan seluruh keranjang

8. **isEmpty() - Cek Keranjang Kosong**
```php
public function isEmpty() {
    return empty($this->items);
}
```
**Fungsi**: Mengecek apakah keranjang kosong

9. **getItemCount() - Hitung Item Spesifik**
```php
public function getItemCount($productId) {
    $productId = (int)$productId;
    return isset($this->items[$productId]) ? $this->items[$productId]['quantity'] : 0;
}
```
**Fungsi**: Mengambil quantity item spesifik

### 4.3 Analisis CartController

#### Struktur CartController
```php
// controllers/CartController.php
class CartController {
    private $cart;
    
    public function __construct() {
        $this->cart = new Cart();
    }
}
```

#### Method Utama CartController

1. **addToCart() - dengan Validasi Stock**
```php
public function addToCart($productId, $quantity = 1) {
    // Check if product exists and has enough stock
    $product = new Product();
    $productData = $product->getProductById($productId);
    
    if (!$productData) {
        return ['success' => false, 'message' => 'Produk tidak ditemukan'];
    }
    
    if ($productData['stock'] < $quantity) {
        return ['success' => false, 'message' => 'Stok tidak mencukupi'];
    }
    
    // Check if item already exists in cart
    $currentQuantity = $this->cart->getItemCount($productId);
    $newQuantity = $currentQuantity + $quantity;
    
    if ($newQuantity > $productData['stock']) {
        return ['success' => false, 'message' => 'Jumlah melebihi stok yang tersedia'];
    }
    
    if ($this->cart->addItem($productId, $quantity)) {
        return ['success' => true, 'message' => 'Produk berhasil ditambahkan ke keranjang'];
    }
    
    return ['success' => false, 'message' => 'Gagal menambahkan produk ke keranjang'];
}
```
**Fungsi**: Menambahkan produk ke keranjang dengan validasi lengkap
**Fitur**:
- Product existence check
- Stock availability check
- Prevent over-ordering
- Comprehensive error handling

2. **updateCartItem() - Update dengan Validasi**
```php
public function updateCartItem($productId, $quantity) {
    if ($quantity <= 0) {
        return $this->removeFromCart($productId);
    }
    
    // Check stock availability
    $product = new Product();
    $productData = $product->getProductById($productId);
    
    if (!$productData) {
        return ['success' => false, 'message' => 'Produk tidak ditemukan'];
    }
    
    if ($quantity > $productData['stock']) {
        return ['success' => false, 'message' => 'Jumlah melebihi stok yang tersedia'];
    }
    
    if ($this->cart->updateItem($productId, $quantity)) {
        return ['success' => true, 'message' => 'Keranjang berhasil diupdate'];
    }
    
    return ['success' => false, 'message' => 'Gagal mengupdate keranjang'];
}
```
**Fungsi**: Mengupdate quantity item dengan validasi stock

3. **removeFromCart() - Hapus Item**
```php
public function removeFromCart($productId) {
    if ($this->cart->removeItem($productId)) {
        return ['success' => true, 'message' => 'Produk berhasil dihapus dari keranjang'];
    }
    
    return ['success' => false, 'message' => 'Gagal menghapus produk dari keranjang'];
}
```
**Fungsi**: Menghapus item dari keranjang

4. **clearCart() - Kosongkan Keranjang**
```php
public function clearCart() {
    if ($this->cart->clearCart()) {
        return ['success' => true, 'message' => 'Keranjang berhasil dikosongkan'];
    }
    
    return ['success' => false, 'message' => 'Gagal mengosongkan keranjang'];
}
```
**Fungsi**: Mengosongkan seluruh keranjang

5. **getCartSummary() - Ringkasan Keranjang**
```php
public function getCartSummary() {
    $items = $this->cart->getItems();
    $totalItems = $this->cart->getTotalItems();
    $totalPrice = $this->cart->getTotalPrice();
    
    return [
        'items' => $items,
        'total_items' => $totalItems,
        'total_price' => $totalPrice,
        'is_empty' => $this->cart->isEmpty()
    ];
}
```
**Fungsi**: Mengambil ringkasan lengkap keranjang

### 4.4 Analisis Order Model

#### Struktur Class Order
```php
// models/Order.php
class Order {
    private $conn;
    private $table_name = "orders";
    
    public $id;
    public $user_id;
    public $total_amount;
    public $status;
    public $shipping_address;
    public $payment_method;
    public $created_at;
    public $updated_at;
    
    public function __construct() {
        $database = new Database();
        $this->conn = $database->getConnection();
    }
}
```

#### Method Utama Order Model

1. **createOrder() - Buat Pesanan Baru**
```php
public function createOrder() {
    $query = "INSERT INTO " . $this->table_name . " 
             SET user_id=:user_id, total_amount=:total_amount, 
                 status=:status, shipping_address=:shipping_address, 
                 payment_method=:payment_method";
    
    $stmt = $this->conn->prepare($query);
    
    // Sanitasi input
    $this->user_id = htmlspecialchars(strip_tags($this->user_id));
    $this->total_amount = htmlspecialchars(strip_tags($this->total_amount));
    $this->status = htmlspecialchars(strip_tags($this->status));
    $this->shipping_address = htmlspecialchars(strip_tags($this->shipping_address));
    $this->payment_method = htmlspecialchars(strip_tags($this->payment_method));
    
    // Bind parameters
    $stmt->bindParam(":user_id", $this->user_id);
    $stmt->bindParam(":total_amount", $this->total_amount);
    $stmt->bindParam(":status", $this->status);
    $stmt->bindParam(":shipping_address", $this->shipping_address);
    $stmt->bindParam(":payment_method", $this->payment_method);
    
    if($stmt->execute()) {
        $this->id = $this->conn->lastInsertId();
        return true;
    }
    
    return false;
}
```
**Fungsi**: Membuat pesanan baru dan mengembalikan ID

2. **getOrdersByUserId() - Ambil Pesanan User**
```php
public function getOrdersByUserId($userId) {
    $query = "SELECT * FROM " . $this->table_name . " 
             WHERE user_id = :user_id 
             ORDER BY created_at DESC";
    
    $stmt = $this->conn->prepare($query);
    $stmt->bindParam(":user_id", $userId);
    $stmt->execute();
    
    return $stmt->fetchAll(PDO::FETCH_ASSOC);
}
```
**Fungsi**: Mengambil semua pesanan user tertentu

3. **getOrderById() - Ambil Detail Pesanan**
```php
public function getOrderById($orderId) {
    $query = "SELECT o.*, u.username, u.email 
             FROM " . $this->table_name . " o 
             JOIN users u ON o.user_id = u.id 
             WHERE o.id = :order_id LIMIT 1";
    
    $stmt = $this->conn->prepare($query);
    $stmt->bindParam(":order_id", $orderId);
    $stmt->execute();
    
    if($stmt->rowCount() > 0) {
        return $stmt->fetch(PDO::FETCH_ASSOC);
    }
    return false;
}
```
**Fungsi**: Mengambil detail pesanan dengan info user

4. **updateOrderStatus() - Update Status Pesanan**
```php
public function updateOrderStatus($orderId, $status) {
    $allowedStatuses = ['pending', 'processing', 'shipped', 'delivered', 'cancelled'];
    
    if (!in_array($status, $allowedStatuses)) {
        return false;
    }
    
    $query = "UPDATE " . $this->table_name . " 
             SET status = :status, updated_at = CURRENT_TIMESTAMP 
             WHERE id = :order_id";
    
    $stmt = $this->conn->prepare($query);
    $stmt->bindParam(":status", $status);
    $stmt->bindParam(":order_id", $orderId);
    
    return $stmt->execute();
}
```
**Fungsi**: Mengupdate status pesanan dengan validasi

5. **getAllOrders() - Ambil Semua Pesanan (Admin)**
```php
public function getAllOrders($limit = null, $offset = null) {
    $query = "SELECT o.*, u.username, u.email 
             FROM " . $this->table_name . " o 
             JOIN users u ON o.user_id = u.id 
             ORDER BY o.created_at DESC";
    
    if ($limit !== null) {
        $query .= " LIMIT :limit";
        if ($offset !== null) {
            $query .= " OFFSET :offset";
        }
    }
    
    $stmt = $this->conn->prepare($query);
    
    if ($limit !== null) {
        $stmt->bindParam(":limit", $limit, PDO::PARAM_INT);
        if ($offset !== null) {
            $stmt->bindParam(":offset", $offset, PDO::PARAM_INT);
        }
    }
    
    $stmt->execute();
    return $stmt->fetchAll(PDO::FETCH_ASSOC);
}
```
**Fungsi**: Mengambil semua pesanan dengan pagination

6. **getOrderItems() - Ambil Item Pesanan**
```php
public function getOrderItems($orderId) {
    $query = "SELECT oi.*, p.name, p.image 
             FROM order_items oi 
             JOIN products p ON oi.product_id = p.id 
             WHERE oi.order_id = :order_id";
    
    $stmt = $this->conn->prepare($query);
    $stmt->bindParam(":order_id", $orderId);
    $stmt->execute();
    
    return $stmt->fetchAll(PDO::FETCH_ASSOC);
}
```
**Fungsi**: Mengambil semua item dalam pesanan

### 4.5 Analisis OrderItem Model

#### Struktur Class OrderItem
```php
// models/OrderItem.php
class OrderItem {
    private $conn;
    private $table_name = "order_items";
    
    public $id;
    public $order_id;
    public $product_id;
    public $quantity;
    public $price;
    public $created_at;
    
    public function __construct() {
        $database = new Database();
        $this->conn = $database->getConnection();
    }
}
```

#### Method Utama OrderItem Model

1. **createOrderItem() - Buat Item Pesanan**
```php
public function createOrderItem() {
    $query = "INSERT INTO " . $this->table_name . " 
             SET order_id=:order_id, product_id=:product_id, 
                 quantity=:quantity, price=:price";
    
    $stmt = $this->conn->prepare($query);
    
    // Sanitasi input
    $this->order_id = htmlspecialchars(strip_tags($this->order_id));
    $this->product_id = htmlspecialchars(strip_tags($this->product_id));
    $this->quantity = htmlspecialchars(strip_tags($this->quantity));
    $this->price = htmlspecialchars(strip_tags($this->price));
    
    // Bind parameters
    $stmt->bindParam(":order_id", $this->order_id);
    $stmt->bindParam(":product_id", $this->product_id);
    $stmt->bindParam(":quantity", $this->quantity);
    $stmt->bindParam(":price", $this->price);
    
    return $stmt->execute();
}
```
**Fungsi**: Membuat item pesanan baru

2. **getOrderItemsByOrderId() - Ambil Item berdasarkan Order**
```php
public function getOrderItemsByOrderId($orderId) {
    $query = "SELECT oi.*, p.name as product_name, p.image as product_image 
             FROM " . $this->table_name . " oi 
             JOIN products p ON oi.product_id = p.id 
             WHERE oi.order_id = :order_id";
    
    $stmt = $this->conn->prepare($query);
    $stmt->bindParam(":order_id", $orderId);
    $stmt->execute();
    
    return $stmt->fetchAll(PDO::FETCH_ASSOC);
}
```
**Fungsi**: Mengambil semua item dalam pesanan tertentu

### 4.6 Analisis OrderController

#### Struktur OrderController
```php
// controllers/OrderController.php
class OrderController {
    private $order;
    private $orderItem;
    private $cart;
    private $product;
    
    public function __construct() {
        $this->order = new Order();
        $this->orderItem = new OrderItem();
        $this->cart = new Cart();
        $this->product = new Product();
    }
}
```

#### Method Utama OrderController

1. **createOrder() - Proses Checkout Lengkap**
```php
public function createOrder($userId, $shippingAddress, $paymentMethod) {
    // Validasi input
    if (empty($shippingAddress) || empty($paymentMethod)) {
        return ['success' => false, 'message' => 'Data pengiriman dan pembayaran harus diisi'];
    }
    
    // Cek keranjang tidak kosong
    if ($this->cart->isEmpty()) {
        return ['success' => false, 'message' => 'Keranjang belanja kosong'];
    }
    
    $cartItems = $this->cart->getItems();
    
    // Validasi stok semua item
    foreach ($cartItems as $item) {
        $productData = $this->product->getProductById($item['product_id']);
        if (!$productData || $productData['stock'] < $item['quantity']) {
            return ['success' => false, 'message' => 'Stok produk ' . $item['name'] . ' tidak mencukupi'];
        }
    }
    
    // Mulai transaction
    $this->order->conn->beginTransaction();
    
    try {
        // Buat order
        $this->order->user_id = $userId;
        $this->order->total_amount = $this->cart->getTotalPrice();
        $this->order->status = 'pending';
        $this->order->shipping_address = $shippingAddress;
        $this->order->payment_method = $paymentMethod;
        
        if (!$this->order->createOrder()) {
            throw new Exception('Gagal membuat pesanan');
        }
        
        $orderId = $this->order->id;
        
        // Buat order items dan update stok
        foreach ($cartItems as $item) {
            // Buat order item
            $this->orderItem->order_id = $orderId;
            $this->orderItem->product_id = $item['product_id'];
            $this->orderItem->quantity = $item['quantity'];
            $this->orderItem->price = $item['price'];
            
            if (!$this->orderItem->createOrderItem()) {
                throw new Exception('Gagal membuat item pesanan');
            }
            
            // Update stok produk
            $updateResult = $this->product->updateStock($item['product_id'], $item['quantity']);
            if (!$updateResult['success']) {
                throw new Exception($updateResult['message']);
            }
        }
        
        // Commit transaction
        $this->order->conn->commit();
        
        // Kosongkan keranjang
        $this->cart->clearCart();
        
        return ['success' => true, 'message' => 'Pesanan berhasil dibuat', 'order_id' => $orderId];
        
    } catch (Exception $e) {
        // Rollback transaction
        $this->order->conn->rollback();
        return ['success' => false, 'message' => $e->getMessage()];
    }
}
```
**Fungsi**: Proses checkout lengkap dengan transaction handling
**Fitur**:
- Input validation
- Stock validation
- Database transaction
- Automatic stock update
- Cart clearing
- Error handling

2. **getOrdersByUser() - Ambil Pesanan User**
```php
public function getOrdersByUser($userId) {
    return $this->order->getOrdersByUserId($userId);
}
```
**Fungsi**: Wrapper untuk mengambil pesanan user

3. **getOrderDetails() - Detail Pesanan Lengkap**
```php
public function getOrderDetails($orderId, $userId = null) {
    $orderData = $this->order->getOrderById($orderId);
    
    if (!$orderData) {
        return ['success' => false, 'message' => 'Pesanan tidak ditemukan'];
    }
    
    // Cek authorization jika bukan admin
    if ($userId !== null && $orderData['user_id'] != $userId) {
        return ['success' => false, 'message' => 'Tidak memiliki akses ke pesanan ini'];
    }
    
    $orderItems = $this->orderItem->getOrderItemsByOrderId($orderId);
    
    return [
        'success' => true,
        'order' => $orderData,
        'items' => $orderItems
    ];
}
```
**Fungsi**: Mengambil detail lengkap pesanan dengan authorization

4. **updateOrderStatus() - Update Status (Admin)**
```php
public function updateOrderStatus($orderId, $status) {
    $allowedStatuses = ['pending', 'processing', 'shipped', 'delivered', 'cancelled'];
    
    if (!in_array($status, $allowedStatuses)) {
        return ['success' => false, 'message' => 'Status tidak valid'];
    }
    
    if ($this->order->updateOrderStatus($orderId, $status)) {
        return ['success' => true, 'message' => 'Status pesanan berhasil diupdate'];
    }
    
    return ['success' => false, 'message' => 'Gagal mengupdate status pesanan'];
}
```
**Fungsi**: Update status pesanan dengan validasi

5. **cancelOrder() - Batalkan Pesanan**
```php
public function cancelOrder($orderId, $userId = null) {
    $orderData = $this->order->getOrderById($orderId);
    
    if (!$orderData) {
        return ['success' => false, 'message' => 'Pesanan tidak ditemukan'];
    }
    
    // Cek authorization
    if ($userId !== null && $orderData['user_id'] != $userId) {
        return ['success' => false, 'message' => 'Tidak memiliki akses ke pesanan ini'];
    }
    
    // Cek status - hanya pending dan processing yang bisa dibatalkan
    if (!in_array($orderData['status'], ['pending', 'processing'])) {
        return ['success' => false, 'message' => 'Pesanan tidak dapat dibatalkan'];
    }
    
    // Mulai transaction untuk restore stok
    $this->order->conn->beginTransaction();
    
    try {
        // Update status ke cancelled
        if (!$this->order->updateOrderStatus($orderId, 'cancelled')) {
            throw new Exception('Gagal mengupdate status pesanan');
        }
        
        // Restore stok produk
        $orderItems = $this->orderItem->getOrderItemsByOrderId($orderId);
        foreach ($orderItems as $item) {
            $productData = $this->product->getProductById($item['product_id']);
            if ($productData) {
                $newStock = $productData['stock'] + $item['quantity'];
                $this->product->id = $item['product_id'];
                $this->product->stock = $newStock;
                
                if (!$this->product->updateStock($item['product_id'], -$item['quantity'])) {
                    throw new Exception('Gagal mengembalikan stok produk');
                }
            }
        }
        
        $this->order->conn->commit();
        return ['success' => true, 'message' => 'Pesanan berhasil dibatalkan'];
        
    } catch (Exception $e) {
        $this->order->conn->rollback();
        return ['success' => false, 'message' => $e->getMessage()];
    }
}
```
**Fungsi**: Membatalkan pesanan dengan restore stok

### 4.7 Sistem Notifikasi Real-time

#### Implementasi Cart Badge
```php
// Dalam header.php
<?php
if (isset($_SESSION['logged_in']) && $_SESSION['logged_in']) {
    require_once __DIR__ . '/../../models/Cart.php';
    $cart = new Cart();
    $cartItemCount = $cart->getTotalItems();
}
?>

<nav class="navbar">
    <ul class="nav-links">
        <?php if (isset($_SESSION['logged_in']) && $_SESSION['logged_in']): ?>
            <li>
                <a href="cart.php" class="cart-link">
                    Keranjang
                    <?php if ($cartItemCount > 0): ?>
                        <span class="cart-badge <?php echo $cartItemCount > 0 ? 'pulse' : ''; ?>">
                            <?php echo $cartItemCount; ?>
                        </span>
                    <?php endif; ?>
                </a>
            </li>
        <?php endif; ?>
    </ul>
</nav>
```

#### Toast Notifications
```javascript
// Dalam view files
function showToast(message, type = 'success') {
    const toast = document.getElementById('toast');
    const toastMessage = document.getElementById('toast-message');
    
    toastMessage.textContent = message;
    toast.className = `toast toast-${type} show`;
    
    setTimeout(() => {
        toast.className = 'toast';
    }, 3000);
}

// AJAX untuk add to cart
function handleAddToCart(event) {
    event.preventDefault();
    
    const form = event.target;
    const formData = new FormData(form);
    const button = form.querySelector('button[type="submit"]');
    
    // Loading state
    button.disabled = true;
    button.textContent = 'Menambahkan...';
    
    fetch('cart_action.php', {
        method: 'POST',
        body: formData
    })
    .then(response => response.json())
    .then(data => {
        if (data.success) {
            showToast(data.message, 'success');
            updateCartBadge();
        } else {
            showToast(data.message, 'error');
        }
    })
    .catch(error => {
        showToast('Terjadi kesalahan', 'error');
    })
    .finally(() => {
        button.disabled = false;
        button.textContent = 'Tambah ke Keranjang';
    });
}

// Update cart badge
function updateCartBadge() {
    fetch('get_cart_count.php')
    .then(response => response.json())
    .then(data => {
        const badge = document.querySelector('.cart-badge');
        if (badge) {
            badge.textContent = data.count;
            if (data.count > 0) {
                badge.classList.add('pulse');
                setTimeout(() => badge.classList.remove('pulse'), 1000);
            }
        }
    });
}
```

### 4.8 View Implementation

#### Cart Page
```php
// views/user/cart.php
<div class="cart-container">
    <h2>Keranjang Belanja</h2>
    
    <?php if (!$cart->isEmpty()): ?>
        <div class="cart-items">
            <?php foreach ($cartItems as $item): ?>
            <div class="cart-item">
                <img src="../../assets/images/<?php echo $item['image']; ?>" alt="<?php echo $item['name']; ?>">
                <div class="item-details">
                    <h3><?php echo htmlspecialchars($item['name']); ?></h3>
                    <p class="price">Rp <?php echo number_format($item['price'], 0, ',', '.'); ?></p>
                </div>
                <div class="quantity-controls">
                    <form method="POST" action="cart_action.php" class="quantity-form">
                        <input type="hidden" name="action" value="update">
                        <input type="hidden" name="product_id" value="<?php echo $item['product_id']; ?>">
                        <button type="button" class="qty-btn" onclick="changeQuantity(this, -1)">-</button>
                        <input type="number" name="quantity" value="<?php echo $item['quantity']; ?>" min="1" class="qty-input">
                        <button type="button" class="qty-btn" onclick="changeQuantity(this, 1)">+</button>
                        <button type="submit" class="update-btn">Update</button>
                    </form>
                </div>
                <div class="item-total">
                    Rp <?php echo number_format($item['price'] * $item['quantity'], 0, ',', '.'); ?>
                </div>
                <div class="item-actions">
                    <a href="cart_action.php?action=remove&product_id=<?php echo $item['product_id']; ?>" 
                       class="remove-btn" 
                       onclick="return confirm('Hapus item ini dari keranjang?')">Hapus</a>
                </div>
            </div>
            <?php endforeach; ?>
        </div>
        
        <div class="cart-summary">
            <div class="summary-row">
                <span>Total Item:</span>
                <span><?php echo $totalItems; ?> item</span>
            </div>
            <div class="summary-row total">
                <span>Total Harga:</span>
                <span>Rp <?php echo number_format($totalPrice, 0, ',', '.'); ?></span>
            </div>
            <div class="cart-actions">
                <a href="checkout.php" class="btn btn-primary btn-large">Checkout</a>
                <a href="cart_action.php?action=clear" 
                   class="btn btn-secondary" 
                   onclick="return confirm('Kosongkan seluruh keranjang?')">Kosongkan Keranjang</a>
            </div>
        </div>
    <?php else: ?>
        <div class="empty-cart">
            <p>Keranjang belanja Anda kosong</p>
            <a href="home.php" class="btn btn-primary">Mulai Belanja</a>
        </div>
    <?php endif; ?>
</div>
```

#### Checkout Page
```php
// views/user/checkout.php
<div class="checkout-container">
    <h2>Checkout</h2>
    
    <div class="checkout-content">
        <div class="order-summary">
            <h3>Ringkasan Pesanan</h3>
            <?php foreach ($cartItems as $item): ?>
            <div class="order-item">
                <span><?php echo htmlspecialchars($item['name']); ?></span>
                <span><?php echo $item['quantity']; ?>x</span>
                <span>Rp <?php echo number_format($item['price'] * $item['quantity'], 0, ',', '.'); ?></span>
            </div>
            <?php endforeach; ?>
            <div class="order-total">
                <strong>Total: Rp <?php echo number_format($totalPrice, 0, ',', '.'); ?></strong>
            </div>
        </div>
        
        <div class="checkout-form">
            <form method="POST" action="order_action.php">
                <input type="hidden" name="action" value="create">
                
                <div class="form-group">
                    <label for="shipping_address">Alamat Pengiriman:</label>
                    <textarea name="shipping_address" id="shipping_address" required 
                              placeholder="Masukkan alamat lengkap untuk pengiriman"></textarea>
                </div>
                
                <div class="form-group">
                    <label for="payment_method">Metode Pembayaran:</label>
                    <select name="payment_method" id="payment_method" required>
                        <option value="">Pilih Metode Pembayaran</option>
                        <option value="transfer_bank">Transfer Bank</option>
                        <option value="e_wallet">E-Wallet</option>
                        <option value="cod">Cash on Delivery (COD)</option>
                    </select>
                </div>
                
                <div class="form-actions">
                    <a href="cart.php" class="btn btn-secondary">Kembali ke Keranjang</a>
                    <button type="submit" class="btn btn-primary">Buat Pesanan</button>
                </div>
            </form>
        </div>
    </div>
</div>
```

#### Order History
```php
// views/user/orders.php
<div class="orders-container">
    <h2>Riwayat Pesanan</h2>
    
    <?php if (!empty($orders)): ?>
        <div class="orders-list">
            <?php foreach ($orders as $order): ?>
            <div class="order-card">
                <div class="order-header">
                    <div class="order-id">Pesanan #<?php echo $order['id']; ?></div>
                    <div class="order-date"><?php echo date('d/m/Y H:i', strtotime($order['created_at'])); ?></div>
                </div>
                <div class="order-info">
                    <div class="order-status status-<?php echo $order['status']; ?>">
                        <?php 
                        $statusLabels = [
                            'pending' => 'Menunggu',
                            'processing' => 'Diproses',
                            'shipped' => 'Dikirim',
                            'delivered' => 'Selesai',
                            'cancelled' => 'Dibatalkan'
                        ];
                        echo $statusLabels[$order['status']] ?? $order['status'];
                        ?>
                    </div>
                    <div class="order-total">Rp <?php echo number_format($order['total_amount'], 0, ',', '.'); ?></div>
                </div>
                <div class="order-actions">
                    <a href="order_detail.php?id=<?php echo $order['id']; ?>" class="btn btn-info">Detail</a>
                    <?php if (in_array($order['status'], ['pending', 'processing'])): ?>
                    <a href="order_action.php?action=cancel&id=<?php echo $order['id']; ?>" 
                       class="btn btn-danger" 
                       onclick="return confirm('Yakin ingin membatalkan pesanan ini?')">Batalkan</a>
                    <?php endif; ?>
                </div>
            </div>
            <?php endforeach; ?>
        </div>
    <?php else: ?>
        <div class="no-orders">
            <p>Belum ada pesanan</p>
            <a href="home.php" class="btn btn-primary">Mulai Belanja</a>
        </div>
    <?php endif; ?>
</div>
```

## Praktikum

### Latihan 1: Analisis Cart System
1. Buka file `models/Cart.php`
2. Analisis penggunaan session untuk menyimpan data
3. Test semua fungsi cart (add, update, remove, clear)
4. Buat diagram flow cart operations

### Latihan 2: Testing Order Process
1. Tambahkan beberapa produk ke keranjang
2. Lakukan proses checkout lengkap
3. Cek data di database (orders dan order_items)
4. Test cancel order dan lihat perubahan stok

### Latihan 3: Implementasi Fitur Baru
1. Tambahkan field "notes" pada order
2. Implementasikan order tracking dengan timeline
3. Buat sistem email notification
4. Tambahkan fitur wishlist

```php
// Contoh order tracking
public function getOrderTimeline($orderId) {
    $query = "SELECT status, updated_at FROM order_status_history 
             WHERE order_id = :order_id ORDER BY updated_at ASC";
    $stmt = $this->conn->prepare($query);
    $stmt->bindParam(":order_id", $orderId);
    $stmt->execute();
    
    return $stmt->fetchAll(PDO::FETCH_ASSOC);
}
```

### Latihan 4: Performance Optimization
1. Implementasikan cart persistence di database
2. Buat sistem caching untuk frequent queries
3. Optimasi query dengan proper indexing
4. Implementasikan lazy loading untuk order items

## Tugas

1. **Enhancement**: Implementasikan sistem discount dan coupon
2. **Analytics**: Buat dashboard analytics untuk penjualan
3. **Notification**: Implementasikan real-time notifications
4. **Mobile**: Buat responsive design untuk mobile

## Evaluasi

### Checklist Penyelesaian
- [ ] Memahami cart system dengan session
- [ ] Dapat melakukan operasi cart (CRUD)
- [ ] Memahami proses checkout lengkap
- [ ] Dapat mengelola order dan status
- [ ] Memahami database transactions
- [ ] Dapat mengimplementasikan notifications
- [ ] Memahami order cancellation process

### Pertanyaan Review
1. Mengapa menggunakan session untuk cart?
2. Apa keuntungan database transaction?
3. Bagaimana cara handle concurrent orders?
4. Mengapa perlu validasi stok saat checkout?
5. Bagaimana cara mengoptimalkan cart performance?
6. Apa perbedaan soft delete vs hard delete untuk orders?
7. Bagaimana cara handle payment integration?

### Mini Project
Buat sistem e-commerce lengkap dengan fitur:
- Shopping cart dengan persistence
- Multi-step checkout process
- Order management dengan status tracking
- Email notifications
- Admin dashboard untuk order management
- Analytics dan reporting

---

**Catatan**: Pastikan memahami konsep transaction dan data consistency sebelum melanjutkan ke bab selanjutnya.