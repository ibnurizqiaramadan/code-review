# 🚀 Getting Started - Code Review Guidelines

Panduan untuk memulai menggunakan sistem code review ini

---

## 📖 Untuk Developer Baru

### **Step 1: Pahami Sistem Denda**

Sistem ini menggunakan denda sebagai **motivasi** untuk menulis code berkualitas, bukan untuk menghukum.

**Tujuan:**
- ✅ Meningkatkan code quality
- ✅ Reduce technical debt
- ✅ Build learning culture
- ✅ Consistency di seluruh tim
- ✅ Accountability

### **Step 2: Pilih Panduan Sesuai Role Anda**

#### **Backend Developer:**

**Node.js / TypeScript:**
1. Baca [backend/README.md](./backend/README.md)
2. Lihat contoh: [backend/EXAMPLES.md](./backend/EXAMPLES.md)
3. Bookmark [QUICK-REFERENCE.md](./QUICK-REFERENCE.md)

**Golang:**
1. Baca [backend/GOLANG.md](./backend/GOLANG.md)
2. Setup golangci-lint
3. Pelajari goroutine management

**PHP / Laravel:**
1. Baca [backend/PHP.md](./backend/PHP.md)
2. Lihat contoh: [backend/PHP-LARAVEL-EXAMPLE.md](./backend/PHP-LARAVEL-EXAMPLE.md)
3. Setup PHP CS Fixer & PHPStan

#### **Frontend Developer:**

1. Baca [frontend/README.md](./frontend/README.md)
2. Lihat contoh: [frontend/EXAMPLES.md](./frontend/EXAMPLES.md)
3. Setup ESLint + TypeScript
4. Bookmark [QUICK-REFERENCE.md](./QUICK-REFERENCE.md)

### **Step 3: Setup Development Environment**

#### **Install Linter & Formatter**

**Node.js / TypeScript:**
```bash
npm install --save-dev eslint prettier @typescript-eslint/parser @typescript-eslint/eslint-plugin
npm install --save-dev husky lint-staged
```

**Golang:**
```bash
go install github.com/golangci/golangci-lint/cmd/golangci-lint@latest
```

**PHP:**
```bash
composer require --dev friendsofphp/php-cs-fixer
composer require --dev phpstan/phpstan
```

#### **Setup Pre-commit Hook**

**package.json (Node.js):**
```json
{
  "scripts": {
    "lint": "eslint . --ext .ts,.tsx,.js,.jsx",
    "format": "prettier --write .",
    "test": "jest"
  },
  "husky": {
    "hooks": {
      "pre-commit": "npm run lint && npm run test"
    }
  }
}
```

### **Step 4: Pelajari Error Levels**

**Prioritas Belajar:**

**Minggu 1-2: Basic (L1-L2)**
- ✅ No syntax error
- ✅ Consistent naming
- ✅ No magic numbers
- ✅ Code formatting

**Minggu 3-4: Logic & Error (L3-L3.5)**
- ✅ Proper async/await
- ✅ Error handling
- ✅ Null checks

**Minggu 5-6: Validation & Security (L4-L6)**
- ✅ Input validation
- ✅ SQL injection prevention
- ✅ Password hashing
- ✅ Authentication

**Minggu 7-8: Architecture & Testing (L7-L9)**
- ✅ Service layer pattern
- ✅ Dependency injection
- ✅ Unit testing
- ✅ Documentation

---

## 👥 Untuk Tech Lead / Reviewer

### **Setup Code Review Process**

#### **1. Create Review Template**

**GitHub PR Template (.github/pull_request_template.md):**
```markdown
## Description
<!-- Describe your changes -->

## Type of Change
- [ ] Bug fix
- [ ] New feature
- [ ] Breaking change
- [ ] Documentation update

## Checklist
- [ ] Code follows style guidelines
- [ ] Self-review completed
- [ ] Comments added for complex logic
- [ ] Tests added/updated
- [ ] Documentation updated
- [ ] No console.log/dd() left
- [ ] All tests passing

## Error Level Check
- [ ] No L1-L2 errors (basic logic & style)
- [ ] No L3-L3.5 errors (logic & error handling)
- [ ] No L4-L6 errors (validation & security)
- [ ] No L7 errors (architecture)
- [ ] Tests added (L9)
```

#### **2. Setup CI/CD Pipeline**

**GitHub Actions (.github/workflows/ci.yml):**
```yaml
name: CI

on: [push, pull_request]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: actions/setup-node@v2
      - run: npm install
      - run: npm run lint

  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: actions/setup-node@v2
      - run: npm install
      - run: npm test
      - run: npm run test:coverage
```

