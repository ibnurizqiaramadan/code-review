# 🎨 Frontend Code Review Guidelines

Panduan khusus untuk code review frontend (React, Vue, Angular, Next.js)

---

## 📋 Frontend-Specific Error Levels

| Level | Jenis Kesalahan | Contoh Frontend | Denda |
|:------|:----------------|:----------------|:------|
| **L1** | Basic Logic | Typo props, salah import, undefined variable | Rp 200 |
| **L2** | Code Style | Inconsistent naming, magic values, unused imports | Rp 400 |
| **L3** | Logic Error | Infinite loop, wrong state update, memory leak | Rp 800 |
| **L3.5** | Error Handling | API error gak di-handle, try-catch kosong | Rp 1.000 |
| **L4** | Data Handling | Props validation, null check, type coercion | Rp 1.200 |
| **L5** | Performance | Re-render berlebihan, large bundle, no lazy loading | Rp 1.600 |
| **L6** | Security | XSS vulnerability, exposed API keys, localStorage misuse | Rp 2.000 |
| **L7** | Architecture | Logic di component, tight coupling, prop drilling | Rp 2.500 |
| **L9** | Testing/Docs | Component tanpa test, no PropTypes/interface | Rp 600 |

---

## ⚛️ React Best Practices

### **1. Component Structure**
```typescript
// ✅ Benar: Functional component dengan TypeScript
interface UserCardProps {
  user: {
    id: string;
    name: string;
    email: string;
  };
  onEdit: (id: string) => void;
}

export const UserCard: React.FC<UserCardProps> = ({ user, onEdit }) => {
  return (
    <div className="user-card">
      <h3>{user.name}</h3>
      <p>{user.email}</p>
      <button onClick={() => onEdit(user.id)}>Edit</button>
    </div>
  );
};
```

### **2. State Management**
```typescript
// ❌ Salah: Mutate state directly
const [users, setUsers] = useState([]);
users.push(newUser); // ❌

// ✅ Benar: Immutable update
setUsers([...users, newUser]);
setUsers(prev => [...prev, newUser]);
```

### **3. useEffect Dependencies**
```typescript
// ❌ Salah: Missing dependencies
useEffect(() => {
  fetchUser(userId);
}, []); // ❌ userId should be in deps

// ✅ Benar
useEffect(() => {
  fetchUser(userId);
}, [userId]);
```

### **4. Prevent Re-renders**
```typescript
// ✅ useMemo untuk expensive calculations
const expensiveValue = useMemo(() => {
  return computeExpensiveValue(data);
}, [data]);

// ✅ useCallback untuk functions
const handleClick = useCallback(() => {
  doSomething(id);
}, [id]);

// ✅ React.memo untuk component
export const UserCard = React.memo<UserCardProps>(({ user }) => {
  return <div>{user.name}</div>;
});
```

---

## 🎯 Common Frontend Errors

### **Case 1: XSS Vulnerability**
```typescript
// ❌ Salah: dangerouslySetInnerHTML
<div dangerouslySetInnerHTML={{ __html: userInput }} />

// ✅ Benar: Sanitize input
import DOMPurify from 'dompurify';

<div dangerouslySetInnerHTML={{ 
  __html: DOMPurify.sanitize(userInput) 
}} />

// Atau lebih baik: hindari innerHTML
<div>{userInput}</div>
```

### **Case 2: API Error Handling**
```typescript
// ❌ Salah: No error handling
const fetchUsers = async () => {
  const response = await fetch('/api/users');
  const data = await response.json();
  setUsers(data);
};

// ✅ Benar
const fetchUsers = async () => {
  try {
    setLoading(true);
    setError(null);
    
    const response = await fetch('/api/users');
    
    if (!response.ok) {
      throw new Error(`HTTP ${response.status}: ${response.statusText}`);
    }
    
    const data = await response.json();
    setUsers(data);
  } catch (error) {
    setError(error.message);
    console.error('Failed to fetch users:', error);
  } finally {
    setLoading(false);
  }
};
```

