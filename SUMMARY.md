# 📚 Code Review Guidelines - Summary

Ringkasan lengkap panduan code review untuk tim development

> **[🚀 Quick Reference Card](./QUICK-REFERENCE.md)** - Panduan cepat yang bisa di-print

---

## 📦 Struktur Dokumentasi

```
code-review/
├── README.md                          # Panduan umum (semua bahasa)
├── SUMMARY.md                         # File ini - Overview lengkap
├── QUICK-REFERENCE.md                 # Quick reference card (printable)
├── WHY-NO-ANY.md                      # Penjelasan kenapa 'any' type harus dihindari
│
├── backend/
│   ├── README.md                      # Node.js/TypeScript guidelines
│   ├── EXAMPLES.md                    # Node.js examples (Express, NestJS)
│   ├── GOLANG.md                      # Golang guidelines (Gin, Echo, Fiber)
│   ├── PHP.md                         # PHP guidelines (Laravel, Symfony)
│   └── PHP-LARAVEL-EXAMPLE.md         # Laravel complete example
│
└── frontend/
    ├── README.md                      # React/Vue/Angular guidelines
    └── EXAMPLES.md                    # Frontend examples (hooks, components)
```

---

## 🎯 Panduan yang Tersedia

### **📚 Panduan Umum** ([README.md](./README.md))
Berlaku untuk semua bahasa pemrograman:
- ✅ L1 - Basic Logic / Syntax Error (Rp 200)
- ✅ L2 - Code Style / Konsistensi (Rp 400) + **TypeScript `any` type**
- ✅ L3 - Logic / Flow Error (Rp 800)
- ✅ L3.5 - Error Handling (Rp 1.000)
- ✅ L4 - Data Handling / Validation (Rp 1.200)
- ✅ L4.5 - Redundancy / Duplicate Logic (Rp 1.400)
- ✅ L5 - Performance Issue (Rp 1.600)
- ✅ L6 - Security / Access Control (Rp 2.000)
- ✅ L7 - Architectural / Design Flaw (Rp 2.500)
- ✅ L8 - Regression / Repeat Offense (×2 denda)
- ✅ L9 - Testing / Documentation Gap (Rp 600)

### **🔧 Backend Guidelines**

#### **Node.js / TypeScript** ([backend/README.md](./backend/README.md))
- API Development (response format, HTTP status codes)
- Database & ORM (N+1 queries, transactions, SQL injection)
- Authentication & Authorization (JWT, password hashing, IDOR)
- Middleware & Error Handling
- Backend Architecture (layered architecture)
- Testing (unit & integration tests)
- **Examples**: [EXAMPLES.md](./backend/EXAMPLES.md)

#### **🔵 Golang** ([backend/GOLANG.md](./backend/GOLANG.md))
- Error Handling (no panic, proper error wrapping)
- Goroutine Management (WaitGroup, semaphore, race conditions)
- Context untuk Timeout & Cancellation
- Clean Architecture (domain, repository, service, handler)
- Complete API Example (Gin framework)
- Testing (table-driven tests, mocks)

#### **🐘 PHP** ([backend/PHP.md](./backend/PHP.md))
- Type Safety (strict_types, type hints)
- Laravel Best Practices (service layer, repository pattern)
- Validation (Form Requests)
- Security (password hashing, XSS, CSRF, SQL injection)
- Performance (eager loading, caching, queues)
- **Laravel Example**: [PHP-LARAVEL-EXAMPLE.md](./backend/PHP-LARAVEL-EXAMPLE.md)

### **🎨 Frontend Guidelines** ([frontend/README.md](./frontend/README.md))

#### **React / Vue / Angular**
- Component Structure (TypeScript interfaces, props validation)
- State Management (immutable updates, useEffect dependencies)
- API Error Handling
- Performance (lazy loading, memoization, virtualization)
- Security (XSS prevention, localStorage)
- Accessibility (semantic HTML, ARIA labels)
- **Examples**: [EXAMPLES.md](./frontend/EXAMPLES.md)

---

## 🎯 Quick Start

### **Untuk Backend Developer:**

1. **Pilih teknologi Anda:**
   - Node.js/TypeScript → [backend/README.md](./backend/README.md)
   - Golang → [backend/GOLANG.md](./backend/GOLANG.md)
   - PHP/Laravel → [backend/PHP.md](./backend/PHP.md)

2. **Baca panduan umum:** [README.md](./README.md)