#### **3. Review Guidelines**

**Saat Review PR:**

1. **Check Automated Tests First**
   - ✅ Linter passing?
   - ✅ Tests passing?
   - ✅ Coverage > 80%?

2. **Manual Review Checklist**
   - [ ] Code readable & maintainable?
   - [ ] Logic correct?
   - [ ] Error handling proper?
   - [ ] Security vulnerabilities?
   - [ ] Performance issues?
   - [ ] Tests adequate?

3. **Give Constructive Feedback**
   ```
   ❌ Bad: "This code is bad"
   ✅ Good: "Consider extracting this logic to a service layer (L7). 
            This will make it easier to test and reuse."
   ```

4. **Categorize Errors**
   ```
   🔴 Blocker (L6-L7): Must fix before merge
   🟠 Important (L4-L5): Should fix before merge
   🟡 Nice to have (L1-L3): Can fix in next commit
   ```

---

## 📊 Tracking & Metrics

### **Setup Denda Tracker**

**Google Sheets / Excel:**
```
| Date | Developer | PR # | Level | Error | Denda | Status |
|------|-----------|------|-------|-------|-------|--------|
| 2024-01-15 | John | #123 | L2 | Magic number | 400 | Paid |
| 2024-01-16 | Jane | #124 | L6 | No auth | 2000 | Paid |
```

### **Monthly Report**

**Metrics to Track:**
- Total denda per developer
- Most common error types
- Improvement trend
- Code quality score

**Example Report:**
```
📊 Code Review Report - January 2024

Total PRs: 50
Total Errors: 25
Total Denda: Rp 250.000

Top Errors:
1. L2 (Code Style): 10 occurrences
2. L4 (Validation): 6 occurrences
3. L3 (Logic Error): 5 occurrences

Top Contributors (Most Improved):
1. John Doe: -50% errors vs last month
2. Jane Smith: -30% errors vs last month

Action Items:
- [ ] Team training on validation (L4)
- [ ] Update ESLint config for L2 errors
- [ ] Code review workshop
```

---

## 🎓 Training Program

### **Monthly Code Review Workshop**

**Agenda:**
1. Review common errors from last month
2. Live code review session
3. Best practices sharing
4. Q&A

**Topics:**
- Month 1: Basic Logic & Code Style (L1-L2)
- Month 2: Error Handling (L3.5)
- Month 3: Security Best Practices (L6)
- Month 4: Architecture Patterns (L7)

### **Pair Programming Sessions**

**Schedule:**
- Junior + Senior: 2 hours/week
- Focus on real PRs
- Live debugging & refactoring

---

## 💡 Tips for Success

### **For Developers:**

1. **Use Checklist** - Print [QUICK-REFERENCE.md](./QUICK-REFERENCE.md)
2. **Self-Review First** - Review your own code before submitting
3. **Ask Questions** - Better to ask than to guess
4. **Learn from Mistakes** - Keep a personal error log
5. **Automate** - Setup linter & pre-commit hooks

### **For Reviewers:**

1. **Be Constructive** - Explain why, not just what
2. **Be Consistent** - Apply same standards to everyone
3. **Be Timely** - Review within 24 hours
4. **Be Thorough** - Don't just approve quickly
5. **Be Encouraging** - Acknowledge good code too

---

## 🆘 FAQ

### **Q: Apakah denda benar-benar dibayar?**
A: Tergantung kesepakatan tim. Bisa real money, atau symbolic (coffee, snacks, dll).

### **Q: Bagaimana dengan junior developer?**
A: Grace period 2 minggu (warning only), minggu ke-3 denda 50%, minggu ke-4+ full.

### **Q: Bagaimana jika tidak setuju dengan denda?**
A: Diskusi dengan tech lead. Bawa evidence. Keputusan tech lead final.

### **Q: Apakah reviewer juga kena denda?**
A: Ya, 30% dari denda developer jika miss critical error (L6-L7).

### **Q: Bagaimana dengan legacy code?**
A: Denda hanya untuk code baru. Legacy code yang di-refactor diberi kelonggaran.

---

## 📞 Support

**Questions?**
- Ask in #code-review Slack channel
- Tag @tech-lead in PR comments
- Schedule 1-on-1 with senior developer

**Resources:**
- [Main Guide](./README.md)
- [Summary](./SUMMARY.md)
- [Quick Reference](./QUICK-REFERENCE.md)

---

**Welcome to the team! Let's write better code together! 🚀**
