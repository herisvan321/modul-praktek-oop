# Modul Praktek: Game Store OOP

---

**📚 Panduan Pembelajaran Object-Oriented Programming dengan PHP**

**Penulis:** Herisvan Hendra, M.Pd.T  
**Institusi:** Universitas PGRI Sumatera Barat 
**Tahun:** 2025 
**Versi:** 1.0

**📋 Modul Pembelajaran:**
- BAB 1: Pengenalan dan Setup Project
- BAB 2: Sistem Autentikasi
- BAB 3: Manajemen Produk
- BAB 4: Sistem Keranjang dan Pesanan
- BAB 5: Admin Dashboard
- BAB 6: Keamanan dan Optimasi
- BAB 7: Analisis CSS Framework

---

## Deskripsi Project

**Game Store OOP** adalah aplikasi e-commerce berbasis web yang dibangun menggunakan PHP dengan paradigma Object-Oriented Programming (OOP). Aplikasi ini dirancang sebagai platform jual beli game digital yang menyediakan fitur lengkap untuk pengguna dan administrator.

### Teknologi yang Digunakan
- **Backend**: PHP 7.4+ dengan OOP
- **Database**: MySQL 5.7+
- **Frontend**: HTML5, CSS3, JavaScript (Vanilla)
- **Styling**: Custom CSS dengan responsive design
- **Web Server**: Apache/Nginx
- **Architecture**: Model-View-Controller (MVC)

### Fitur Utama

#### Untuk Pengguna (User)
1. **Sistem Autentikasi**
   - Registrasi akun baru dengan validasi
   - Login/logout dengan session management
   - Password hashing untuk keamanan

2. **Katalog Produk**
   - Browse produk game dengan kategori
   - Search dan filter produk
   - Detail produk dengan deskripsi lengkap
   - Sistem rating dan review

3. **Keranjang Belanja**
   - Add/remove produk ke keranjang
   - Update quantity produk
   - Real-time cart notification
   - Persistent cart dengan session

4. **Sistem Pemesanan**
   - Checkout dengan multiple payment methods
   - Order tracking dan history
   - Invoice generation
   - Email notifications

5. **User Interface**
   - Responsive design untuk mobile dan desktop
   - Custom CSS styling dengan modern design
   - Toast notifications untuk feedback
   - Loading states dan animations
   - Clean dan user-friendly interface
   - Dark/light theme support

#### Untuk Administrator (Admin)
1. **Dashboard Analytics**
   - Real-time statistics dan KPIs
   - Sales charts dan graphs
   - Revenue tracking
   - User activity monitoring

2. **Manajemen Produk**
   - CRUD operations untuk produk
   - Bulk product management
   - Image upload dan management
   - Stock monitoring dengan alerts

3. **Manajemen User**
   - User listing dengan pagination
   - Role management (user/admin)
   - User activity logs
   - Account management

4. **Manajemen Pesanan**
   - Order listing dengan filtering
   - Order status management
   - Order details dan tracking
   - Bulk order operations

5. **Sistem Monitoring**
   - Performance monitoring
   - Error logging dan tracking
   - Security audit logs
   - System health checks

### Arsitektur Aplikasi

#### Model-View-Controller (MVC)
```
├── models/           # Data layer (Database interactions)
│   ├── User.php
│   ├── Product.php
│   ├── Cart.php
│   ├── Order.php
│   └── OrderItem.php
├── controllers/      # Business logic layer
│   ├── AuthController.php
│   ├── ProductController.php
│   ├── CartController.php
│   ├── OrderController.php
│   └── AdminController.php
├── views/           # Presentation layer
│   ├── layouts/     # Shared layouts
│   ├── user/        # User interfaces
│   └── admin/       # Admin interfaces
├── config/          # Configuration files
│   ├── database.php
│   └── session_config.php
├── assets/          # Static assets
│   ├── css/         # Stylesheets
│   │   └── style.css (2332+ lines)
│   └── images/      # Product images
│       ├── *.jpg    # Game cover images
│       └── *.svg    # Vector graphics
```

