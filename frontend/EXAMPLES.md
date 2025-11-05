# 🎨 Frontend Code Examples - Detail

Contoh lengkap implementasi frontend yang benar dan salah

---

## 📁 Project Structure (Recommended)

```
frontend/
├── src/
│   ├── components/
│   │   ├── common/
│   │   │   ├── Button.tsx
│   │   │   └── Input.tsx
│   │   └── features/
│   │       └── UserCard.tsx
│   ├── pages/
│   │   ├── Home.tsx
│   │   └── Dashboard.tsx
│   ├── hooks/
│   │   ├── useAuth.ts
│   │   └── useApi.ts
│   ├── services/
│   │   └── api.service.ts
│   ├── store/
│   │   └── userStore.ts
│   ├── types/
│   │   └── user.types.ts
│   ├── utils/
│   │   └── helpers.ts
│   ├── styles/
│   │   └── globals.css
│   └── App.tsx
├── tests/
│   ├── unit/
│   └── e2e/
├── .env.example
├── .eslintrc.json
├── tsconfig.json
└── package.json
```

---

## ⚛️ React Component Examples

### **1. Type-Safe Component**
```typescript
// src/types/user.types.ts
export interface User {
  id: string;
  name: string;
  email: string;
  role: 'admin' | 'user';
  avatar?: string;
}

// src/components/features/UserCard.tsx
import React from 'react';
import { User } from '../../types/user.types';

interface UserCardProps {
  user: User;
  onEdit?: (id: string) => void;
  onDelete?: (id: string) => void;
  isLoading?: boolean;
}

export const UserCard: React.FC<UserCardProps> = ({
  user,
  onEdit,
  onDelete,
  isLoading = false,
}) => {
  const handleEdit = () => {
    if (onEdit) {
      onEdit(user.id);
    }
  };

  const handleDelete = () => {
    if (onDelete && window.confirm('Are you sure?')) {
      onDelete(user.id);
    }
  };

  if (isLoading) {
    return <div className="skeleton">Loading...</div>;
  }

  return (
    <div className="user-card">
      {user.avatar && (
        <img
          src={user.avatar}
          alt={`${user.name}'s avatar`}
          loading="lazy"
        />
      )}
      <h3>{user.name}</h3>
      <p>{user.email}</p>
      <span className={`badge badge-${user.role}`}>{user.role}</span>

      <div className="actions">
        {onEdit && (
          <button onClick={handleEdit} aria-label="Edit user">
            Edit
          </button>
        )}
        {onDelete && (
          <button onClick={handleDelete} aria-label="Delete user">
            Delete
          </button>
        )}
      </div>
    </div>
  );
};
```

### **2. Custom Hooks**
```typescript
// src/hooks/useApi.ts
import { useState, useCallback } from 'react';

interface UseApiOptions<T> {
  onSuccess?: (data: T) => void;
  onError?: (error: Error) => void;
}

export function useApi<T = any>(
  apiFunc: (...args: any[]) => Promise<T>,
  options?: UseApiOptions<T>
) {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<Error | null>(null);

  const execute = useCallback(
    async (...args: any[]) => {
      try {
        setLoading(true);
        setError(null);

        const result = await apiFunc(...args);
        setData(result);

        if (options?.onSuccess) {
          options.onSuccess(result);
        }

        return result;
      } catch (err) {
        const error = err instanceof Error ? err : new Error('Unknown error');
        setError(error);

        if (options?.onError) {
          options.onError(error);
        }

        throw error;
      } finally {
        setLoading(false);
      }
    },
    [apiFunc, options]
  );

  const reset = useCallback(() => {
    setData(null);
    setError(null);
    setLoading(false);
  }, []);

  return { data, loading, error, execute, reset };
}

// Usage
function UserList() {
  const { data: users, loading, error, execute } = useApi(
    userService.getAll,
    {
      onSuccess: (data) => {
        console.log('Users loaded:', data.length);
      },
      onError: (error) => {
        console.error('Failed to load users:', error);
      },
    }
  );

  useEffect(() => {
    execute();
  }, [execute]);

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;

  return (
    <div>
      {users?.map(user => (
        <UserCard key={user.id} user={user} />
      ))}
    </div>
  );
}
```

### **3. Form with Validation**
```typescript
// src/components/features/UserForm.tsx
import React, { useState } from 'react';
import { z } from 'zod';

const userSchema = z.object({
  name: z.string().min(2, 'Name must be at least 2 characters'),
  email: z.string().email('Invalid email address'),
  password: z.string().min(8, 'Password must be at least 8 characters'),
  role: z.enum(['user', 'admin']),
});

