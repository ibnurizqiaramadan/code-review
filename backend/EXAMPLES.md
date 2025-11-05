`# 🔧 Backend Code Examples - Detail

Contoh lengkap implementasi backend yang benar dan salah

---

## 📁 Project Structure (Recommended)

```
backend/
├── src/
│   ├── config/
│   │   ├── database.ts
│   │   └── env.ts
│   ├── controllers/
│   │   ├── auth.controller.ts
│   │   └── user.controller.ts
│   ├── services/
│   │   ├── auth.service.ts
│   │   └── user.service.ts
│   ├── repositories/
│   │   └── user.repository.ts
│   ├── models/
│   │   └── user.model.ts
│   ├── middleware/
│   │   ├── auth.middleware.ts
│   │   ├── validation.middleware.ts
│   │   └── error.middleware.ts
│   ├── validators/
│   │   └── user.validator.ts
│   ├── utils/
│   │   ├── response.util.ts
│   │   └── logger.util.ts
│   ├── types/
│   │   └── express.d.ts
│   ├── routes/
│   │   ├── auth.routes.ts
│   │   └── user.routes.ts
│   └── app.ts
├── tests/
│   ├── unit/
│   └── integration/
├── .env.example
├── .eslintrc.json
├── tsconfig.json
└── package.json
```

---

## 🌐 Complete API Example

### **1. Response Utility**
```typescript
// src/utils/response.util.ts
export interface ApiResponse<T = unknown> {
  success: boolean;
  data?: T;
  error?: {
    code: string;
    message: string;
    details?: unknown;
  };
  meta?: {
    page?: number;
    limit?: number;
    total?: number;
    totalPages?: number;
  };
}

export class ResponseUtil {
  static success<T>(data: T, meta?: ApiResponse<T>['meta']): ApiResponse<T> {
    return {
      success: true,
      data,
      ...(meta && { meta }),
    };
  }

  static error(
    code: string,
    message: string,
    details?: unknown
  ): ApiResponse<never> {
    return {
      success: false,
      error: {
        code,
        message,
        details,
      },
    };
  }

  static paginated<T>(
    data: T[],
    page: number,
    limit: number,
    total: number
  ): ApiResponse<T[]> {
    return {
      success: true,
      data,
      meta: {
        page,
        limit,
        total,
        totalPages: Math.ceil(total / limit),
      },
    };
  }
}
```

### **2. Validation Middleware**
```typescript
// src/validators/user.validator.ts
import Joi from 'joi';

export const userValidators = {
  create: Joi.object({
    email: Joi.string().email().required(),
    password: Joi.string().min(8).required(),
    name: Joi.string().min(2).max(100).required(),
    role: Joi.string().valid('user', 'admin').default('user'),
  }),

  update: Joi.object({
    email: Joi.string().email().optional(),
    name: Joi.string().min(2).max(100).optional(),
    role: Joi.string().valid('user', 'admin').optional(),
  }).min(1),

  login: Joi.object({
    email: Joi.string().email().required(),
    password: Joi.string().required(),
  }),
};

// src/middleware/validation.middleware.ts
import { Request, Response, NextFunction } from 'express';
import { ObjectSchema } from 'joi';
import { ResponseUtil } from '../utils/response.util';

export const validate = (schema: ObjectSchema) => {
  return (req: Request, res: Response, next: NextFunction) => {
    const { error, value } = schema.validate(req.body, {
      abortEarly: false,
      stripUnknown: true,
    });

    if (error) {
      const errors = error.details.map(detail => ({
        field: detail.path.join('.'),
        message: detail.message,
      }));

      return res.status(400).json(
        ResponseUtil.error('VALIDATION_ERROR', 'Invalid input', errors)
      );
    }

    req.body = value;
    next();
  };
};
```

### **3. Authentication Middleware**
```typescript
// src/middleware/auth.middleware.ts
import { Request, Response, NextFunction } from 'express';
import jwt from 'jsonwebtoken';
import { ResponseUtil } from '../utils/response.util';
import { UserRepository } from '../repositories/user.repository';

interface JwtPayload {
  userId: string;
  role: string;
  exp: number;
}

export const authenticate = async (
  req: Request,
  res: Response,
  next: NextFunction
) => {
  try {
    const authHeader = req.headers.authorization;

    if (!authHeader || !authHeader.startsWith('Bearer ')) {
      return res.status(401).json(
        ResponseUtil.error('NO_TOKEN', 'Authentication required')
      );
    }

    const token = authHeader.split(' ')[1];

    const decoded = jwt.verify(
      token,
      process.env.JWT_SECRET!
    ) as JwtPayload;

    if (decoded.exp < Date.now() / 1000) {
      return res.status(401).json(
        ResponseUtil.error('TOKEN_EXPIRED', 'Token has expired')
      );
    }

    const userRepo = new UserRepository();
    const user = await userRepo.findById(decoded.userId);

    if (!user || !user.isActive) {
      return res.status(401).json(
        ResponseUtil.error('INVALID_USER', 'User not found or inactive')
      );
    }

    req.user = user;
    next();
  } catch (error) {
    return res.status(401).json(
      ResponseUtil.error('INVALID_TOKEN', 'Invalid token')
    );
  }
};

