# BAB 5: ADMIN DASHBOARD DAN MANAJEMEN SISTEM

---

**📚 Modul Pembelajaran Object-Oriented Programming dengan PHP**

**Penulis:** Herisvan Hendra, M.Pd.T  
**BAB:** 5 - Admin Dashboard  
**Estimasi Waktu:** 3-4 jam  
**Level:** Intermediate to Advanced

---

## Deskripsi
Bab ini membahas implementasi dashboard admin dan manajemen sistem dalam aplikasi Game Store menggunakan konsep OOP. Mahasiswa akan mempelajari cara kerja AdminController, implementasi role-based access control, dan fitur-fitur administratif untuk mengelola seluruh aspek aplikasi.

## Tujuan Pembelajaran
Setelah menyelesaikan bab ini, mahasiswa diharapkan dapat:
1. Memahami implementasi role-based access control
2. Menganalisis AdminController dan fungsinya
3. Memahami dashboard analytics dan reporting
4. Mengimplementasikan manajemen user, produk, dan pesanan
5. Memahami konsep data visualization
6. Membuat sistem monitoring dan notifikasi admin

## Materi

### 5.1 Arsitektur Admin Dashboard

#### Komponen Utama
1. **AdminController**: Mengelola operasi administratif
2. **Role-Based Access Control**: Sistem pembatasan akses
3. **Dashboard Analytics**: Visualisasi data dan statistik
4. **User Management**: Pengelolaan data pengguna
5. **Product Management**: Pengelolaan katalog produk
6. **Order Management**: Pengelolaan pesanan
7. **System Settings**: Pengaturan aplikasi

#### Flow Admin System
```
Admin Login → Authentication → Authorization → Admin Dashboard → Management Modules
```

### 5.2 Analisis AdminController

#### Struktur Class AdminController
```php
// controllers/AdminController.php
class AdminController {
    private $user;
    private $product;
    private $order;
    
    public function __construct() {
        $this->user = new User();
        $this->product = new Product();
        $this->order = new Order();
    }
}
```

#### Method Utama AdminController

1. **getDashboardStats() - Statistik Dashboard**
```php
public function getDashboardStats() {
    $stats = [
        'total_users' => $this->user->getTotalUsers(),
        'total_products' => $this->product->getTotalProducts(),
        'total_orders' => $this->order->getTotalOrders(),
        'pending_orders' => $this->order->getOrderCountByStatus('pending'),
        'revenue' => $this->order->getTotalRevenue(),
        'low_stock_products' => $this->product->getLowStockProducts(5),
        'recent_orders' => $this->order->getRecentOrders(5),
        'top_selling_products' => $this->getTopSellingProducts(5),
        'sales_by_date' => $this->getSalesByDate(7) // Last 7 days
    ];
    
    return $stats;
}
```
**Fungsi**: Mengambil semua statistik untuk dashboard admin
**Fitur**:
- Aggregated data dari multiple models
- Key performance indicators
- Recent activity tracking
- Low stock alerts

2. **getAllUsers() - Manajemen User**
```php
public function getAllUsers($limit = null, $offset = null) {
    return $this->user->getAllUsers($limit, $offset);
}

public function getUserById($id) {
    return $this->user->getUserById($id);
}

public function updateUserRole($userId, $role) {
    $allowedRoles = ['user', 'admin'];
    
    if (!in_array($role, $allowedRoles)) {
        return ['success' => false, 'message' => 'Role tidak valid'];
    }
    
    if ($this->user->updateUserRole($userId, $role)) {
        return ['success' => true, 'message' => 'Role user berhasil diupdate'];
    }
    
    return ['success' => false, 'message' => 'Gagal mengupdate role user'];
}

public function deleteUser($userId) {
    // Prevent self-deletion
    if ($userId == $_SESSION['user_id']) {
        return ['success' => false, 'message' => 'Tidak dapat menghapus akun sendiri'];
    }
    
    if ($this->user->deleteUser($userId)) {
        return ['success' => true, 'message' => 'User berhasil dihapus'];
    }
    
    return ['success' => false, 'message' => 'Gagal menghapus user'];
}
```
**Fungsi**: Mengelola data pengguna
**Fitur**:
- User listing dengan pagination
- Role management
- Safe deletion dengan validasi