type UserFormData = z.infer<typeof userSchema>;

interface UserFormProps {
  onSubmit: (data: UserFormData) => Promise<void>;
  initialData?: Partial<UserFormData>;
}

export const UserForm: React.FC<UserFormProps> = ({
  onSubmit,
  initialData,
}) => {
  const [formData, setFormData] = useState<UserFormData>({
    name: initialData?.name || '',
    email: initialData?.email || '',
    password: '',
    role: initialData?.role || 'user',
  });

  const [errors, setErrors] = useState<Record<string, string>>({});
  const [isSubmitting, setIsSubmitting] = useState(false);

  const handleChange = (
    e: React.ChangeEvent<HTMLInputElement | HTMLSelectElement>
  ) => {
    const { name, value } = e.target;
    setFormData(prev => ({ ...prev, [name]: value }));

    // Clear error for this field
    if (errors[name]) {
      setErrors(prev => {
        const newErrors = { ...prev };
        delete newErrors[name];
        return newErrors;
      });
    }
  };

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();

    try {
      // Validate
      const validatedData = userSchema.parse(formData);

      setIsSubmitting(true);
      await onSubmit(validatedData);

      // Reset form on success
      setFormData({
        name: '',
        email: '',
        password: '',
        role: 'user',
      });
      setErrors({});
    } catch (error) {
      if (error instanceof z.ZodError) {
        const newErrors: Record<string, string> = {};
        error.errors.forEach(err => {
          if (err.path[0]) {
            newErrors[err.path[0].toString()] = err.message;
          }
        });
        setErrors(newErrors);
      }
    } finally {
      setIsSubmitting(false);
    }
  };

  return (
    <form onSubmit={handleSubmit} className="user-form">
      <div className="form-group">
        <label htmlFor="name">Name</label>
        <input
          id="name"
          name="name"
          type="text"
          value={formData.name}
          onChange={handleChange}
          className={errors.name ? 'error' : ''}
          disabled={isSubmitting}
        />
        {errors.name && <span className="error-message">{errors.name}</span>}
      </div>

      <div className="form-group">
        <label htmlFor="email">Email</label>
        <input
          id="email"
          name="email"
          type="email"
          value={formData.email}
          onChange={handleChange}
          className={errors.email ? 'error' : ''}
          disabled={isSubmitting}
        />
        {errors.email && <span className="error-message">{errors.email}</span>}
      </div>

      <div className="form-group">
        <label htmlFor="password">Password</label>
        <input
          id="password"
          name="password"
          type="password"
          value={formData.password}
          onChange={handleChange}
          className={errors.password ? 'error' : ''}
          disabled={isSubmitting}
        />
        {errors.password && (
          <span className="error-message">{errors.password}</span>
        )}
      </div>

      <div className="form-group">
        <label htmlFor="role">Role</label>
        <select
          id="role"
          name="role"
          value={formData.role}
          onChange={handleChange}
          disabled={isSubmitting}
        >
          <option value="user">User</option>
          <option value="admin">Admin</option>
        </select>
      </div>

      <button type="submit" disabled={isSubmitting}>
        {isSubmitting ? 'Submitting...' : 'Submit'}
      </button>
    </form>
  );
};
```

### **4. API Service**
```typescript
// src/services/api.service.ts
import axios, { AxiosInstance, AxiosError } from 'axios';

interface ApiResponse<T> {
  success: boolean;
  data?: T;
  error?: {
    code: string;
    message: string;
  };
}

class ApiService {
  private api: AxiosInstance;

  constructor() {
    this.api = axios.create({
      baseURL: import.meta.env.VITE_API_URL || 'http://localhost:3000/api',
      timeout: 10000,
      headers: {
        'Content-Type': 'application/json',
      },
    });

    this.setupInterceptors();
  }

  private setupInterceptors() {
    // Request interceptor
    this.api.interceptors.request.use(
      (config) => {
        const token = localStorage.getItem('token');
        if (token) {
          config.headers.Authorization = `Bearer ${token}`;
        }
        return config;
      },
      (error) => {
        return Promise.reject(error);
      }
    );

    // Response interceptor
    this.api.interceptors.response.use(
      (response) => response.data,
      (error: AxiosError<ApiResponse<any>>) => {
        if (error.response?.status === 401) {
          // Token expired or invalid
          localStorage.removeItem('token');
          window.location.href = '/login';
        }

        const errorMessage =
          error.response?.data?.error?.message ||
          error.message ||
          'Something went wrong';

        return Promise.reject(new Error(errorMessage));
      }
    );
  }

