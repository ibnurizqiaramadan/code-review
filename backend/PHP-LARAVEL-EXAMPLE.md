# 🐘 Laravel Complete Example

Contoh lengkap implementasi Laravel dengan best practices

---

## 📁 Project Structure

```
app/
├── Http/
│   ├── Controllers/
│   │   └── Api/
│   │       └── UserController.php
│   ├── Requests/
│   │   ├── CreateUserRequest.php
│   │   └── UpdateUserRequest.php
│   ├── Resources/
│   │   └── UserResource.php
│   └── Middleware/
│       └── CheckRole.php
├── Services/
│   └── UserService.php
├── Repositories/
│   ├── UserRepository.php
│   └── Eloquent/
│       └── EloquentUserRepository.php
├── Models/
│   └── User.php
├── Exceptions/
│   ├── UserNotFoundException.php
│   └── DuplicateEmailException.php
└── Events/
    └── UserRegistered.php
```

---

## 📝 Complete Implementation

### **1. Model**
```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;
use Laravel\Sanctum\HasApiTokens;

class User extends Authenticatable
{
    use HasApiTokens, HasFactory, Notifiable;

    protected $fillable = [
        'name',
        'email',
        'password',
        'role',
    ];

    protected $hidden = [
        'password',
        'remember_token',
    ];

    protected $casts = [
        'email_verified_at' => 'datetime',
        'password' => 'hashed',
    ];

    // Relationships
    public function orders()
    {
        return $this->hasMany(Order::class);
    }

    // Scopes
    public function scopeActive($query)
    {
        return $query->where('is_active', true);
    }

    public function scopeRole($query, string $role)
    {
        return $query->where('role', $role);
    }
}
```

### **2. Repository Interface**
```php
<?php

namespace App\Repositories;

use App\Models\User;
use Illuminate\Pagination\LengthAwarePaginator;

interface UserRepository
{
    public function create(array $data): User;
    public function findById(int $id): ?User;
    public function findByEmail(string $email): ?User;
    public function existsByEmail(string $email): bool;
    public function update(int $id, array $data): User;
    public function delete(int $id): bool;
    public function paginate(int $perPage = 15): LengthAwarePaginator;
}
```

### **3. Repository Implementation**
```php
<?php

namespace App\Repositories\Eloquent;

use App\Models\User;
use App\Repositories\UserRepository;
use Illuminate\Pagination\LengthAwarePaginator;

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

    public function findByEmail(string $email): ?User
    {
        return User::where('email', $email)->first();
    }

    public function existsByEmail(string $email): bool
    {
        return User::where('email', $email)->exists();
    }

    public function update(int $id, array $data): User
    {
        $user = User::findOrFail($id);
        $user->update($data);
        return $user->fresh();
    }

    public function delete(int $id): bool
    {
        $user = User::findOrFail($id);
        return $user->delete();
    }

    public function paginate(int $perPage = 15): LengthAwarePaginator
    {
        return User::latest()->paginate($perPage);
    }
}
```

### **4. Service Layer**
```php
<?php

namespace App\Services;

use App\Events\UserRegistered;
use App\Exceptions\DuplicateEmailException;
use App\Exceptions\UserNotFoundException;
use App\Models\User;
use App\Repositories\UserRepository;
use Illuminate\Pagination\LengthAwarePaginator;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Hash;

class UserService
{
    public function __construct(
        private UserRepository $userRepository
    ) {}

    public function create(array $data): User
    {
        if ($this->userRepository->existsByEmail($data['email'])) {
            throw new DuplicateEmailException('Email already registered');
        }

        return DB::transaction(function () use ($data) {
            $data['password'] = Hash::make($data['password']);
            $data['role'] = $data['role'] ?? 'user';

            $user = $this->userRepository->create($data);

            event(new UserRegistered($user));

            return $user;
        });
    }

    public function getById(int $id): User
    {
        $user = $this->userRepository->findById($id);

        if (!$user) {
            throw new UserNotFoundException("User not found with ID: {$id}");
        }

        return $user;
    }

    public function getAll(int $perPage = 15): LengthAwarePaginator
    {
        return $this->userRepository->paginate($perPage);
    }

    public function update(int $id, array $data): User
    {
        $user = $this->getById($id);

        if (isset($data['email']) && $data['email'] !== $user->email) {
            if ($this->userRepository->existsByEmail($data['email'])) {
                throw new DuplicateEmailException('Email already in use');
            }
        }

        if (isset($data['password'])) {
            $data['password'] = Hash::make($data['password']);
        }

        return $this->userRepository->update($id, $data);
    }

    public function delete(int $id): bool
    {
        $this->getById($id); // Check if exists
        return $this->userRepository->delete($id);
    }
}
```