3. **getAllOrders() - Manajemen Pesanan**
```php
public function getAllOrders($limit = null, $offset = null) {
    return $this->order->getAllOrders($limit, $offset);
}

public function getOrderById($id) {
    return $this->order->getOrderById($id);
}

public function getOrderItems($orderId) {
    return $this->order->getOrderItems($orderId);
}

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

public function cancelOrder($orderId) {
    $orderController = new OrderController();
    return $orderController->cancelOrder($orderId);
}
```
**Fungsi**: Mengelola pesanan dari admin panel
**Fitur**:
- Order listing dengan pagination
- Status management
- Order details retrieval
- Order cancellation

4. **getOrderStatistics() - Statistik Pesanan**
```php
public function getOrderStatistics($period = 'monthly') {
    $stats = [];
    
    switch ($period) {
        case 'daily':
            $stats = $this->order->getDailyOrderStats(30); // Last 30 days
            break;
        case 'weekly':
            $stats = $this->order->getWeeklyOrderStats(12); // Last 12 weeks
            break;
        case 'monthly':
            $stats = $this->order->getMonthlyOrderStats(12); // Last 12 months
            break;
        case 'yearly':
            $stats = $this->order->getYearlyOrderStats(5); // Last 5 years
            break;
    }
    
    return $stats;
}
```
**Fungsi**: Mengambil statistik pesanan berdasarkan periode
**Fitur**:
- Flexible time period
- Aggregated statistics
- Time-series data

5. **getTopSellingProducts() - Produk Terlaris**
```php
public function getTopSellingProducts($limit = 5) {
    $query = "SELECT p.id, p.name, p.image, p.price, p.category, 
                    SUM(oi.quantity) as total_sold, 
                    SUM(oi.quantity * oi.price) as total_revenue 
             FROM products p 
             JOIN order_items oi ON p.id = oi.product_id 
             JOIN orders o ON oi.order_id = o.id 
             WHERE o.status != 'cancelled' 
             GROUP BY p.id 
             ORDER BY total_sold DESC 
             LIMIT :limit";
    
    $stmt = $this->product->conn->prepare($query);
    $stmt->bindParam(':limit', $limit, PDO::PARAM_INT);
    $stmt->execute();
    
    return $stmt->fetchAll(PDO::FETCH_ASSOC);
}
```
**Fungsi**: Mengambil produk terlaris berdasarkan quantity
**Fitur**:
- Complex join query
- Aggregated sales data
- Revenue calculation

6. **getSalesByDate() - Penjualan per Tanggal**
```php
public function getSalesByDate($days = 7) {
    $query = "SELECT DATE(o.created_at) as date, 
                    COUNT(o.id) as order_count, 
                    SUM(o.total_amount) as total_sales 
             FROM orders o 
             WHERE o.status != 'cancelled' 
               AND o.created_at >= DATE_SUB(CURRENT_DATE(), INTERVAL :days DAY) 
             GROUP BY DATE(o.created_at) 
             ORDER BY date ASC";
    
    $stmt = $this->order->conn->prepare($query);
    $stmt->bindParam(':days', $days, PDO::PARAM_INT);
    $stmt->execute();
    
    return $stmt->fetchAll(PDO::FETCH_ASSOC);
}
```
**Fungsi**: Mengambil data penjualan per tanggal
**Fitur**:
- Date-based aggregation
- Time-series data
- Sales trend analysis

### 5.3 Role-Based Access Control

#### Implementasi RBAC
```php
// In AuthController.php
public function isAdmin() {
    return isset($_SESSION['role']) && $_SESSION['role'] === 'admin';
}

public function requireAdmin() {
    $this->requireLogin();
    if (!$this->isAdmin()) {
        header('Location: ../user/home.php');
        exit();
    }
}

// In admin view files
<?php
session_start();
require_once '../../controllers/AuthController.php';

$auth = new AuthController();
$auth->requireAdmin();
?>
```