#### Database Schema
```sql
-- Users table
CREATE TABLE users (
    id INT PRIMARY KEY AUTO_INCREMENT,
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    password VARCHAR(255) NOT NULL,
    role ENUM('user', 'admin') DEFAULT 'user',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Products table
CREATE TABLE products (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL,
    description TEXT,
    price DECIMAL(10,2) NOT NULL,
    image VARCHAR(255),
    category VARCHAR(50),
    stock INT DEFAULT 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Orders table
CREATE TABLE orders (
    id INT PRIMARY KEY AUTO_INCREMENT,
    user_id INT,
    total_amount DECIMAL(10,2) NOT NULL,
    status ENUM('pending', 'processing', 'shipped', 'delivered', 'cancelled') DEFAULT 'pending',
    shipping_address TEXT,
    payment_method VARCHAR(50),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id)
);

-- Order Items table
CREATE TABLE order_items (
    id INT PRIMARY KEY AUTO_INCREMENT,
    order_id INT,
    product_id INT,
    quantity INT NOT NULL,
    price DECIMAL(10,2) NOT NULL,
    FOREIGN KEY (order_id) REFERENCES orders(id),
    FOREIGN KEY (product_id) REFERENCES products(id)
);
```

## Frontend Styling & CSS

### Custom CSS Framework
Aplikasi ini menggunakan custom CSS framework yang telah dioptimasi untuk:

#### Core Features
- **Reset & Base Styles**: Normalisasi cross-browser
- **Responsive Grid System**: Flexible layout untuk semua device
- **Component Library**: Button, form, card, dan navigation components
- **Animation System**: Loading animations, transitions, dan micro-interactions
- **Color Palette**: Consistent color scheme dengan CSS variables

#### File Structure
```
assets/css/style.css (2332+ lines)
├── Reset & Base Styles
├── Loading Animations
├── Form Components
├── Button Styles
├── Navigation
├── Card Components
├── Admin Dashboard
├── Responsive Design
└── Utility Classes
```

#### Key CSS Features
- **Modern CSS3**: Flexbox, Grid, Custom Properties
- **Responsive Design**: Mobile-first approach
- **Performance Optimized**: Efficient selectors dan minimal reflow
- **Accessibility**: ARIA-friendly styling
- **Cross-browser**: Support untuk modern browsers

#### Usage dalam Views
```html
<!-- Semua view files menggunakan CSS utama -->
<link rel="stylesheet" href="../../assets/css/style.css">

<!-- Component examples -->
<button class="btn btn-primary">Primary Button</button>
<div class="card">Card Content</div>
<form class="form-modern">Form Elements</form>
```

## Struktur Modul Praktek

Modul praktek ini terdiri dari 6 bab yang disusun secara sistematis untuk pembelajaran bertahap:

### [BAB 1: Pengenalan dan Setup Project](bab-1-pengenalan-dan-setup.md)
**Durasi**: 2-3 jam

**Topik yang Dibahas**:
- Konsep dasar OOP dalam PHP
- Setup environment development
- Struktur project dan MVC pattern
- Database setup dan konfigurasi
- Testing environment

**Learning Outcomes**:
- Memahami konsep OOP dan MVC
- Dapat setup environment development
- Memahami struktur project
- Dapat menjalankan aplikasi

**Praktikum**:
- Setup XAMPP/MAMP
- Clone dan setup project
- Database import dan konfigurasi
- Testing aplikasi

### [BAB 2: Sistem Autentikasi dan Manajemen User](bab-2-sistem-autentikasi.md)
**Durasi**: 3-4 jam

**Topik yang Dibahas**:
- AuthController implementation
- User model dan database operations
- Session management
- Password hashing dan security
- Input validation

**Learning Outcomes**:
- Memahami sistem autentikasi
- Dapat mengimplementasikan login/register
- Memahami session management
- Memahami security best practices

**Praktikum**:
- Analisis AuthController
- Testing login/register functionality
- Implementasi validation
- Security testing

### [BAB 3: Manajemen Produk dan Katalog Game](bab-3-manajemen-produk.md)
**Durasi**: 4-5 jam

**Topik yang Dibahas**:
- ProductController dan Product model
- CRUD operations untuk produk
- Image upload dan management
- Search dan filtering
- Category management