### **5. Form Requests**
```php
<?php

namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;

class CreateUserRequest extends FormRequest
{
    public function authorize(): bool
    {
        return true;
    }

    public function rules(): array
    {
        return [
            'name' => ['required', 'string', 'min:2', 'max:100'],
            'email' => ['required', 'email', 'unique:users,email'],
            'password' => ['required', 'string', 'min:8', 'confirmed'],
            'role' => ['nullable', 'in:user,admin'],
        ];
    }

    public function messages(): array
    {
        return [
            'email.unique' => 'Email already registered',
            'password.min' => 'Password must be at least 8 characters',
            'password.confirmed' => 'Password confirmation does not match',
        ];
    }
}

// UpdateUserRequest.php
class UpdateUserRequest extends FormRequest
{
    public function authorize(): bool
    {
        return true;
    }

    public function rules(): array
    {
        $userId = $this->route('user');

        return [
            'name' => ['sometimes', 'string', 'min:2', 'max:100'],
            'email' => ['sometimes', 'email', "unique:users,email,{$userId}"],
            'password' => ['sometimes', 'string', 'min:8', 'confirmed'],
            'role' => ['sometimes', 'in:user,admin'],
        ];
    }
}
```

### **6. API Resource**
```php
<?php

namespace App\Http\Resources;

use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\JsonResource;

class UserResource extends JsonResource
{
    public function toArray(Request $request): array
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'email' => $this->email,
            'role' => $this->role,
            'created_at' => $this->created_at->toIso8601String(),
            'updated_at' => $this->updated_at->toIso8601String(),
        ];
    }
}
```

### **7. Controller**
```php
<?php

namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use App\Http\Requests\CreateUserRequest;
use App\Http\Requests\UpdateUserRequest;
use App\Http\Resources\UserResource;
use App\Services\UserService;
use Illuminate\Http\JsonResponse;
use Illuminate\Http\Resources\Json\AnonymousResourceCollection;

class UserController extends Controller
{
    public function __construct(
        private UserService $userService
    ) {}

    public function index(): AnonymousResourceCollection
    {
        $users = $this->userService->getAll(
            perPage: request()->get('per_page', 15)
        );

        return UserResource::collection($users);
    }

    public function store(CreateUserRequest $request): JsonResponse
    {
        $user = $this->userService->create($request->validated());

        return response()->json([
            'success' => true,
            'data' => new UserResource($user),
        ], 201);
    }

    public function show(int $id): JsonResponse
    {
        $user = $this->userService->getById($id);

        return response()->json([
            'success' => true,
            'data' => new UserResource($user),
        ]);
    }

    public function update(UpdateUserRequest $request, int $id): JsonResponse
    {
        $user = $this->userService->update($id, $request->validated());

        return response()->json([
            'success' => true,
            'data' => new UserResource($user),
        ]);
    }

    public function destroy(int $id): JsonResponse
    {
        $this->userService->delete($id);

        return response()->json([
            'success' => true,
            'message' => 'User deleted successfully',
        ], 204);
    }
}
```

### **8. Custom Exceptions**
```php
<?php

namespace App\Exceptions;

use Exception;

class UserNotFoundException extends Exception
{
    protected $code = 404;
    protected $message = 'User not found';
}

class DuplicateEmailException extends Exception
{
    protected $code = 409;
    protected $message = 'Email already exists';
}
```