#### User Role Management
```php
// In User.php model
public function updateUserRole($userId, $role) {
    $allowedRoles = ['user', 'admin'];
    
    if (!in_array($role, $allowedRoles)) {
        return false;
    }
    
    $query = "UPDATE " . $this->table_name . " SET role = :role WHERE id = :id";
    $stmt = $this->conn->prepare($query);
    $stmt->bindParam(':role', $role);
    $stmt->bindParam(':id', $userId);
    
    return $stmt->execute();
}
```

### 5.4 Dashboard Analytics

#### Key Performance Indicators (KPIs)
```php
// In dashboard.php
<div class="dashboard-stats">
    <div class="stat-card">
        <div class="stat-icon"><i class="fas fa-users"></i></div>
        <div class="stat-content">
            <h3>Total Users</h3>
            <p class="stat-value"><?php echo $stats['total_users']; ?></p>
        </div>
    </div>
    
    <div class="stat-card">
        <div class="stat-icon"><i class="fas fa-gamepad"></i></div>
        <div class="stat-content">
            <h3>Total Products</h3>
            <p class="stat-value"><?php echo $stats['total_products']; ?></p>
        </div>
    </div>
    
    <div class="stat-card">
        <div class="stat-icon"><i class="fas fa-shopping-cart"></i></div>
        <div class="stat-content">
            <h3>Total Orders</h3>
            <p class="stat-value"><?php echo $stats['total_orders']; ?></p>
        </div>
    </div>
    
    <div class="stat-card">
        <div class="stat-icon"><i class="fas fa-money-bill-wave"></i></div>
        <div class="stat-content">
            <h3>Total Revenue</h3>
            <p class="stat-value">Rp <?php echo number_format($stats['revenue'], 0, ',', '.'); ?></p>
        </div>
    </div>
</div>
```

#### Sales Chart Implementation
```php
// In dashboard.php
<div class="chart-container">
    <h3>Sales Trend (Last 7 Days)</h3>
    <canvas id="salesChart"></canvas>
</div>

<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<script>
    // Sales chart
    const salesData = <?php echo json_encode($stats['sales_by_date']); ?>;
    const dates = salesData.map(item => item.date);
    const sales = salesData.map(item => item.total_sales);
    
    const ctx = document.getElementById('salesChart').getContext('2d');
    new Chart(ctx, {
        type: 'line',
        data: {
            labels: dates,
            datasets: [{
                label: 'Daily Sales (Rp)',
                data: sales,
                backgroundColor: 'rgba(54, 162, 235, 0.2)',
                borderColor: 'rgba(54, 162, 235, 1)',
                borderWidth: 2,
                tension: 0.1
            }]
        },
        options: {
            responsive: true,
            scales: {
                y: {
                    beginAtZero: true,
                    ticks: {
                        callback: function(value) {
                            return 'Rp ' + value.toLocaleString('id-ID');
                        }
                    }
                }
            }
        }
    });
</script>
```

#### Top Products Chart
```php
// In dashboard.php
<div class="chart-container">
    <h3>Top Selling Products</h3>
    <canvas id="productsChart"></canvas>
</div>

<script>
    // Top products chart
    const productData = <?php echo json_encode($stats['top_selling_products']); ?>;
    const productNames = productData.map(item => item.name);
    const productSales = productData.map(item => item.total_sold);
    
    const productCtx = document.getElementById('productsChart').getContext('2d');
    new Chart(productCtx, {
        type: 'bar',
        data: {
            labels: productNames,
            datasets: [{
                label: 'Units Sold',
                data: productSales,
                backgroundColor: [
                    'rgba(255, 99, 132, 0.5)',
                    'rgba(54, 162, 235, 0.5)',
                    'rgba(255, 206, 86, 0.5)',
                    'rgba(75, 192, 192, 0.5)',
                    'rgba(153, 102, 255, 0.5)'
                ],
                borderColor: [
                    'rgba(255, 99, 132, 1)',
                    'rgba(54, 162, 235, 1)',
                    'rgba(255, 206, 86, 1)',
                    'rgba(75, 192, 192, 1)',
                    'rgba(153, 102, 255, 1)'
                ],
                borderWidth: 1
            }]
        },
        options: {
            responsive: true,
            scales: {
                y: {
                    beginAtZero: true
                }
            }
        }
    });
</script>
```

