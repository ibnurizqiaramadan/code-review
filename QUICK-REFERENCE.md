# 🚀 Code Review Quick Reference Card

Panduan cepat yang bisa di-print atau di-bookmark

---

## 💰 Tabel Denda


| Level | Denda | Kategori |
|:------|:------|:---------|
| **L1** | Rp 200 | Basic Logic / Syntax |
| **L2** | Rp 400 | Code Style |
| **L3** | Rp 800 | Logic Error |
| **L3.5** | Rp 1.000 | Error Handling |
| **L4** | Rp 1.200 | Data Validation |
| **L4.5** | Rp 1.400 | Code Duplication |
| **L5** | Rp 1.600 | Performance |
| **L6** | Rp 2.000 | Security |
| **L7** | Rp 2.500 | Architecture |
| **L8** | ×2 denda | Repeat Offense |
| **L9** | Rp 600 | Testing/Docs |

---

## ✅ Checklist Sebelum Commit

### **🔍 Code Quality (L1-L2)**
- [ ] No syntax error atau typo
- [ ] Gunakan `===` bukan `==`
- [ ] Consistent naming (camelCase/PascalCase)
- [ ] No magic numbers (gunakan konstanta)
- [ ] No dead code atau unused imports
- [ ] **TypeScript**: No `any` type

### **⚡ Logic & Error (L3-L3.5)**
- [ ] Async/await di-handle dengan benar
- [ ] No infinite loop atau race condition
- [ ] Semua error di-catch dan di-log
- [ ] Error message informatif
- [ ] Promise rejection di-handle

### **🛡️ Data & Validation (L4)**
- [ ] Input user di-validasi
- [ ] Null/undefined di-check
- [ ] Type checking untuk data penting
- [ ] No SQL injection (parameterized query)
- [ ] XSS prevention (sanitize input)

### **🔁 Code Quality (L4.5)**
- [ ] No duplicate logic
- [ ] Extract ke helper/utility function
- [ ] DRY principle (Don't Repeat Yourself)

### **🚀 Performance (L5)**
- [ ] No N+1 query (gunakan JOIN)
- [ ] Caching untuk data statis
- [ ] Pagination untuk list data
- [ ] Efficient data structure

### **🔐 Security (L6)**
- [ ] No hardcoded credentials
- [ ] Password di-hash (bcrypt/Hash)
- [ ] JWT token di-verify
- [ ] Authentication & authorization
- [ ] CORS configured
- [ ] Rate limiting

### **🏗️ Architecture (L7)**
- [ ] Logic di service layer, bukan controller
- [ ] No circular dependency
- [ ] Dependency injection
- [ ] Single Responsibility Principle
- [ ] Interface/abstraction

### **🧪 Testing & Docs (L9)**
- [ ] Unit test untuk service/logic
- [ ] Integration test untuk API
- [ ] Test coverage > 80%
- [ ] API documentation (Swagger/JSDoc)
- [ ] Complex logic ada comment

---

## 🔧 Backend Quick Checks

### **API Development**
```
✅ Consistent response format
✅ Correct HTTP status codes (200, 201, 204, 400, 401, 403, 404, 500)
✅ Input validation (Joi/Zod/class-validator)
✅ Error handling middleware
```

### **Database**
```
✅ No N+1 queries → Use JOIN/include
✅ Use transactions for multi-step operations
✅ Parameterized queries (prevent SQL injection)
✅ Database indexing
```

### **Security**
```
✅ JWT verify (not just decode)
✅ Password hashing (bcrypt/Hash)
✅ IDOR prevention (check ownership)
✅ Rate limiting
✅ CORS configuration
```

---

## 🎨 Frontend Quick Checks

### **React/Vue/Angular**
```
✅ TypeScript interfaces untuk props
✅ PropTypes/Zod validation
✅ No unused imports/variables
✅ Immutable state updates
✅ useEffect dependencies lengkap
✅ Cleanup di useEffect
```

### **Performance**
```
✅ Lazy loading untuk routes
✅ useMemo untuk expensive calculations
✅ useCallback untuk functions
✅ React.memo untuk pure components
✅ Image optimization
```

### **Security**
```
✅ No XSS (sanitize user input)
✅ No dangerouslySetInnerHTML tanpa sanitize
✅ No API keys in code
✅ Secure localStorage usage
```

---

## 🐛 Common Mistakes

### **Backend**
```
❌ Error diabaikan: data, _ := readFile()
❌ No transaction: save user, save order (bisa inconsistent)
❌ SQL injection: query = "SELECT * FROM users WHERE id = " + id
❌ Password plain text: user.password = req.body.password
❌ No auth check: app.delete('/users/:id') tanpa middleware
```

### **Frontend**
```
❌ Missing dependencies: useEffect(() => { fetch(userId) }, [])
❌ Mutate state: users.push(newUser)
❌ No cleanup: setInterval() tanpa clearInterval
❌ XSS: dangerouslySetInnerHTML={{ __html: userInput }}
❌ Props drilling: pass props 5 level deep
```

---

## 🛠️ Tools Setup

### **ESLint Rules (Must Have)**
```json
{
  "rules": {
    "eqeqeq": ["error", "always"],
    "no-unused-vars": ["error"],
    "no-console": ["warn"],
    "@typescript-eslint/no-explicit-any": "error"
  }
}
```

### **Pre-commit Hook**
```json
{
  "husky": {
    "hooks": {
      "pre-commit": "npm run lint && npm run test"
    }
  }
}
```

---

## 📊 Contoh Perhitungan Denda

### **Scenario 1: Simple PR**
```
- L1: Typo variabel → Rp 200
- L2: Magic number → Rp 400
Total: Rp 600
```

### **Scenario 2: Security Issue**
```
- L4: Input tidak divalidasi → Rp 1.200
- L6: Password tidak di-hash → Rp 2.000
Total: Rp 3.200
```

### **Scenario 3: Repeat Offense**
```
PR #100: Pakai == → Rp 200 (L1)
PR #105: Pakai == lagi → Rp 400 (L8: 200 × 2)
PR #110: Pakai == lagi → Rp 800 (L8: 400 × 2)
```

---

## 🎯 Priority Fix Order

1. **🔴 Critical (L6-L7)** - Security & Architecture
   - Fix immediately, block deployment
   
2. **🟠 High (L4-L5)** - Validation & Performance
   - Fix before merge
   
3. **🟡 Medium (L3-L3.5)** - Logic & Error Handling
   - Fix before merge
   
4. **🟢 Low (L1-L2)** - Style & Basic Logic
   - Can be fixed in next commit (but should be rare)

---

## 💡 Pro Tips

1. **Setup linter** → Auto-fix L1 & L2
2. **Use TypeScript** → Prevent type errors
3. **Write tests first** → TDD approach
4. **Code review checklist** → Before submit PR
5. **Pair programming** → Reduce errors
6. **Learn from mistakes** → Keep error log

---

## 📚 Resources

- **Main Guide**: [README.md](./README.md)
- **Summary**: [SUMMARY.md](./SUMMARY.md)
- **Backend**: [backend/README.md](./backend/README.md)
  - [Node.js](./backend/README.md)
  - [Golang](./backend/GOLANG.md)
  - [PHP](./backend/PHP.md)
- **Frontend**: [frontend/README.md](./frontend/README.md)

---

**Print this page and keep it near your desk! 📌**