**Learning Outcomes**:
- Memahami CRUD operations
- Dapat mengimplementasikan search/filter
- Memahami file upload handling
- Memahami data pagination

**Praktikum**:
- Analisis ProductController
- Testing CRUD operations
- Implementasi search/filter
- Image upload testing

### [BAB 4: Sistem Keranjang Belanja dan Manajemen Pesanan](bab-4-sistem-keranjang-dan-pesanan.md)
**Durasi**: 4-5 jam

**Topik yang Dibahas**:
- CartController dan Cart model
- OrderController dan Order model
- Shopping cart functionality
- Checkout process
- Order management

**Learning Outcomes**:
- Memahami shopping cart logic
- Dapat mengimplementasikan checkout
- Memahami order management
- Memahami transaction handling

**Praktikum**:
- Analisis cart functionality
- Testing checkout process
- Order management testing
- Transaction flow analysis

### [BAB 5: Admin Dashboard dan Manajemen Sistem](bab-5-admin-dashboard.md)
**Durasi**: 3-4 jam

**Topik yang Dibahas**:
- AdminController implementation
- Dashboard analytics
- User management
- Order management
- System monitoring

**Learning Outcomes**:
- Memahami admin panel functionality
- Dapat mengimplementasikan analytics
- Memahami role-based access control
- Memahami system monitoring

**Praktikum**:
- Analisis AdminController
- Dashboard testing
- User management testing
- Analytics implementation

### [BAB 6: Keamanan dan Optimasi Aplikasi](bab-6-keamanan-dan-optimasi.md)
**Durasi**: 4-5 jam

**Topik yang Dibahas**:
- Web application security
- Input validation dan sanitization
- SQL injection prevention
- XSS protection
- Performance optimization
- Caching strategies

**Learning Outcomes**:
- Memahami web security concepts
- Dapat mengimplementasikan security measures
- Memahami performance optimization
- Dapat mengimplementasikan caching

**Praktikum**:
- Security audit
- Performance testing
- Security implementation
- Optimization implementation

### [BAB 7: Analisis dan Penjelasan File CSS Framework](bab-7-css-framework.md)
**Durasi**: 2.5 jam

**Topik yang Dibahas**:
- Analisis struktur file `style.css` (2332+ baris)
- Pemahaman custom CSS framework
- Implementasi responsive design
- Animasi dan transisi CSS
- Best practices CSS organization
- Performance optimization CSS

**Learning Outcomes**:
- Memahami struktur dan organisasi CSS yang baik
- Dapat menganalisis dan memodifikasi custom CSS framework
- Memahami implementasi responsive design
- Dapat menerapkan animasi dan transisi CSS
- Memahami best practices dalam penulisan CSS
- Dapat melakukan optimasi CSS untuk performa

**Praktikum**:
- Analisis file `assets/css/style.css`
- Modifikasi tema dan styling
- Implementasi fitur dark/light mode
- Optimasi CSS untuk performa
- Testing responsive design
- Customization CSS framework

## Cara Penggunaan Modul

### Persiapan
1. **Setup Environment**
   - Install XAMPP/MAMP/WAMP
   - Pastikan PHP 7.4+ dan MySQL 5.7+ tersedia
   - Clone project dari repository

2. **Database Setup**
   ```bash
   # Import database
   mysql -u root -p game_store_oop < database.sql
   ```

3. **Configuration**
   - Update database credentials di `config/database.php`
   - Set proper file permissions
   - Test aplikasi di browser

### Pembelajaran
1. **Baca Teori**: Mulai dengan membaca materi di setiap bab
2. **Analisis Kode**: Pelajari implementasi kode yang ada
3. **Praktikum**: Lakukan latihan yang disediakan
4. **Tugas**: Kerjakan tugas untuk memperdalam pemahaman
5. **Evaluasi**: Gunakan checklist dan pertanyaan review

### Tips Pembelajaran
1. **Hands-on Practice**: Selalu praktik langsung dengan kode
2. **Debugging**: Pelajari cara debugging dan error handling
3. **Documentation**: Baca dokumentasi PHP dan MySQL
4. **Best Practices**: Ikuti coding standards dan best practices
5. **Testing**: Selalu test fitur yang dibuat