### 5.5 User Management

#### User Listing
```php
// views/admin/kelola_pengguna.php
<div class="user-management">
    <h2>Kelola Pengguna</h2>
    
    <div class="user-list">
        <table class="data-table">
            <thead>
                <tr>
                    <th>ID</th>
                    <th>Username</th>
                    <th>Email</th>
                    <th>Role</th>
                    <th>Tanggal Daftar</th>
                    <th>Aksi</th>
                </tr>
            </thead>
            <tbody>
                <?php foreach($users as $user): ?>
                <tr>
                    <td><?php echo $user['id']; ?></td>
                    <td><?php echo htmlspecialchars($user['username']); ?></td>
                    <td><?php echo htmlspecialchars($user['email']); ?></td>
                    <td>
                        <span class="role-badge role-<?php echo $user['role']; ?>">
                            <?php echo $user['role'] === 'admin' ? 'Admin' : 'User'; ?>
                        </span>
                    </td>
                    <td><?php echo date('d/m/Y', strtotime($user['created_at'])); ?></td>
                    <td class="actions">
                        <?php if($user['id'] != $_SESSION['user_id']): ?>
                            <?php if($user['role'] === 'user'): ?>
                            <a href="user_action.php?action=make_admin&id=<?php echo $user['id']; ?>" 
                               class="btn btn-sm btn-warning" 
                               onclick="return confirm('Jadikan user ini sebagai admin?')">Make Admin</a>
                            <?php else: ?>
                            <a href="user_action.php?action=remove_admin&id=<?php echo $user['id']; ?>" 
                               class="btn btn-sm btn-info" 
                               onclick="return confirm('Hapus hak admin dari user ini?')">Remove Admin</a>
                            <?php endif; ?>
                            
                            <a href="user_action.php?action=delete&id=<?php echo $user['id']; ?>" 
                               class="btn btn-sm btn-danger" 
                               onclick="return confirm('Yakin ingin menghapus user ini?')">Delete</a>
                        <?php else: ?>
                            <span class="text-muted">Current User</span>
                        <?php endif; ?>
                    </td>
                </tr>
                <?php endforeach; ?>
            </tbody>
        </table>
    </div>
    
    <!-- Pagination -->
    <?php if($totalPages > 1): ?>
    <div class="pagination">
        <?php for($i = 1; $i <= $totalPages; $i++): ?>
            <a href="?page=<?php echo $i; ?>" class="<?php echo $currentPage == $i ? 'active' : ''; ?>">
                <?php echo $i; ?>
            </a>
        <?php endfor; ?>
    </div>
    <?php endif; ?>
</div>
```

#### User Action Handler
```php
// views/admin/user_action.php
<?php
session_start();
require_once '../../controllers/AdminController.php';
require_once '../../controllers/AuthController.php';

$auth = new AuthController();
$auth->requireAdmin();

$adminController = new AdminController();

if(isset($_GET['action']) && isset($_GET['id'])) {
    $action = $_GET['action'];
    $userId = (int)$_GET['id'];
    
    switch($action) {
        case 'make_admin':
            $result = $adminController->updateUserRole($userId, 'admin');
            break;
            
        case 'remove_admin':
            $result = $adminController->updateUserRole($userId, 'user');
            break;
            
        case 'delete':
            $result = $adminController->deleteUser($userId);
            break;
            
        default:
            $result = ['success' => false, 'message' => 'Aksi tidak valid'];
    }
    
    $_SESSION['message'] = $result['message'];
    $_SESSION['message_type'] = $result['success'] ? 'success' : 'error';
}

header('Location: kelola_pengguna.php');
exit();
?>
```

