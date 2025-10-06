# State Management 範例

展示 React (Zustand), Vue (Pinia), Angular (Services + RxJS) 的狀態管理最佳實踐。

---

## 1. React + Zustand

### stores/authStore.ts

```typescript
import { create } from 'zustand';
import { persist } from 'zustand/middleware';
import { authAPI } from '@/api/endpoints';

export interface User {
  id: string;
  email: string;
  name: string;
  role: string;
}

interface AuthState {
  user: User | null;
  token: string | null;
  isAuthenticated: boolean;
  login: (email: string, password: string) => Promise<void>;
  logout: () => void;
  updateUser: (user: Partial<User>) => void;
}

export const useAuthStore = create<AuthState>()(
  persist(
    (set) => ({
      user: null,
      token: null,
      isAuthenticated: false,

      login: async (email, password) => {
        try {
          const { data } = await authAPI.login(email, password);
          set({
            user: data.user,
            token: data.token,
            isAuthenticated: true,
          });
        } catch (error) {
          throw new Error('Login failed');
        }
      },

      logout: () => {
        set({ user: null, token: null, isAuthenticated: false });
      },

      updateUser: (userData) => {
        set((state) => ({
          user: state.user ? { ...state.user, ...userData } : null,
        }));
      },
    }),
    {
      name: 'auth-storage',
      partialize: (state) => ({ token: state.token, user: state.user }),
    }
  )
);
```

### 使用範例

```typescript
function LoginForm() {
  const { login } = useAuthStore();
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [loading, setLoading] = useState(false);

  const handleSubmit = async (e: FormEvent) => {
    e.preventDefault();
    setLoading(true);

    try {
      await login(email, password);
      // Redirect to dashboard
    } catch (error) {
      // Show error message
    } finally {
      setLoading(false);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <Input value={email} onChange={(e) => setEmail(e.target.value)} />
      <Input
        type="password"
        value={password}
        onChange={(e) => setPassword(e.target.value)}
      />
      <Button type="submit" loading={loading}>
        Login
      </Button>
    </form>
  );
}

function UserMenu() {
  const { user, logout } = useAuthStore();

  if (!user) return null;

  return (
    <div>
      <p>Welcome, {user.name}</p>
      <Button onClick={logout}>Logout</Button>
    </div>
  );
}
```

---

## 2. Vue 3 + Pinia

### stores/authStore.ts

```typescript
import { defineStore } from 'pinia';
import { ref, computed } from 'vue';
import { authAPI } from '@/api/endpoints';

export interface User {
  id: string;
  email: string;
  name: string;
  role: string;
}

export const useAuthStore = defineStore('auth', () => {
  // State
  const user = ref<User | null>(null);
  const token = ref<string | null>(null);

  // Getters
  const isAuthenticated = computed(() => !!token.value);

  // Actions
  async function login(email: string, password: string) {
    try {
      const { data } = await authAPI.login(email, password);
      user.value = data.user;
      token.value = data.token;
      localStorage.setItem('token', data.token);
    } catch (error) {
      throw new Error('Login failed');
    }
  }

  function logout() {
    user.value = null;
    token.value = null;
    localStorage.removeItem('token');
  }

  function updateUser(userData: Partial<User>) {
    if (user.value) {
      user.value = { ...user.value, ...userData };
    }
  }

  // Initialize from localStorage
  function initializeAuth() {
    const savedToken = localStorage.getItem('token');
    if (savedToken) {
      token.value = savedToken;
      // Optionally fetch user data
    }
  }

  return {
    user,
    token,
    isAuthenticated,
    login,
    logout,
    updateUser,
    initializeAuth,
  };
});
```

### 使用範例

```vue
<template>
  <form @submit.prevent="handleSubmit">
    <input v-model="email" type="email" placeholder="Email" />
    <input v-model="password" type="password" placeholder="Password" />
    <button type="submit" :disabled="loading">
      {{ loading ? 'Logging in...' : 'Login' }}
    </button>
  </form>
</template>

<script setup lang="ts">
import { ref } from 'vue';
import { useAuthStore } from '@/stores/authStore';
import { useRouter } from 'vue-router';

const authStore = useAuthStore();
const router = useRouter();

const email = ref('');
const password = ref('');
const loading = ref(false);

async function handleSubmit() {
  loading.value = true;

  try {
    await authStore.login(email.value, password.value);
    router.push('/dashboard');
  } catch (error) {
    // Show error message
  } finally {
    loading.value = false;
  }
}
</script>
```

---

## 3. Angular + Services + RxJS

### services/auth.service.ts