### **9. Exception Handler**
```php
<?php

namespace App\Exceptions;

use Illuminate\Foundation\Exceptions\Handler as ExceptionHandler;
use Illuminate\Validation\ValidationException;
use Symfony\Component\HttpKernel\Exception\NotFoundHttpException;
use Throwable;

class Handler extends ExceptionHandler
{
    public function render($request, Throwable $e)
    {
        if ($request->expectsJson()) {
            if ($e instanceof ValidationException) {
                return response()->json([
                    'success' => false,
                    'error' => [
                        'code' => 'VALIDATION_ERROR',
                        'message' => 'Validation failed',
                        'details' => $e->errors(),
                    ],
                ], 422);
            }

            if ($e instanceof UserNotFoundException) {
                return response()->json([
                    'success' => false,
                    'error' => [
                        'code' => 'USER_NOT_FOUND',
                        'message' => $e->getMessage(),
                    ],
                ], 404);
            }

            if ($e instanceof DuplicateEmailException) {
                return response()->json([
                    'success' => false,
                    'error' => [
                        'code' => 'DUPLICATE_EMAIL',
                        'message' => $e->getMessage(),
                    ],
                ], 409);
            }

            if ($e instanceof NotFoundHttpException) {
                return response()->json([
                    'success' => false,
                    'error' => [
                        'code' => 'NOT_FOUND',
                        'message' => 'Resource not found',
                    ],
                ], 404);
            }
        }

        return parent::render($request, $e);
    }
}
```

### **10. Routes**
```php
<?php

use App\Http\Controllers\Api\UserController;
use Illuminate\Support\Facades\Route;

Route::prefix('v1')->group(function () {
    Route::post('/register', [UserController::class, 'store']);
    
    Route::middleware('auth:sanctum')->group(function () {
        Route::apiResource('users', UserController::class)->except(['store']);
    });
});
```

### **11. Service Provider (Bind Repository)**
```php
<?php

namespace App\Providers;

use App\Repositories\Eloquent\EloquentUserRepository;
use App\Repositories\UserRepository;
use Illuminate\Support\ServiceProvider;

class RepositoryServiceProvider extends ServiceProvider
{
    public function register(): void
    {
        $this->app->bind(
            UserRepository::class,
            EloquentUserRepository::class
        );
    }
}
```

---

## 🧪 Testing

### **Feature Test**
```php
<?php

namespace Tests\Feature;

use App\Models\User;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Tests\TestCase;

class UserApiTest extends TestCase
{
    use RefreshDatabase;

    public function test_can_create_user(): void
    {
        $response = $this->postJson('/api/v1/register', [
            'name' => 'John Doe',
            'email' => 'john@example.com',
            'password' => 'password123',
            'password_confirmation' => 'password123',
        ]);

        $response->assertStatus(201)
            ->assertJsonStructure([
                'success',
                'data' => [
                    'id',
                    'name',
                    'email',
                    'role',
                ],
            ]);

        $this->assertDatabaseHas('users', [
            'email' => 'john@example.com',
        ]);
    }

    public function test_cannot_create_user_with_duplicate_email(): void
    {
        User::factory()->create(['email' => 'john@example.com']);

        $response = $this->postJson('/api/v1/register', [
            'name' => 'John Doe',
            'email' => 'john@example.com',
            'password' => 'password123',
            'password_confirmation' => 'password123',
        ]);

        $response->assertStatus(409)
            ->assertJson([
                'success' => false,
                'error' => [
                    'code' => 'DUPLICATE_EMAIL',
                ],
            ]);
    }
}
```

### **Unit Test**
```php
<?php

namespace Tests\Unit;

use App\Exceptions\DuplicateEmailException;
use App\Models\User;
use App\Repositories\UserRepository;
use App\Services\UserService;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Mockery;
use Tests\TestCase;

class UserServiceTest extends TestCase
{
    use RefreshDatabase;

    public function test_create_user_successfully(): void
    {
        $repo = Mockery::mock(UserRepository::class);
        $repo->shouldReceive('existsByEmail')->andReturn(false);
        $repo->shouldReceive('create')->andReturn(new User([
            'id' => 1,
            'name' => 'John',
            'email' => 'john@example.com',
        ]));

        $service = new UserService($repo);
        $user = $service->create([
            'name' => 'John',
            'email' => 'john@example.com',
            'password' => 'password123',
        ]);

        $this->assertEquals('John', $user->name);
    }

    public function test_create_user_throws_exception_for_duplicate_email(): void
    {
        $repo = Mockery::mock(UserRepository::class);
        $repo->shouldReceive('existsByEmail')->andReturn(true);

        $service = new UserService($repo);

        $this->expectException(DuplicateEmailException::class);

        $service->create([
            'name' => 'John',
            'email' => 'john@example.com',
            'password' => 'password123',
        ]);
    }
}
```

---

Lihat [PHP.md](./PHP.md) untuk checklist lengkap.