### 5.6 Order Management

#### Order Listing
```php
// views/admin/kelola_pesanan.php
<div class="order-management">
    <h2>Kelola Pesanan</h2>
    
    <!-- Filter -->
    <div class="order-filter">
        <form method="GET" class="filter-form">
            <div class="form-group">
                <label for="status">Filter Status:</label>
                <select name="status" id="status" onchange="this.form.submit()">
                    <option value="">Semua Status</option>
                    <option value="pending" <?php echo $status === 'pending' ? 'selected' : ''; ?>>Menunggu</option>
                    <option value="processing" <?php echo $status === 'processing' ? 'selected' : ''; ?>>Diproses</option>
                    <option value="shipped" <?php echo $status === 'shipped' ? 'selected' : ''; ?>>Dikirim</option>
                    <option value="delivered" <?php echo $status === 'delivered' ? 'selected' : ''; ?>>Selesai</option>
                    <option value="cancelled" <?php echo $status === 'cancelled' ? 'selected' : ''; ?>>Dibatalkan</option>
                </select>
            </div>
        </form>
    </div>
    
    <div class="order-list">
        <table class="data-table">
            <thead>
                <tr>
                    <th>ID</th>
                    <th>Pelanggan</th>
                    <th>Total</th>
                    <th>Status</th>
                    <th>Tanggal</th>
                    <th>Aksi</th>
                </tr>
            </thead>
            <tbody>
                <?php foreach($orders as $order): ?>
                <tr>
                    <td><?php echo $order['id']; ?></td>
                    <td><?php echo htmlspecialchars($order['username']); ?></td>
                    <td>Rp <?php echo number_format($order['total_amount'], 0, ',', '.'); ?></td>
                    <td>
                        <span class="status-badge status-<?php echo $order['status']; ?>">
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
                        </span>
                    </td>
                    <td><?php echo date('d/m/Y H:i', strtotime($order['created_at'])); ?></td>
                    <td class="actions">
                        <a href="detail_pesanan.php?id=<?php echo $order['id']; ?>" class="btn btn-sm btn-info">Detail</a>
                        
                        <?php if($order['status'] !== 'cancelled' && $order['status'] !== 'delivered'): ?>
                        <div class="dropdown">
                            <button class="btn btn-sm btn-warning dropdown-toggle">Update Status</button>
                            <div class="dropdown-content">
                                <?php if($order['status'] !== 'pending'): ?>
                                <a href="order_action.php?action=update_status&id=<?php echo $order['id']; ?>&status=pending">Menunggu</a>
                                <?php endif; ?>
                                
                                <?php if($order['status'] !== 'processing'): ?>
                                <a href="order_action.php?action=update_status&id=<?php echo $order['id']; ?>&status=processing">Diproses</a>
                                <?php endif; ?>
                                
                                <?php if($order['status'] !== 'shipped'): ?>
                                <a href="order_action.php?action=update_status&id=<?php echo $order['id']; ?>&status=shipped">Dikirim</a>
                                <?php endif; ?>
                                
                                <?php if($order['status'] !== 'delivered'): ?>
                                <a href="order_action.php?action=update_status&id=<?php echo $order['id']; ?>&status=delivered">Selesai</a>
                                <?php endif; ?>
                            </div>
                        </div>
                        
                        <a href="order_action.php?action=cancel&id=<?php echo $order['id']; ?>" 
                           class="btn btn-sm btn-danger" 
                           onclick="return confirm('Yakin ingin membatalkan pesanan ini?')">Batalkan</a>
                        <?php endif; ?>
                    </td>
                </tr>
                <?php endforeach; ?>
            </tbody>
        </table>
    </div>
    
    <!-- Pagination -->
    <?php if($totalPages > 1): ?>
    <div class="pagination">
        <?php for($i = 1; $i <= $totalPages; $i++): ?>
            <a href="?page=<?php echo $i; ?>&status=<?php echo $status; ?>" 
               class="<?php echo $currentPage == $i ? 'active' : ''; ?>">
                <?php echo $i; ?>
            </a>
        <?php endfor; ?>
    </div>
    <?php endif; ?>
</div>
```