  async get<T>(url: string, params?: any): Promise<T> {
    return this.api.get(url, { params });
  }

  async post<T>(url: string, data?: any): Promise<T> {
    return this.api.post(url, data);
  }

  async put<T>(url: string, data?: any): Promise<T> {
    return this.api.put(url, data);
  }

  async delete<T>(url: string): Promise<T> {
    return this.api.delete(url);
  }
}

export const apiService = new ApiService();

// User service
export const userService = {
  getAll: () => apiService.get<User[]>('/users'),
  getById: (id: string) => apiService.get<User>(`/users/${id}`),
  create: (data: Partial<User>) => apiService.post<User>('/users', data),
  update: (id: string, data: Partial<User>) =>
    apiService.put<User>(`/users/${id}`, data),
  delete: (id: string) => apiService.delete(`/users/${id}`),
};
```

### **5. State Management (Zustand)**
```typescript
// src/store/userStore.ts
import { create } from 'zustand';
import { persist } from 'zustand/middleware';
import { User } from '../types/user.types';

interface AuthState {
  user: User | null;
  token: string | null;
  isAuthenticated: boolean;
  login: (user: User, token: string) => void;
  logout: () => void;
  updateUser: (user: Partial<User>) => void;
}

export const useAuthStore = create<AuthState>()(
  persist(
    (set) => ({
      user: null,
      token: null,
      isAuthenticated: false,

      login: (user, token) => {
        localStorage.setItem('token', token);
        set({ user, token, isAuthenticated: true });
      },

      logout: () => {
        localStorage.removeItem('token');
        set({ user: null, token: null, isAuthenticated: false });
      },

      updateUser: (updatedUser) => {
        set((state) => ({
          user: state.user ? { ...state.user, ...updatedUser } : null,
        }));
      },
    }),
    {
      name: 'auth-storage',
      partialize: (state) => ({
        user: state.user,
        token: state.token,
        isAuthenticated: state.isAuthenticated,
      }),
    }
  )
);

// Usage
function Profile() {
  const { user, logout } = useAuthStore();

  if (!user) {
    return <div>Please login</div>;
  }

  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
      <button onClick={logout}>Logout</button>
    </div>
  );
}
```

### **6. Protected Route**
```typescript
// src/components/common/ProtectedRoute.tsx
import React from 'react';
import { Navigate, Outlet } from 'react-router-dom';
import { useAuthStore } from '../../store/userStore';

interface ProtectedRouteProps {
  allowedRoles?: string[];
}

export const ProtectedRoute: React.FC<ProtectedRouteProps> = ({
  allowedRoles,
}) => {
  const { isAuthenticated, user } = useAuthStore();

  if (!isAuthenticated) {
    return <Navigate to="/login" replace />;
  }

  if (allowedRoles && user && !allowedRoles.includes(user.role)) {
    return <Navigate to="/unauthorized" replace />;
  }

  return <Outlet />;
};

// Usage in App.tsx
function App() {
  return (
    <Routes>
      <Route path="/login" element={<Login />} />
      
      <Route element={<ProtectedRoute />}>
        <Route path="/dashboard" element={<Dashboard />} />
        <Route path="/profile" element={<Profile />} />
      </Route>

      <Route element={<ProtectedRoute allowedRoles={['admin']} />}>
        <Route path="/admin" element={<AdminPanel />} />
      </Route>
    </Routes>
  );
}
```

---

## 🧪 Testing Examples

### **Component Test**
```typescript
// tests/unit/UserCard.test.tsx
import { render, screen, fireEvent } from '@testing-library/react';
import { UserCard } from '../../src/components/features/UserCard';
import { User } from '../../src/types/user.types';

describe('UserCard', () => {
  const mockUser: User = {
    id: '1',
    name: 'John Doe',
    email: 'john@example.com',
    role: 'user',
  };

  it('should render user information', () => {
    render(<UserCard user={mockUser} />);

    expect(screen.getByText('John Doe')).toBeInTheDocument();
    expect(screen.getByText('john@example.com')).toBeInTheDocument();
    expect(screen.getByText('user')).toBeInTheDocument();
  });

  it('should call onEdit when edit button is clicked', () => {
    const onEdit = jest.fn();
    render(<UserCard user={mockUser} onEdit={onEdit} />);

    const editButton = screen.getByLabelText('Edit user');
    fireEvent.click(editButton);

    expect(onEdit).toHaveBeenCalledWith('1');
  });

  it('should show loading state', () => {
    render(<UserCard user={mockUser} isLoading={true} />);

    expect(screen.getByText('Loading...')).toBeInTheDocument();
    expect(screen.queryByText('John Doe')).not.toBeInTheDocument();
  });

  it('should not render edit button if onEdit is not provided', () => {
    render(<UserCard user={mockUser} />);

    expect(screen.queryByLabelText('Edit user')).not.toBeInTheDocument();
  });
});
```

### **Hook Test**
```typescript
// tests/unit/useApi.test.ts
import { renderHook, waitFor } from '@testing-library/react';
import { useApi } from '../../src/hooks/useApi';