export const authorize = (...roles: string[]) => {
  return (req: Request, res: Response, next: NextFunction) => {
    if (!req.user) {
      return res.status(401).json(
        ResponseUtil.error('UNAUTHORIZED', 'Authentication required')
      );
    }

    if (!roles.includes(req.user.role)) {
      return res.status(403).json(
        ResponseUtil.error('FORBIDDEN', 'Access denied')
      );
    }

    next();
  };
};
```

### **4. Service Layer**
```typescript
// src/services/user.service.ts
import bcrypt from 'bcrypt';
import jwt from 'jsonwebtoken';
import { UserRepository } from '../repositories/user.repository';
import { AppError } from '../middleware/error.middleware';

export class UserService {
  private readonly SALT_ROUNDS = 10;
  private userRepo: UserRepository;

  constructor() {
    this.userRepo = new UserRepository();
  }

  async register(email: string, password: string, name: string) {
    const existingUser = await this.userRepo.findByEmail(email);

    if (existingUser) {
      throw new AppError(409, 'EMAIL_EXISTS', 'Email already registered');
    }

    const hashedPassword = await bcrypt.hash(password, this.SALT_ROUNDS);

    const user = await this.userRepo.create({
      email,
      password: hashedPassword,
      name,
      role: 'user',
    });

    const { password: _, ...userWithoutPassword } = user;

    return userWithoutPassword;
  }

  async login(email: string, password: string) {
    const user = await this.userRepo.findByEmail(email);

    if (!user) {
      throw new AppError(401, 'INVALID_CREDENTIALS', 'Invalid credentials');
    }

    const isValidPassword = await bcrypt.compare(password, user.password);

    if (!isValidPassword) {
      throw new AppError(401, 'INVALID_CREDENTIALS', 'Invalid credentials');
    }

    const token = jwt.sign(
      { userId: user.id, role: user.role },
      process.env.JWT_SECRET!,
      { expiresIn: '7d' }
    );

    return {
      token,
      user: {
        id: user.id,
        email: user.email,
        name: user.name,
        role: user.role,
      },
    };
  }

  async getAll(page: number = 1, limit: number = 10) {
    const offset = (page - 1) * limit;
    return await this.userRepo.findAll(limit, offset);
  }

  async getById(id: string) {
    const user = await this.userRepo.findById(id);

    if (!user) {
      throw new AppError(404, 'USER_NOT_FOUND', 'User not found');
    }

    const { password: _, ...userWithoutPassword } = user;
    return userWithoutPassword;
  }

  async update(id: string, data: Partial<{ email: string; name: string }>) {
    const user = await this.userRepo.findById(id);

    if (!user) {
      throw new AppError(404, 'USER_NOT_FOUND', 'User not found');
    }

    if (data.email && data.email !== user.email) {
      const existingUser = await this.userRepo.findByEmail(data.email);
      if (existingUser) {
        throw new AppError(409, 'EMAIL_EXISTS', 'Email already in use');
      }
    }

    const updatedUser = await this.userRepo.update(id, data);
    const { password: _, ...userWithoutPassword } = updatedUser;

    return userWithoutPassword;
  }

  async delete(id: string) {
    const user = await this.userRepo.findById(id);

    if (!user) {
      throw new AppError(404, 'USER_NOT_FOUND', 'User not found');
    }

    await this.userRepo.delete(id);
  }
}
```

### **5. Controller Layer**
```typescript
// src/controllers/user.controller.ts
import { Request, Response, NextFunction } from 'express';
import { UserService } from '../services/user.service';
import { ResponseUtil } from '../utils/response.util';

export class UserController {
  private userService: UserService;

  constructor() {
    this.userService = new UserService();
  }

  register = async (req: Request, res: Response, next: NextFunction) => {
    try {
      const { email, password, name } = req.body;
      const user = await this.userService.register(email, password, name);

      res.status(201).json(ResponseUtil.success(user));
    } catch (error) {
      next(error);
    }
  };

  login = async (req: Request, res: Response, next: NextFunction) => {
    try {
      const { email, password } = req.body;
      const result = await this.userService.login(email, password);

      res.json(ResponseUtil.success(result));
    } catch (error) {
      next(error);
    }
  };

  getAll = async (req: Request, res: Response, next: NextFunction) => {
    try {
      const page = parseInt(req.query.page as string) || 1;
      const limit = parseInt(req.query.limit as string) || 10;

      const { users, total } = await this.userService.getAll(page, limit);

      res.json(ResponseUtil.paginated(users, page, limit, total));
    } catch (error) {
      next(error);
    }
  };

  getById = async (req: Request, res: Response, next: NextFunction) => {
    try {
      const user = await this.userService.getById(req.params.id);
      res.json(ResponseUtil.success(user));
    } catch (error) {
      next(error);
    }
  };

  update = async (req: Request, res: Response, next: NextFunction) => {
    try {
      const user = await this.userService.update(req.params.id, req.body);
      res.json(ResponseUtil.success(user));
    } catch (error) {
      next(error);
    }
  };