#### Order Detail
```php
// views/admin/detail_pesanan.php
<div class="order-detail">
    <div class="detail-header">
        <h2>Detail Pesanan #<?php echo $order['id']; ?></h2>
        <div class="status-badge status-<?php echo $order['status']; ?>">
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
    </div>
    
    <div class="detail-content">
        <div class="detail-section">
            <h3>Informasi Pelanggan</h3>
            <table class="detail-table">
                <tr>
                    <th>Username:</th>
                    <td><?php echo htmlspecialchars($order['username']); ?></td>
                </tr>
                <tr>
                    <th>Email:</th>
                    <td><?php echo htmlspecialchars($order['email']); ?></td>
                </tr>
            </table>
        </div>
        
        <div class="detail-section">
            <h3>Informasi Pengiriman</h3>
            <div class="shipping-address">
                <?php echo nl2br(htmlspecialchars($order['shipping_address'])); ?>
            </div>
        </div>
        
        <div class="detail-section">
            <h3>Informasi Pembayaran</h3>
            <table class="detail-table">
                <tr>
                    <th>Metode Pembayaran:</th>
                    <td>
                        <?php 
                        $paymentLabels = [
                            'transfer_bank' => 'Transfer Bank',
                            'e_wallet' => 'E-Wallet',
                            'cod' => 'Cash on Delivery (COD)'
                        ];
                        echo $paymentLabels[$order['payment_method']] ?? $order['payment_method'];
                        ?>
                    </td>
                </tr>
                <tr>
                    <th>Total Pembayaran:</th>
                    <td>Rp <?php echo number_format($order['total_amount'], 0, ',', '.'); ?></td>
                </tr>
                <tr>
                    <th>Tanggal Pesanan:</th>
                    <td><?php echo date('d/m/Y H:i', strtotime($order['created_at'])); ?></td>
                </tr>
            </table>
        </div>
        
        <div class="detail-section">
            <h3>Item Pesanan</h3>
            <table class="items-table">
                <thead>
                    <tr>
                        <th>Produk</th>
                        <th>Harga</th>
                        <th>Jumlah</th>
                        <th>Subtotal</th>
                    </tr>
                </thead>
                <tbody>
                    <?php foreach($orderItems as $item): ?>
                    <tr>
                        <td class="product-cell">
                            <img src="../../assets/images/<?php echo $item['product_image']; ?>" 
                                 alt="<?php echo $item['product_name']; ?>" class="product-thumbnail">
                            <span><?php echo htmlspecialchars($item['product_name']); ?></span>
                        </td>
                        <td>Rp <?php echo number_format($item['price'], 0, ',', '.'); ?></td>
                        <td><?php echo $item['quantity']; ?></td>
                        <td>Rp <?php echo number_format($item['price'] * $item['quantity'], 0, ',', '.'); ?></td>
                    </tr>
                    <?php endforeach; ?>
                </tbody>
                <tfoot>
                    <tr>
                        <th colspan="3">Total</th>
                        <th>Rp <?php echo number_format($order['total_amount'], 0, ',', '.'); ?></th>
                    </tr>
                </tfoot>
            </table>
        </div>
    </div>
    
    <div class="detail-actions">
        <a href="kelola_pesanan.php" class="btn btn-secondary">Kembali</a>
        
        <?php if($order['status'] !== 'cancelled' && $order['status'] !== 'delivered'): ?>
            <div class="action-group">
                <div class="dropdown">
                    <button class="btn btn-warning dropdown-toggle">Update Status</button>
                    <div class="dropdown-content">
                        <?php if($order['status'] !== 'pending'): ?>
                        <a href="order_action.php?action=update_status&id=<?php echo $order['id']; ?>&status=pending&redirect=detail">Menunggu</a>
                        <?php endif; ?>
                        
                        <?php if($order['status'] !== 'processing'): ?>
                        <a href="order_action.php?action=update_status&id=<?php echo $order['id']; ?>&status=processing&redirect=detail">Diproses</a>
                        <?php endif; ?>
                        
                        <?php if($order['status'] !== 'shipped'): ?>
                        <a href="order_action.php?action=update_status&id=<?php echo $order['id']; ?>&status=shipped&redirect=detail">Dikirim</a>
                        <?php endif; ?>
                        
                        <?php if($order['status'] !== 'delivered'): ?>
                        <a href="order_action.php?action=update_status&id=<?php echo $order['id']; ?>&status=delivered&redirect=detail">Selesai</a>
                        <?php endif; ?>
                    </div>
                </div>
                
                <a href="order_action.php?action=cancel&id=<?php echo $order['id']; ?>&redirect=detail" 
                   class="btn btn-danger" 
                   onclick="return confirm('Yakin ingin membatalkan pesanan ini?')">Batalkan Pesanan</a>
            </div>
        <?php endif; ?>
    </div>
</div>
```