3. **Lihat contoh implementasi:**
   - Node.js → [backend/EXAMPLES.md](./backend/EXAMPLES.md)
   - Laravel → [backend/PHP-LARAVEL-EXAMPLE.md](./backend/PHP-LARAVEL-EXAMPLE.md)

4. **Gunakan checklist sebelum commit**

### **Untuk Frontend Developer:**

1. **Baca panduan frontend:** [frontend/README.md](./frontend/README.md)

2. **Baca panduan umum:** [README.md](./README.md)

3. **Lihat contoh implementasi:** [frontend/EXAMPLES.md](./frontend/EXAMPLES.md)

4. **Gunakan checklist sebelum commit**

---

## ✅ Checklist Sebelum Commit (Ringkasan)

### **Semua Developer**
- [ ] No syntax error atau typo
- [ ] Consistent naming convention
- [ ] No magic numbers/strings
- [ ] Error handling lengkap
- [ ] Input validation
- [ ] No security vulnerabilities
- [ ] Logic di layer yang tepat
- [ ] Ada unit test
- [ ] Code documented

### **Backend Specific**
- [ ] Consistent API response format
- [ ] Correct HTTP status codes
- [ ] No N+1 queries
- [ ] Use database transactions
- [ ] Authentication & authorization
- [ ] Password hashing
- [ ] No SQL injection

### **Frontend Specific**
- [ ] TypeScript interfaces untuk props
- [ ] No XSS vulnerabilities
- [ ] Proper state management
- [ ] Performance optimized (lazy loading, memoization)
- [ ] Accessibility (semantic HTML, ARIA)
- [ ] Error boundaries

---

## 🛠️ Tools Recommended

### **Backend**
- **ESLint** - Code quality
- **Prettier** - Code formatting
- **Jest/Vitest** - Testing
- **Swagger/OpenAPI** - API documentation

### **Frontend**
- **ESLint + Prettier** - Code quality
- **TypeScript** - Type safety
- **React Testing Library** - Component testing
- **Playwright** - E2E testing

### **Golang**
- **golangci-lint** - Linting
- **gofmt** - Formatting
- **testify** - Testing assertions

### **PHP**
- **PHP CS Fixer** - Code style
- **PHPStan** - Static analysis
- **PHPUnit** - Testing

---

## 📊 Sistem Denda

| Level | Denda | Contoh |
|:------|:------|:-------|
| L1 | Rp 200 | Typo, syntax error |
| L2 | Rp 400 | Code style, naming |
| L3 | Rp 800 | Logic error |
| L3.5 | Rp 1.000 | Error handling |
| L4 | Rp 1.200 | Data validation |
| L4.5 | Rp 1.400 | Code duplication |
| L5 | Rp 1.600 | Performance issue |
| L6 | Rp 2.000 | Security vulnerability |
| L7 | Rp 2.500 | Architecture flaw |
| L8 | ×2 denda | Repeat offense |
| L9 | Rp 600 | No test/docs |

---

## 🎓 Learning Path

### **Junior Developer (0-1 tahun)**
1. Fokus pada L1-L2 (basic logic & code style)
2. Pelajari error handling (L3.5)
3. Mulai belajar testing (L9)

### **Mid Developer (1-3 tahun)**
1. Master L1-L4 (logic, error handling, validation)
2. Pelajari performance optimization (L5)
3. Pahami security best practices (L6)

### **Senior Developer (3+ tahun)**
1. Master semua level
2. Fokus pada architecture (L7)
3. Mentor junior developers
4. Review code dengan detail

---

## 💡 Tips Sukses

1. **Setup ESLint + Prettier** - Auto-fix banyak error L1 & L2
2. **Pre-commit Hook (Husky)** - Prevent bad code dari masuk repo
3. **Code Review Checklist** - Gunakan checklist sebelum submit PR
4. **Pair Programming** - Kurangi error dengan review real-time
5. **Test-Driven Development** - Tulis test dulu, code kemudian
6. **Regular Refactoring** - Jangan biarkan technical debt menumpuk
7. **Learn from Mistakes** - Catat error yang sering terjadi
8. **CI/CD Pipeline** - Automated testing sebelum merge

---

## 🎯 Goal

Bukan untuk "menghukum" developer, tapi untuk **membangun habit menulis code berkualitas tinggi**.

> "The best code is code that doesn't need to be reviewed because it's already perfect. But since that's impossible, let's at least make it as good as possible before review."

---

**Happy Coding! 🚀**
