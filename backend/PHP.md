# 🐘 PHP Backend Code Review Guidelines

Panduan khusus untuk code review PHP backend (Laravel, Symfony, CodeIgniter)

---

## 📋 PHP-Specific Error Levels

| Level | Jenis Kesalahan | Contoh PHP | Denda |
|:------|:----------------|:-----------|:------|
| **L1** | Basic Logic | Typo variable, syntax error, undefined variable | Rp 200 |
| **L2** | Code Style | PSR-12 violation, inconsistent naming | Rp 400 |
| **L3** | Logic Error | Wrong comparison, type juggling | Rp 800 |
| **L3.5** | Error Handling | Exception tidak di-catch, error suppression | Rp 1.000 |
| **L4** | Data Validation | Input tidak divalidasi, SQL injection | Rp 1.200 |
| **L5** | Performance | N+1 query, no caching | Rp 1.600 |
| **L6** | Security | XSS, CSRF, no password hash | Rp 2.000 |
| **L7** | Architecture | Logic di controller, no SOLID | Rp 2.500 |
| **L9** | Testing/Docs | No unit test, API tanpa docs | Rp 600 |

---

## 🎯 PHP Best Practices

### **1. Type Safety (PHP 8+)**
```php
declare(strict_types=1);

function calculateTotal(array $items): float {
    $total = 0.0;
    foreach ($items as $item) {
        $total += (float)$item['price'] * (int)$item['quantity'];
    }
    return $total;
}

// Strict comparison
if ($userId === 1) { // ✅ Not ==
    // ...
}
```

### **2. Laravel Best Practices**

#### **Controller (Thin)**
```php
class UserController extends Controller
{
    public function __construct(
        private UserService $userService
    ) {}

    public function store(CreateUserRequest $request): JsonResponse
    {
        $user = $this->userService->create($request->validated());
        
        return response()->json([
            'success' => true,
            'data' => new UserResource($user)
        ], 201);
    }
}
```

#### **Service Layer (Business Logic)**
```php
class UserService
{
    public function __construct(
        private UserRepository $userRepository
    ) {}

    public function create(array $data): User
    {
        if ($this->userRepository->existsByEmail($data['email'])) {
            throw new DuplicateEmailException();
        }

        $data['password'] = Hash::make($data['password']);
        
        return DB::transaction(function () use ($data) {
            $user = $this->userRepository->create($data);
            event(new UserRegistered($user));
            return $user;
        });
    }
}
```

#### **Repository Pattern**
```php
interface UserRepository
{
    public function create(array $data): User;
    public function findById(int $id): ?User;
    public function existsByEmail(string $email): bool;
}

class EloquentUserRepository implements UserRepository
{
    public function create(array $data): User
    {
        return User::create($data);
    }

    public function findById(int $id): ?User
    {
        return User::find($id);
    }

    public function existsByEmail(string $email): bool
    {
        return User::where('email', $email)->exists();
    }
}
```

### **3. Validation**
```php
class CreateUserRequest extends FormRequest
{
    public function rules(): array
    {
        return [
            'email' => ['required', 'email', 'unique:users,email'],
            'name' => ['required', 'string', 'min:2', 'max:100'],
            'password' => ['required', 'string', 'min:8', 'confirmed'],
            'role' => ['nullable', 'in:user,admin'],
        ];
    }

    public function messages(): array
    {
        return [
            'email.unique' => 'Email already registered',
            'password.min' => 'Password must be at least 8 characters',
        ];
    }
}
```

### **4. Security**
```php
// Password hashing
$hashedPassword = Hash::make($password);

// XSS Prevention
{{ $user->name }} // Auto-escaped in Blade
{!! $html !!} // Unescaped - use with caution

// SQL Injection Prevention
User::where('email', $email)->first(); // ✅ Query builder
DB::select('SELECT * FROM users WHERE email = ?', [$email]); // ✅ Prepared

// CSRF Protection (automatic in Laravel)
<form method="POST">
    @csrf
</form>
```

---

## ✅ PHP Checklist

### **Code Quality**
- [ ] `declare(strict_types=1)` di setiap file
- [ ] Type hints untuk parameters & return types
- [ ] PSR-12 code style
- [ ] No error suppression (@)
- [ ] Strict comparison (===)

### **Laravel Specific**
- [ ] Logic di service layer, bukan controller
- [ ] Repository pattern untuk data access
- [ ] Form Request untuk validation
- [ ] Resource classes untuk API response
- [ ] Use transactions untuk multi-step operations

### **Security**
- [ ] Password hashing (Hash::make)
- [ ] Input validation
- [ ] CSRF protection
- [ ] XSS prevention (Blade escaping)
- [ ] SQL injection prevention (Query Builder)

### **Performance**
- [ ] Eager loading (with()) untuk prevent N+1
- [ ] Caching untuk data statis
- [ ] Queue untuk long-running tasks
- [ ] Database indexing

### **Testing**
- [ ] Feature tests untuk API endpoints
- [ ] Unit tests untuk services
- [ ] Test coverage > 80%

---

Lihat [README.md](./README.md) untuk panduan umum backend.
