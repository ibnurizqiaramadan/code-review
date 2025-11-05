# 🔧 Backend Code Review Guidelines

Panduan khusus untuk code review backend

## 🎯 Pilih Teknologi Backend Anda

- **[Node.js / TypeScript](./README.md)** - Express, NestJS, Fastify
- **[🔵 Golang](./GOLANG.md)** - Gin, Echo, Fiber, Chi
- **[🐘 PHP](./PHP.md)** - Laravel, Symfony, CodeIgniter

---

## 📚 Panduan Umum Backend (Node.js/TypeScript)

---

## 📋 Backend-Specific Error Levels

| Level | Jenis Kesalahan | Contoh Backend | Denda |
|:------|:----------------|:---------------|:------|
| **L1** | Basic Logic | Typo di route path, salah HTTP method | Rp 200 |
| **L2** | Code Style | Inconsistent response format, naming convention | Rp 400 |
| **L3** | Logic Error | Async/await salah, race condition | Rp 800 |
| **L3.5** | Error Handling | Error gak di-catch, status code salah | Rp 1.000 |
| **L4** | Data Validation | Input gak di-validate, SQL injection | Rp 1.200 |
| **L5** | Performance | N+1 query, gak pakai caching, memory leak | Rp 1.600 |
| **L6** | Security | Auth bypass, credential exposed, CORS misconfigured | Rp 2.000 |
| **L7** | Architecture | Logic di controller, tight coupling | Rp 2.500 |
| **L9** | Testing/Docs | API tanpa docs, endpoint tanpa test | Rp 600 |

---

## 🌐 API Development Best Practices

### **1. Consistent Response Format**
```typescript
interface ApiResponse<T> {
  success: boolean;
  data?: T;
  error?: {
    code: string;
    message: string;
  };
}
```

### **2. Proper HTTP Status Codes**
- `200` - GET success
- `201` - POST success (created)
- `204` - DELETE success (no content)
- `400` - Bad request / validation error
- `401` - Unauthorized (not authenticated)
- `403` - Forbidden (not authorized)
- `404` - Not found
- `500` - Internal server error

### **3. Input Validation**
Always validate input dengan Joi/Zod sebelum processing

### **4. Error Handling**
Gunakan global error handler dan async wrapper

---

## 🗄️ Database Best Practices

### **1. Prevent N+1 Query**
Gunakan JOIN atau include untuk relasi

### **2. Use Transactions**
Untuk operasi multi-step yang harus atomic

### **3. SQL Injection Prevention**
Selalu gunakan parameterized query

---

## 🔐 Security Checklist

- [ ] JWT token di-verify (bukan decode)
- [ ] Password di-hash dengan bcrypt
- [ ] Input validation & sanitization
- [ ] CORS configured properly
- [ ] Rate limiting
- [ ] No credentials in code
- [ ] Authorization check (prevent IDOR)

---

## ✅ Backend Commit Checklist

### **API**
- [ ] Consistent response format
- [ ] Correct HTTP status codes
- [ ] Input validation
- [ ] Error handling

### **Database**
- [ ] No N+1 queries
- [ ] Use transactions
- [ ] Parameterized queries
- [ ] Pagination for lists

### **Security**
- [ ] Authentication & authorization
- [ ] Password hashing
- [ ] No SQL injection
- [ ] CORS & rate limiting

### **Architecture**
- [ ] Logic in service layer
- [ ] No circular dependencies
- [ ] Dependency injection

### **Testing**
- [ ] Unit tests for services
- [ ] Integration tests for APIs
- [ ] 80%+ coverage

---

Lihat file utama [README.md](../README.md) untuk detail lengkap setiap error level.