### 5.7 System Monitoring

#### Low Stock Alert
```php
// In dashboard.php
<div class="alert-section">
    <h3>Low Stock Alert</h3>
    <?php if(!empty($stats['low_stock_products'])): ?>
    <div class="alert alert-warning">
        <p><i class="fas fa-exclamation-triangle"></i> <?php echo count($stats['low_stock_products']); ?> products are running low on stock!</p>
    </div>
    <table class="data-table">
        <thead>
            <tr>
                <th>Product</th>
                <th>Current Stock</th>
                <th>Action</th>
            </tr>
        </thead>
        <tbody>
            <?php foreach($stats['low_stock_products'] as $product): ?>
            <tr>
                <td><?php echo htmlspecialchars($product['name']); ?></td>
                <td>
                    <span class="stock-badge <?php echo $product['stock'] <= 0 ? 'out-of-stock' : 'low-stock'; ?>">
                        <?php echo $product['stock']; ?>
                    </span>
                </td>
                <td>
                    <a href="edit_produk.php?id=<?php echo $product['id']; ?>" class="btn btn-sm btn-warning">Update Stock</a>
                </td>
            </tr>
            <?php endforeach; ?>
        </tbody>
    </table>
    <?php else: ?>
    <div class="alert alert-success">
        <p><i class="fas fa-check-circle"></i> All products have sufficient stock.</p>
    </div>
    <?php endif; ?>
</div>
```

#### Recent Orders
```php
// In dashboard.php
<div class="recent-section">
    <h3>Recent Orders</h3>
    <?php if(!empty($stats['recent_orders'])): ?>
    <table class="data-table">
        <thead>
            <tr>
                <th>Order ID</th>
                <th>Customer</th>
                <th>Amount</th>
                <th>Status</th>
                <th>Date</th>
                <th>Action</th>
            </tr>
        </thead>
        <tbody>
            <?php foreach($stats['recent_orders'] as $order): ?>
            <tr>
                <td>#<?php echo $order['id']; ?></td>
                <td><?php echo htmlspecialchars($order['username']); ?></td>
                <td>Rp <?php echo number_format($order['total_amount'], 0, ',', '.'); ?></td>
                <td>
                    <span class="status-badge status-<?php echo $order['status']; ?>">
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
                    </span>
                </td>
                <td><?php echo date('d/m/Y H:i', strtotime($order['created_at'])); ?></td>
                <td>
                    <a href="detail_pesanan.php?id=<?php echo $order['id']; ?>" class="btn btn-sm btn-info">View</a>
                </td>
            </tr>
            <?php endforeach; ?>
        </tbody>
    </table>
    <div class="view-all">
        <a href="kelola_pesanan.php" class="btn btn-outline">View All Orders</a>
    </div>
    <?php else: ?>
    <div class="alert alert-info">
        <p>No recent orders found.</p>
    </div>
    <?php endif; ?>
</div>
```

## Praktikum

