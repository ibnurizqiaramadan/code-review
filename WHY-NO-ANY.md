# 🚫 Why No `any` Type in TypeScript

Penjelasan lengkap kenapa `any` type harus dihindari dan alternatifnya

---

## ❌ Masalah dengan `any` Type

### **1. Kehilangan Type Safety**
```typescript
// ❌ Dengan any - tidak ada type checking
function processData(data: any) {
  return data.name.toUpperCase(); // Tidak ada error jika data.name undefined
}

processData({ age: 25 }); // Runtime error! ❌

// ✅ Dengan proper type - type checking berfungsi
interface User {
  name: string;
  age: number;
}

function processData(data: User) {
  return data.name.toUpperCase(); // ✅ Type-safe
}

// processData({ age: 25 }); // ❌ Compile error - missing 'name'
```

### **2. Tidak Ada Autocomplete**
```typescript
// ❌ Dengan any
const user: any = getUser();
user. // Tidak ada autocomplete, tidak tahu property apa yang tersedia

// ✅ Dengan proper type
interface User {
  id: string;
  name: string;
  email: string;
}

const user: User = getUser();
user. // Autocomplete menampilkan: id, name, email
```

### **3. Refactoring Jadi Sulit**
```typescript
// ❌ Dengan any
function saveUser(data: any) {
  database.save(data);
}

// Jika struktur User berubah, tidak ada yang memberitahu
// bahwa function ini perlu diupdate

// ✅ Dengan proper type
interface User {
  id: string;
  name: string;
  email: string;
}

function saveUser(data: User) {
  database.save(data);
}

// Jika User interface berubah, TypeScript akan error
// di semua tempat yang perlu diupdate
```

---

## ✅ Alternatif untuk `any`

### **1. Gunakan `unknown` untuk Type yang Tidak Diketahui**
```typescript
// ❌ Salah
function parseJson(jsonString: string): any {
  return JSON.parse(jsonString);
}

const data = parseJson('{"name": "John"}');
console.log(data.name); // Tidak ada type checking

// ✅ Benar - gunakan unknown
function parseJson(jsonString: string): unknown {
  return JSON.parse(jsonString);
}

const data = parseJson('{"name": "John"}');
// console.log(data.name); // ❌ Error - data is unknown

// Harus melakukan type checking dulu
if (typeof data === 'object' && data !== null && 'name' in data) {
  console.log((data as { name: string }).name); // ✅ Type-safe
}
```

### **2. Gunakan Generic Type**
```typescript
// ❌ Salah
function wrapInArray(value: any): any[] {
  return [value];
}

// ✅ Benar - gunakan generic
function wrapInArray<T>(value: T): T[] {
  return [value];
}

const numbers = wrapInArray(5); // Type: number[]
const strings = wrapInArray('hello'); // Type: string[]
```

### **3. Gunakan Union Type**
```typescript
// ❌ Salah
function formatValue(value: any): string {
  return String(value);
}

// ✅ Benar - gunakan union type
function formatValue(value: string | number | boolean): string {
  return String(value);
}

formatValue('hello'); // ✅
formatValue(123); // ✅
formatValue(true); // ✅
// formatValue({}); // ❌ Error - object not allowed
```

### **4. Gunakan Type Guard**
```typescript
// Define type guard
interface User {
  id: string;
  name: string;
  email: string;
}

function isUser(obj: unknown): obj is User {
  return (
    typeof obj === 'object' &&
    obj !== null &&
    'id' in obj &&
    'name' in obj &&
    'email' in obj &&
    typeof (obj as User).id === 'string' &&
    typeof (obj as User).name === 'string' &&
    typeof (obj as User).email === 'string'
  );
}

// Usage
function processData(data: unknown) {
  if (isUser(data)) {
    // TypeScript tahu bahwa data adalah User
    console.log(data.name.toUpperCase()); // ✅ Type-safe
  } else {
    throw new Error('Invalid user data');
  }
}
```

### **5. Gunakan Utility Types**
```typescript
interface User {
  id: string;
  name: string;
  email: string;
  password: string;
}

// Partial - semua property jadi optional
type PartialUser = Partial<User>;
// { id?: string; name?: string; email?: string; password?: string; }

// Pick - pilih property tertentu
type UserCredentials = Pick<User, 'email' | 'password'>;
// { email: string; password: string; }

// Omit - exclude property tertentu
type UserWithoutPassword = Omit<User, 'password'>;
// { id: string; name: string; email: string; }

// Record - object dengan key-value type
type UserRoles = Record<string, {
  permissions: string[];
  level: number;
}>;
```

### **6. Gunakan Interface untuk Object**
```typescript
// ❌ Salah
const config: any = {
  apiUrl: 'https://api.example.com',
  timeout: 5000,
};

// ✅ Benar
interface Config {
  apiUrl: string;
  timeout: number;
  retries?: number;
}

const config: Config = {
  apiUrl: 'https://api.example.com',
  timeout: 5000,
};
```