```typescript
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { BehaviorSubject, Observable, tap } from 'rxjs';

export interface User {
  id: string;
  email: string;
  name: string;
  role: string;
}

interface AuthResponse {
  user: User;
  token: string;
}

@Injectable({ providedIn: 'root' })
export class AuthService {
  private userSubject = new BehaviorSubject<User | null>(null);
  private tokenSubject = new BehaviorSubject<string | null>(null);

  public user$ = this.userSubject.asObservable();
  public token$ = this.tokenSubject.asObservable();
  public isAuthenticated$ = new BehaviorSubject<boolean>(false);

  constructor(private http: HttpClient) {
    this.initializeAuth();
  }

  private initializeAuth(): void {
    const token = localStorage.getItem('token');
    if (token) {
      this.tokenSubject.next(token);
      this.isAuthenticated$.next(true);
      // Optionally fetch user data
    }
  }

  login(email: string, password: string): Observable<AuthResponse> {
    return this.http
      .post<AuthResponse>('/api/auth/login', { email, password })
      .pipe(
        tap((response) => {
          this.userSubject.next(response.user);
          this.tokenSubject.next(response.token);
          this.isAuthenticated$.next(true);
          localStorage.setItem('token', response.token);
        })
      );
  }

  logout(): void {
    this.userSubject.next(null);
    this.tokenSubject.next(null);
    this.isAuthenticated$.next(false);
    localStorage.removeItem('token');
  }

  updateUser(userData: Partial<User>): void {
    const currentUser = this.userSubject.value;
    if (currentUser) {
      this.userSubject.next({ ...currentUser, ...userData });
    }
  }

  get currentUser(): User | null {
    return this.userSubject.value;
  }

  get isAuthenticated(): boolean {
    return this.isAuthenticated$.value;
  }
}
```

### 使用範例 (Component)

```typescript
import { Component } from '@angular/core';
import { FormBuilder, FormGroup, Validators } from '@angular/forms';
import { Router } from '@angular/router';
import { AuthService } from '@/services/auth.service';

@Component({
  selector: 'app-login',
  template: `
    <form [formGroup]="loginForm" (ngSubmit)="onSubmit()">
      <input formControlName="email" type="email" placeholder="Email" />
      <input
        formControlName="password"
        type="password"
        placeholder="Password"
      />
      <button type="submit" [disabled]="loginForm.invalid || loading">
        {{ loading ? 'Logging in...' : 'Login' }}
      </button>
    </form>
  `,
})
export class LoginComponent {
  loginForm: FormGroup;
  loading = false;

  constructor(
    private fb: FormBuilder,
    private authService: AuthService,
    private router: Router
  ) {
    this.loginForm = this.fb.group({
      email: ['', [Validators.required, Validators.email]],
      password: ['', [Validators.required, Validators.minLength(8)]],
    });
  }

  onSubmit(): void {
    if (this.loginForm.valid) {
      this.loading = true;
      const { email, password } = this.loginForm.value;

      this.authService.login(email, password).subscribe({
        next: () => {
          this.router.navigate(['/dashboard']);
        },
        error: (error) => {
          // Show error message
          this.loading = false;
        },
        complete: () => {
          this.loading = false;
        },
      });
    }
  }
}
```

---

## 最佳實踐總結

### React + Zustand

**優點:**

- 簡單、輕量（< 1KB）
- 無 Provider 包裝
- TypeScript 支援優秀
- 支援 Middleware（persist, devtools）

**使用時機:**

- 簡單到中等複雜度的應用
- 需要快速開發
- 不需要 Redux DevTools 時間旅行除錯

### Vue 3 + Pinia

**優點:**

- 官方推薦（取代 Vuex）
- Composition API 整合良好
- TypeScript 支援優秀
- DevTools 整合

**使用時機:**

- Vue 3 應用（官方推薦）
- 需要模組化 store
- 需要 DevTools 整合

### Angular + Services + RxJS

**優點:**

- 與 Angular 生態系統完美整合
- RxJS 強大的響應式編程
- Dependency Injection
- 適合複雜應用

**使用時機:**

- Angular 應用（標準做法）
- 需要複雜的資料流管理
- 需要 RxJS operators

---

## 測試範例

### React + Zustand 測試

```typescript
import { renderHook, act } from '@testing-library/react';
import { useAuthStore } from './authStore';
import { authAPI } from '@/api/endpoints';
import { vi } from 'vitest';

vi.mock('@/api/endpoints');

describe('useAuthStore', () => {
  beforeEach(() => {
    useAuthStore.setState({ user: null, token: null, isAuthenticated: false });
  });

  it('logs in successfully', async () => {
    const mockUser = { id: '1', email: 'test@example.com', name: 'Test' };
    vi.mocked(authAPI.login).mockResolvedValue({
      data: { user: mockUser, token: 'token123' },
    });

    const { result } = renderHook(() => useAuthStore());

    await act(async () => {
      await result.current.login('test@example.com', 'password');
    });

    expect(result.current.user).toEqual(mockUser);
    expect(result.current.token).toBe('token123');
    expect(result.current.isAuthenticated).toBe(true);
  });

  it('logs out successfully', () => {
    const { result } = renderHook(() => useAuthStore());

    act(() => {
      useAuthStore.setState({
        user: { id: '1', email: 'test@example.com', name: 'Test' },
        token: 'token123',
        isAuthenticated: true,
      });
    });

    act(() => {
      result.current.logout();
    });

    expect(result.current.user).toBeNull();
    expect(result.current.token).toBeNull();
    expect(result.current.isAuthenticated).toBe(false);
  });
});
```