## Prerequisites

### Pengetahuan Dasar
- **PHP Fundamentals**: Variables, functions, arrays, loops
- **HTML/CSS**: Basic web development
- **MySQL**: Basic SQL queries
- **Web Concepts**: HTTP, sessions, cookies

### Software Requirements
- **Web Server**: Apache/Nginx
- **PHP**: Version 7.4 or higher
- **MySQL**: Version 5.7 or higher
- **Browser**: Modern web browser
- **Text Editor**: VS Code, Sublime Text, atau IDE lainnya

## Learning Path

### Beginner Level (Bab 1-2)
- Fokus pada setup dan konsep dasar
- Pahami struktur project dan MVC
- Pelajari autentikasi dan session

### Intermediate Level (Bab 3-4)
- Implementasi CRUD operations
- Pelajari business logic yang kompleks
- Pahami data relationships

### Advanced Level (Bab 5-6)
- Admin panel dan analytics
- Security dan optimization
- Performance monitoring

## Assessment

### Kriteria Penilaian
1. **Pemahaman Konsep** (25%)
   - OOP principles
   - MVC architecture
   - Database design

2. **Implementasi Kode** (35%)
   - Code quality
   - Best practices
   - Functionality

3. **Problem Solving** (25%)
   - Debugging skills
   - Logic implementation
   - Error handling

4. **Security & Performance** (15%)
   - Security measures
   - Code optimization
   - Performance considerations

### Final Project
Setelah menyelesaikan semua bab, mahasiswa diharapkan dapat:
- Membangun aplikasi e-commerce sederhana
- Mengimplementasikan fitur-fitur utama
- Menerapkan security best practices
- Melakukan optimization dan monitoring

## Resources

### Dokumentasi
- [PHP Official Documentation](https://www.php.net/docs.php)
- [MySQL Documentation](https://dev.mysql.com/doc/)
- [MDN Web Docs](https://developer.mozilla.org/)

### Tools
- [XAMPP](https://www.apachefriends.org/)
- [phpMyAdmin](https://www.phpmyadmin.net/)
- [Visual Studio Code](https://code.visualstudio.com/)
- [Postman](https://www.postman.com/) (untuk API testing)

### Libraries & Frameworks
- [Chart.js](https://www.chartjs.org/) (untuk dashboard charts)
- **Custom CSS Framework** (included - 2332+ lines)
- [Bootstrap](https://getbootstrap.com/) (optional untuk styling tambahan)
- [jQuery](https://jquery.com/) (optional untuk JavaScript)

### CSS Resources
- [MDN CSS Reference](https://developer.mozilla.org/en-US/docs/Web/CSS)
- [CSS Grid Guide](https://css-tricks.com/snippets/css/complete-guide-grid/)
- [Flexbox Guide](https://css-tricks.com/snippets/css/a-guide-to-flexbox/)
- [Responsive Design Patterns](https://web.dev/responsive-web-design-basics/)

## Troubleshooting

### Common Issues
1. **Database Connection Error**
   - Check database credentials
   - Ensure MySQL service is running
   - Verify database exists

2. **Permission Errors**
   - Set proper file permissions (755 for directories, 644 for files)
   - Check web server user permissions

3. **Session Issues**
   - Check session configuration
   - Verify session directory permissions
   - Clear browser cookies

4. **Image Upload Issues**
   - Check upload directory permissions
   - Verify file size limits
   - Check allowed file types

### Getting Help
- Check error logs in web server
- Use browser developer tools
- Enable PHP error reporting for development
- Consult documentation and online resources

## Contributing

Jika Anda menemukan bug atau ingin berkontribusi:
1. Fork repository
2. Create feature branch
3. Make changes
4. Submit pull request

## License

Project ini dibuat untuk tujuan edukasi dan pembelajaran. Silakan gunakan dan modifikasi sesuai kebutuhan pembelajaran.

---

**Selamat Belajar!** 🚀

Modul ini dirancang untuk memberikan pemahaman komprehensif tentang pengembangan aplikasi web menggunakan PHP OOP. Ikuti setiap bab secara berurutan dan jangan ragu untuk bereksperimen dengan kode yang ada.