  delete = async (req: Request, res: Response, next: NextFunction) => {
    try {
      await this.userService.delete(req.params.id);
      res.status(204).send();
    } catch (error) {
      next(error);
    }
  };
}
```

### **6. Routes**
```typescript
// src/routes/user.routes.ts
import { Router } from 'express';
import { UserController } from '../controllers/user.controller';
import { authenticate, authorize } from '../middleware/auth.middleware';
import { validate } from '../middleware/validation.middleware';
import { userValidators } from '../validators/user.validator';

const router = Router();
const userController = new UserController();

// Public routes
router.post('/register', validate(userValidators.create), userController.register);
router.post('/login', validate(userValidators.login), userController.login);

// Protected routes
router.get('/', authenticate, authorize('admin'), userController.getAll);
router.get('/:id', authenticate, userController.getById);
router.put('/:id', authenticate, validate(userValidators.update), userController.update);
router.delete('/:id', authenticate, authorize('admin'), userController.delete);

export default router;
```

### **7. Error Handler**
```typescript
// src/middleware/error.middleware.ts
import { Request, Response, NextFunction } from 'express';
import { ResponseUtil } from '../utils/response.util';
import { logger } from '../utils/logger.util';

export class AppError extends Error {
  constructor(
    public statusCode: number,
    public code: string,
    message: string,
    public isOperational = true
  ) {
    super(message);
    Object.setPrototypeOf(this, AppError.prototype);
  }
}

export const errorHandler = (
  err: Error,
  req: Request,
  res: Response,
  next: NextFunction
) => {
  logger.error({
    message: err.message,
    stack: err.stack,
    url: req.url,
    method: req.method,
    body: req.body,
    user: req.user?.id,
  });

  if (err instanceof AppError) {
    return res.status(err.statusCode).json(
      ResponseUtil.error(err.code, err.message)
    );
  }

  if (process.env.NODE_ENV === 'development') {
    return res.status(500).json({
      success: false,
      error: {
        code: 'INTERNAL_ERROR',
        message: err.message,
        stack: err.stack,
      },
    });
  }

  return res.status(500).json(
    ResponseUtil.error('INTERNAL_ERROR', 'Something went wrong')
  );
};
```

---

## 🧪 Testing Examples

### **Unit Test**
```typescript
// tests/unit/user.service.test.ts
import { UserService } from '../../src/services/user.service';
import { UserRepository } from '../../src/repositories/user.repository';
import { AppError } from '../../src/middleware/error.middleware';
import bcrypt from 'bcrypt';

jest.mock('../../src/repositories/user.repository');
jest.mock('bcrypt');

describe('UserService', () => {
  let userService: UserService;
  let mockUserRepo: jest.Mocked<UserRepository>;

  beforeEach(() => {
    mockUserRepo = new UserRepository() as jest.Mocked<UserRepository>;
    userService = new UserService();
    (userService as any).userRepo = mockUserRepo;
  });

  describe('register', () => {
    it('should register user successfully', async () => {
      mockUserRepo.findByEmail.mockResolvedValue(null);
      mockUserRepo.create.mockResolvedValue({
        id: '1',
        email: 'test@example.com',
        name: 'Test User',
        password: 'hashed',
        role: 'user',
      } as any);

      (bcrypt.hash as jest.Mock).mockResolvedValue('hashed');

      const result = await userService.register(
        'test@example.com',
        'password123',
        'Test User'
      );

      expect(result).not.toHaveProperty('password');
      expect(result.email).toBe('test@example.com');
    });

    it('should throw error if email exists', async () => {
      mockUserRepo.findByEmail.mockResolvedValue({ id: '1' } as any);

      await expect(
        userService.register('test@example.com', 'password123', 'Test')
      ).rejects.toThrow(AppError);
    });
  });
});
```

### **Integration Test**
```typescript
// tests/integration/user.test.ts
import request from 'supertest';
import app from '../../src/app';
import { setupTestDB, cleanupTestDB } from '../helpers/testDb';

describe('User API', () => {
  beforeAll(async () => {
    await setupTestDB();
  });

  afterAll(async () => {
    await cleanupTestDB();
  });

  describe('POST /users/register', () => {
    it('should register user with valid data', async () => {
      const response = await request(app)
        .post('/users/register')
        .send({
          email: 'test@example.com',
          password: 'password123',
          name: 'Test User',
        })
        .expect(201);

      expect(response.body.success).toBe(true);
      expect(response.body.data).toHaveProperty('id');
      expect(response.body.data.email).toBe('test@example.com');
    });

    it('should return 400 with invalid email', async () => {
      const response = await request(app)
        .post('/users/register')
        .send({
          email: 'invalid-email',
          password: 'password123',
          name: 'Test',
        })
        .expect(400);

      expect(response.body.success).toBe(false);
      expect(response.body.error.code).toBe('VALIDATION_ERROR');
    });
  });
});
```

---

Lihat [README.md](./README.md) untuk checklist lengkap.