### Latihan 1: Analisis Dashboard
1. Buka file `controllers/AdminController.php`
2. Analisis method `getDashboardStats()`
3. Identifikasi query untuk statistik
4. Buat diagram flow untuk dashboard data

### Latihan 2: Testing Admin Features
1. Login sebagai admin
2. Explore dashboard dan statistik
3. Test user management (role update, delete)
4. Test order management (status update, view details)
5. Test product management

### Latihan 3: Implementasi Fitur Baru
1. Tambahkan filter tanggal untuk statistik
2. Implementasikan export data ke CSV/Excel
3. Buat sistem notifikasi admin untuk pesanan baru
4. Tambahkan chart untuk kategori produk

```php
// Export to CSV example
public function exportOrdersToCSV($startDate = null, $endDate = null) {
    $query = "SELECT o.id, u.username, u.email, o.total_amount, o.status, 
                    o.shipping_address, o.payment_method, o.created_at 
             FROM orders o 
             JOIN users u ON o.user_id = u.id";
    
    $params = [];
    
    if ($startDate && $endDate) {
        $query .= " WHERE o.created_at BETWEEN :start_date AND :end_date";
        $params[':start_date'] = $startDate . ' 00:00:00';
        $params[':end_date'] = $endDate . ' 23:59:59';
    }
    
    $query .= " ORDER BY o.created_at DESC";
    
    $stmt = $this->order->conn->prepare($query);
    
    foreach ($params as $key => $value) {
        $stmt->bindValue($key, $value);
    }
    
    $stmt->execute();
    $orders = $stmt->fetchAll(PDO::FETCH_ASSOC);
    
    // Create CSV
    $filename = 'orders_export_' . date('Y-m-d') . '.csv';
    $output = fopen('php://output', 'w');
    
    // Set headers
    header('Content-Type: text/csv');
    header('Content-Disposition: attachment; filename="' . $filename . '"');
    
    // Add CSV headers
    fputcsv($output, ['Order ID', 'Customer', 'Email', 'Amount', 'Status', 'Address', 'Payment Method', 'Date']);
    
    // Add data rows
    foreach ($orders as $order) {
        fputcsv($output, [
            $order['id'],
            $order['username'],
            $order['email'],
            $order['total_amount'],
            $order['status'],
            $order['shipping_address'],
            $order['payment_method'],
            $order['created_at']
        ]);
    }
    
    fclose($output);
    exit;
}
```

### Latihan 4: Advanced Analytics
1. Implementasikan customer segmentation
2. Buat analisis cohort untuk retention
3. Implementasikan heatmap untuk aktivitas user
4. Buat dashboard untuk product performance

## Tugas

1. **Enhancement**: Implementasikan dashboard yang lebih komprehensif
2. **Analytics**: Buat sistem reporting bulanan dengan export
3. **Security**: Implementasikan log aktivitas admin
4. **Performance**: Optimasi query untuk dashboard

## Evaluasi

### Checklist Penyelesaian
- [ ] Memahami role-based access control
- [ ] Dapat menggunakan AdminController
- [ ] Memahami dashboard analytics
- [ ] Dapat mengelola user sebagai admin
- [ ] Dapat mengelola pesanan sebagai admin
- [ ] Memahami data visualization
- [ ] Dapat mengimplementasikan fitur monitoring

### Pertanyaan Review
1. Apa perbedaan antara authentication dan authorization?
2. Mengapa perlu role-based access control?
3. Bagaimana cara mengoptimalkan query untuk dashboard?
4. Apa keuntungan menggunakan Chart.js?
5. Bagaimana cara mengamankan admin panel?
6. Apa saja KPI yang penting untuk e-commerce?
7. Bagaimana cara handle export data yang besar?

### Mini Project
Buat admin dashboard dengan fitur:
- Comprehensive analytics dengan multiple charts
- User management dengan role system
- Order management dengan filtering dan export
- Product performance analysis
- System monitoring dan alerts
- Activity logging dan audit trail

---

**Catatan**: Pastikan memahami konsep security dan best practices dalam implementasi admin panel sebelum melanjutkan ke bab selanjutnya.