### **Case 3: Memory Leak**
```typescript
// ❌ Salah: No cleanup
useEffect(() => {
  const interval = setInterval(() => {
    fetchData();
  }, 5000);
}, []);

// ✅ Benar: Cleanup
useEffect(() => {
  const interval = setInterval(() => {
    fetchData();
  }, 5000);
  
  return () => clearInterval(interval);
}, []);
```

### **Case 4: Props Drilling**
```typescript
// ❌ Salah: Props drilling
<GrandParent>
  <Parent user={user}>
    <Child user={user}>
      <GrandChild user={user} />
    </Child>
  </Parent>
</GrandParent>

// ✅ Benar: Context API
const UserContext = createContext<User | null>(null);

export const UserProvider: React.FC = ({ children }) => {
  const [user, setUser] = useState<User | null>(null);
  
  return (
    <UserContext.Provider value={user}>
      {children}
    </UserContext.Provider>
  );
};

// Usage
const GrandChild = () => {
  const user = useContext(UserContext);
  return <div>{user?.name}</div>;
};
```

---

## 🚀 Performance Optimization

### **1. Code Splitting & Lazy Loading**
```typescript
// ✅ Lazy load routes
const Dashboard = lazy(() => import('./pages/Dashboard'));
const Profile = lazy(() => import('./pages/Profile'));

<Suspense fallback={<Loading />}>
  <Routes>
    <Route path="/dashboard" element={<Dashboard />} />
    <Route path="/profile" element={<Profile />} />
  </Routes>
</Suspense>
```

### **2. Image Optimization**
```typescript
// ✅ Next.js Image component
import Image from 'next/image';

<Image
  src="/profile.jpg"
  alt="Profile"
  width={200}
  height={200}
  loading="lazy"
/>
```

### **3. Virtualization untuk Long Lists**
```typescript
import { FixedSizeList } from 'react-window';

<FixedSizeList
  height={600}
  itemCount={items.length}
  itemSize={50}
  width="100%"
>
  {({ index, style }) => (
    <div style={style}>{items[index].name}</div>
  )}
</FixedSizeList>
```

---

## ✅ Frontend Commit Checklist

### **Component**
- [ ] TypeScript interfaces untuk props
- [ ] PropTypes atau Zod validation
- [ ] No unused imports/variables
- [ ] Consistent naming (PascalCase untuk components)
- [ ] No inline styles (gunakan CSS modules/Tailwind)

### **State Management**
- [ ] Immutable state updates
- [ ] useEffect dependencies lengkap
- [ ] Cleanup di useEffect
- [ ] No unnecessary re-renders

### **API & Data**
- [ ] Error handling untuk API calls
- [ ] Loading states
- [ ] Null/undefined checks
- [ ] Type-safe API responses

### **Performance**
- [ ] Lazy loading untuk routes
- [ ] useMemo/useCallback untuk optimization
- [ ] React.memo untuk pure components
- [ ] Image optimization
- [ ] Bundle size check

### **Security**
- [ ] No XSS vulnerability
- [ ] Sanitize user input
- [ ] No API keys in code
- [ ] Secure localStorage usage
- [ ] HTTPS only

### **Accessibility**
- [ ] Semantic HTML
- [ ] Alt text untuk images
- [ ] Keyboard navigation
- [ ] ARIA labels
- [ ] Color contrast

### **Testing**
- [ ] Unit tests untuk utils/hooks
- [ ] Component tests dengan React Testing Library
- [ ] E2E tests untuk critical flows

---

## 🛠️ Recommended Tools

### **Development**
- **TypeScript** - Type safety
- **ESLint + Prettier** - Code quality
- **Vite** - Fast build tool

### **State Management**
- **Zustand** - Lightweight
- **Redux Toolkit** - Complex apps
- **TanStack Query** - Server state

### **Testing**
- **Vitest** - Unit tests
- **React Testing Library** - Component tests
- **Playwright** - E2E tests

### **UI Libraries**
- **Tailwind CSS** - Utility-first CSS
- **shadcn/ui** - Component library
- **Lucide** - Icons

---

Lihat file utama [README.md](../README.md) untuk detail lengkap setiap error level.