describe('useApi', () => {
  it('should handle successful API call', async () => {
    const mockApiFunc = jest.fn().mockResolvedValue({ data: 'test' });
    const onSuccess = jest.fn();

    const { result } = renderHook(() =>
      useApi(mockApiFunc, { onSuccess })
    );

    expect(result.current.loading).toBe(false);
    expect(result.current.data).toBeNull();

    result.current.execute();

    expect(result.current.loading).toBe(true);

    await waitFor(() => {
      expect(result.current.loading).toBe(false);
      expect(result.current.data).toEqual({ data: 'test' });
      expect(onSuccess).toHaveBeenCalledWith({ data: 'test' });
    });
  });

  it('should handle API error', async () => {
    const mockApiFunc = jest.fn().mockRejectedValue(new Error('API Error'));
    const onError = jest.fn();

    const { result } = renderHook(() =>
      useApi(mockApiFunc, { onError })
    );

    result.current.execute();

    await waitFor(() => {
      expect(result.current.loading).toBe(false);
      expect(result.current.error).toEqual(new Error('API Error'));
      expect(onError).toHaveBeenCalled();
    });
  });
});
```

### **E2E Test (Playwright)**
```typescript
// tests/e2e/user-flow.spec.ts
import { test, expect } from '@playwright/test';

test.describe('User Management Flow', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('http://localhost:5173');
  });

  test('should login and view users', async ({ page }) => {
    // Login
    await page.fill('input[name="email"]', 'admin@example.com');
    await page.fill('input[name="password"]', 'password123');
    await page.click('button[type="submit"]');

    // Wait for redirect to dashboard
    await expect(page).toHaveURL(/.*dashboard/);

    // Navigate to users page
    await page.click('a[href="/users"]');

    // Check if users are loaded
    await expect(page.locator('.user-card')).toHaveCount(3);
  });

  test('should create new user', async ({ page }) => {
    // Assume already logged in
    await page.goto('http://localhost:5173/users/new');

    // Fill form
    await page.fill('input[name="name"]', 'New User');
    await page.fill('input[name="email"]', 'newuser@example.com');
    await page.fill('input[name="password"]', 'password123');
    await page.selectOption('select[name="role"]', 'user');

    // Submit
    await page.click('button[type="submit"]');

    // Check success message
    await expect(page.locator('.success-message')).toBeVisible();
  });
});
```

---

## 🎨 Performance Optimization Examples

### **1. Lazy Loading**
```typescript
import { lazy, Suspense } from 'react';
import { BrowserRouter, Routes, Route } from 'react-router-dom';

const Dashboard = lazy(() => import('./pages/Dashboard'));
const Profile = lazy(() => import('./pages/Profile'));
const Settings = lazy(() => import('./pages/Settings'));

function App() {
  return (
    <BrowserRouter>
      <Suspense fallback={<div>Loading...</div>}>
        <Routes>
          <Route path="/dashboard" element={<Dashboard />} />
          <Route path="/profile" element={<Profile />} />
          <Route path="/settings" element={<Settings />} />
        </Routes>
      </Suspense>
    </BrowserRouter>
  );
}
```

### **2. Memoization**
```typescript
import { useMemo, useCallback, memo } from 'react';

// Expensive calculation
function UserList({ users, filter }) {
  const filteredUsers = useMemo(() => {
    return users.filter(user =>
      user.name.toLowerCase().includes(filter.toLowerCase())
    );
  }, [users, filter]);

  const handleUserClick = useCallback((userId) => {
    console.log('User clicked:', userId);
  }, []);

  return (
    <div>
      {filteredUsers.map(user => (
        <UserCard
          key={user.id}
          user={user}
          onClick={handleUserClick}
        />
      ))}
    </div>
  );
}

// Memoized component
const UserCard = memo(({ user, onClick }) => {
  return (
    <div onClick={() => onClick(user.id)}>
      {user.name}
    </div>
  );
});
```

---

Lihat [README.md](./README.md) untuk checklist lengkap.