### **7. Gunakan `Record<string, unknown>` untuk Dynamic Object**
```typescript
// ❌ Salah
function logObject(obj: any) {
  Object.keys(obj).forEach(key => {
    console.log(`${key}: ${obj[key]}`);
  });
}

// ✅ Benar
function logObject(obj: Record<string, unknown>) {
  Object.keys(obj).forEach(key => {
    console.log(`${key}: ${obj[key]}`);
  });
}
```

---

## 🎯 Kapan Boleh Menggunakan `any`?

### **Hampir Tidak Pernah!** Tapi ada beberapa kasus edge case:

### **1. Migration dari JavaScript ke TypeScript**
```typescript
// Temporary saat migration
// TODO: Add proper types
const legacyCode: any = require('./legacy-module');
```

### **2. Third-party Library Tanpa Type Definitions**
```typescript
// Jika library tidak punya @types dan tidak bisa dibuat type-nya
declare module 'some-old-library' {
  const content: any;
  export default content;
}

// Tapi lebih baik buat type definition sendiri:
declare module 'some-old-library' {
  export interface LibraryConfig {
    apiKey: string;
    timeout: number;
  }
  
  export function init(config: LibraryConfig): void;
}
```

### **3. Debugging Sementara**
```typescript
// TEMPORARY - untuk debugging
const debugData: any = complexFunction();
console.log(debugData);

// TODO: Replace with proper type after understanding the structure
```

**⚠️ PENTING:** Jika menggunakan `any` untuk kasus di atas, **HARUS** ada comment `// TODO: Add proper types` dan issue tracking untuk memperbaikinya!

---

## 📊 Comparison

| Aspek | `any` | `unknown` | Proper Type |
|:------|:------|:----------|:------------|
| **Type Safety** | ❌ Tidak ada | ⚠️ Harus di-check | ✅ Full type safety |
| **Autocomplete** | ❌ Tidak ada | ❌ Tidak ada | ✅ Full autocomplete |
| **Refactoring** | ❌ Sulit | ⚠️ Sedang | ✅ Mudah |
| **Runtime Error** | ⚠️ Tinggi | ⚠️ Sedang | ✅ Rendah |
| **Maintainability** | ❌ Buruk | ⚠️ Sedang | ✅ Baik |

---

## 🛠️ ESLint Rules untuk Prevent `any`

```json
{
  "rules": {
    "@typescript-eslint/no-explicit-any": "error",
    "@typescript-eslint/no-unsafe-assignment": "error",
    "@typescript-eslint/no-unsafe-member-access": "error",
    "@typescript-eslint/no-unsafe-call": "error",
    "@typescript-eslint/no-unsafe-return": "error"
  }
}
```

---

## 💡 Best Practices

### **1. Start with Specific Types**
```typescript
// ✅ Define interface first
interface ApiResponse<T> {
  success: boolean;
  data: T;
  error?: string;
}

function fetchData<T>(): Promise<ApiResponse<T>> {
  // Implementation
}
```

### **2. Use `unknown` for Unknown Data**
```typescript
// ✅ Use unknown, then narrow down
function processApiResponse(response: unknown) {
  if (isApiResponse(response)) {
    // Now TypeScript knows the type
    return response.data;
  }
  throw new Error('Invalid response');
}
```

### **3. Create Type Guards**
```typescript
// ✅ Reusable type guards
function isString(value: unknown): value is string {
  return typeof value === 'string';
}

function isNumber(value: unknown): value is number {
  return typeof value === 'number';
}
```

### **4. Use Zod for Runtime Validation**
```typescript
import { z } from 'zod';

// ✅ Schema validation + type inference
const UserSchema = z.object({
  id: z.string(),
  name: z.string(),
  email: z.string().email(),
  age: z.number().min(0).max(120),
});

type User = z.infer<typeof UserSchema>;

function validateUser(data: unknown): User {
  return UserSchema.parse(data); // Throws if invalid
}
```

---

## 🎓 Summary

### **❌ JANGAN:**
- Gunakan `any` untuk "cepat-cepatan"
- Gunakan `any` karena malas define type
- Gunakan `any` untuk "nanti aja di-fix"

### **✅ LAKUKAN:**
- Gunakan `unknown` untuk data yang tidak diketahui
- Gunakan generic type untuk reusable code
- Gunakan type guard untuk type narrowing
- Gunakan utility types untuk transform types
- Buat interface/type yang spesifik

---

## 📚 Resources

- [TypeScript Handbook - any](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#any)
- [TypeScript Handbook - unknown](https://www.typescriptlang.org/docs/handbook/2/functions.html#unknown)
- [TypeScript Deep Dive - Type Guards](https://basarat.gitbook.io/typescript/type-system/typeguard)

---

**Remember:** `any` defeats the purpose of using TypeScript! 🚫

Lihat [README.md](./README.md) untuk panduan lengkap code review.
