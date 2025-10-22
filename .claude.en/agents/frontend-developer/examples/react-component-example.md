# React Component 範例

本文件展示 React 18+ 元件開發的最佳實踐。

---

## 1. 基礎元件範例 - Button

### Button.tsx

```typescript
import { ButtonHTMLAttributes, ReactNode } from 'react';
import styles from './Button.module.css';

export interface ButtonProps extends ButtonHTMLAttributes<HTMLButtonElement> {
  /** Button content */
  children: ReactNode;
  /** Button variant */
  variant?: 'primary' | 'secondary' | 'danger';
  /** Button size */
  size?: 'small' | 'medium' | 'large';
  /** Loading state */
  loading?: boolean;
  /** Full width */
  fullWidth?: boolean;
}

export function Button({
  children,
  variant = 'primary',
  size = 'medium',
  loading = false,
  fullWidth = false,
  disabled,
  className,
  ...props
}: ButtonProps) {
  return (
    <button
      className={`
        ${styles.button}
        ${styles[variant]}
        ${styles[size]}
        ${fullWidth ? styles.fullWidth : ''}
        ${className || ''}
      `}
      disabled={disabled || loading}
      aria-busy={loading}
      {...props}
    >
      {loading ? (
        <>
          <span className={styles.spinner} role="progressbar" aria-label="Loading" />
          <span className={styles.loadingText}>{children}</span>
        </>
      ) : (
        children
      )}
    </button>
  );
}
```

### Button.module.css

```css
.button {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 0.5rem;
  border: none;
  border-radius: 0.375rem;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.2s;
}

.button:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

/* Variants */
.primary {
  background-color: var(--color-primary);
  color: white;
}

.primary:hover:not(:disabled) {
  background-color: var(--color-primary-dark);
}

.secondary {
  background-color: var(--color-secondary);
  color: var(--color-text);
}

.danger {
  background-color: var(--color-danger);
  color: white;
}

/* Sizes */
.small {
  padding: 0.5rem 1rem;
  font-size: 0.875rem;
}

.medium {
  padding: 0.75rem 1.5rem;
  font-size: 1rem;
}

.large {
  padding: 1rem 2rem;
  font-size: 1.125rem;
}

.fullWidth {
  width: 100%;
}

/* Loading */
.spinner {
  width: 1em;
  height: 1em;
  border: 2px solid currentColor;
  border-right-color: transparent;
  border-radius: 50%;
  animation: spin 0.75s linear infinite;
}

@keyframes spin {
  to {
    transform: rotate(360deg);
  }
}

.loadingText {
  opacity: 0.7;
}
```

### Button.test.tsx

```typescript
import { render, screen, fireEvent } from '@testing-library/react';
import { describe, it, expect, vi } from 'vitest';
import { Button } from './Button';

describe('Button', () => {
  it('renders with text', () => {
    render(<Button>Click me</Button>);
    expect(screen.getByText('Click me')).toBeInTheDocument();
  });

  it('calls onClick when clicked', () => {
    const onClick = vi.fn();
    render(<Button onClick={onClick}>Click</Button>);

    fireEvent.click(screen.getByText('Click'));

    expect(onClick).toHaveBeenCalledTimes(1);
  });

  it('shows loading spinner when loading', () => {
    render(<Button loading>Submit</Button>);

    expect(screen.getByRole('progressbar')).toBeInTheDocument();
    expect(screen.getByText('Submit')).toBeInTheDocument();
  });

  it('disables button when loading', () => {
    render(<Button loading>Submit</Button>);

    const button = screen.getByRole('button');
    expect(button).toBeDisabled();
    expect(button).toHaveAttribute('aria-busy', 'true');
  });

  it('applies variant styles', () => {
    const { rerender } = render(<Button variant="primary">Primary</Button>);
    expect(screen.getByRole('button')).toHaveClass('primary');

    rerender(<Button variant="danger">Danger</Button>);
    expect(screen.getByRole('button')).toHaveClass('danger');
  });

  it('applies size styles', () => {
    render(<Button size="large">Large</Button>);
    expect(screen.getByRole('button')).toHaveClass('large');
  });

  it('applies fullWidth styles', () => {
    render(<Button fullWidth>Full Width</Button>);
    expect(screen.getByRole('button')).toHaveClass('fullWidth');
  });
});
```

---

## 2. Form 元件範例 - FormField

### FormField.tsx

```typescript
import { InputHTMLAttributes, ReactNode } from 'react';
import styles from './FormField.module.css';

export interface FormFieldProps extends InputHTMLAttributes<HTMLInputElement> {
  /** Field label */
  label: string;
  /** Error message */
  error?: string;
  /** Helper text */
  helperText?: string;
  /** Required indicator */
  required?: boolean;
}

export function FormField({
  label,
  error,
  helperText,
  required,
  id,
  className,
  ...props
}: FormFieldProps) {
  const fieldId = id || `field-${label.toLowerCase().replace(/\s+/g, '-')}`;
  const errorId = `${fieldId}-error`;
  const helperId = `${fieldId}-helper`;

  return (
    <div className={`${styles.formField} ${className || ''}`}>
      <label htmlFor={fieldId} className={styles.label}>
        {label}
        {required && <span className={styles.required} aria-label="required">*</span>}
      </label>

      <input
        id={fieldId}
        className={`${styles.input} ${error ? styles.inputError : ''}`}
        aria-invalid={!!error}
        aria-describedby={error ? errorId : helperText ? helperId : undefined}
        required={required}
        {...props}
      />

      {error && (
        <p id={errorId} className={styles.error} role="alert">
          {error}
        </p>
      )}

      {helperText && !error && (
        <p id={helperId} className={styles.helper}>
          {helperText}
        </p>
      )}
    </div>
  );
}
```

### FormField.module.css

```css
.formField {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.label {
  font-weight: 500;
  color: var(--color-text);
}

.required {
  color: var(--color-danger);
  margin-left: 0.25rem;
}

.input {
  padding: 0.75rem;
  border: 1px solid var(--color-border);
  border-radius: 0.375rem;
  font-size: 1rem;
  transition: border-color 0.2s;
}

.input:focus {
  outline: none;
  border-color: var(--color-primary);
  box-shadow: 0 0 0 3px rgba(var(--color-primary-rgb), 0.1);
}

.inputError {
  border-color: var(--color-danger);
}

.inputError:focus {
  box-shadow: 0 0 0 3px rgba(var(--color-danger-rgb), 0.1);
}

.error {
  color: var(--color-danger);
  font-size: 0.875rem;
  margin: 0;
}

.helper {
  color: var(--color-text-muted);
  font-size: 0.875rem;
  margin: 0;
}
```

---

## 3. Container/Presentational Pattern 範例

### UserProfilePage.tsx (Container)

```typescript
import { useUserProfile } from '@/hooks/useUserProfile';
import { UserProfileView } from './UserProfileView';
import { LoadingSpinner } from '@/components/atoms/LoadingSpinner';
import { ErrorMessage } from '@/components/atoms/ErrorMessage';

export function UserProfilePage() {
  const { data: user, isLoading, error } = useUserProfile();

  if (isLoading) {
    return (
      <div className="loading-container" role="status" aria-live="polite">
        <LoadingSpinner />
        <p>Loading profile...</p>
      </div>
    );
  }

  if (error) {
    return (
      <ErrorMessage
        title="Failed to load profile"
        message={error.message}
        retry={() => window.location.reload()}
      />
    );
  }

  if (!user) {
    return <ErrorMessage title="User not found" />;
  }

  return <UserProfileView user={user} />;
}
```

### UserProfileView.tsx (Presentational)

```typescript
import { User } from '@/types/api.types';
import styles from './UserProfile.module.css';

export interface UserProfileViewProps {
  user: User;
}

export function UserProfileView({ user }: UserProfileViewProps) {
  return (
    <div className={styles.container}>
      <header className={styles.header}>
        <img
          src={user.avatar}
          alt={`${user.name}'s avatar`}
          className={styles.avatar}
        />
        <div>
          <h1 className={styles.name}>{user.name}</h1>
          <p className={styles.email}>{user.email}</p>
        </div>
      </header>

      <section className={styles.details} aria-labelledby="details-heading">
        <h2 id="details-heading" className={styles.sectionTitle}>
          Profile Details
        </h2>
        <dl className={styles.detailsList}>
          <dt>Username</dt>
          <dd>{user.username}</dd>

          <dt>Joined</dt>
          <dd>{new Date(user.createdAt).toLocaleDateString()}</dd>

          <dt>Role</dt>
          <dd>{user.role}</dd>
        </dl>
      </section>
    </div>
  );
}
```

---

## 4. Custom Hook 範例

### useUserProfile.ts

```typescript
import { useQuery } from '@tanstack/react-query';
import { userAPI } from '@/api/endpoints';
import { User } from '@/types/api.types';

export function useUserProfile() {
  return useQuery<User, Error>({
    queryKey: ['user', 'profile'],
    queryFn: async () => {
      const response = await userAPI.getProfile();
      return response.data;
    },
    staleTime: 5 * 60 * 1000, // 5 minutes
    retry: 2,
  });
}
```

### useForm.ts (Form validation hook)

```typescript
import { useState, ChangeEvent, FormEvent } from 'react';

interface ValidationRule<T> {
  validate: (value: T) => boolean;
  message: string;
}

export function useForm<T extends Record<string, any>>(
  initialValues: T,
  validationRules?: Partial<Record<keyof T, ValidationRule<T[keyof T]>[]>>
) {
  const [values, setValues] = useState<T>(initialValues);
  const [errors, setErrors] = useState<Partial<Record<keyof T, string>>>({});
  const [touched, setTouched] = useState<Partial<Record<keyof T, boolean>>>({});

  const handleChange = (field: keyof T) => (
    e: ChangeEvent<HTMLInputElement>
  ) => {
    const value = e.target.value;
    setValues((prev) => ({ ...prev, [field]: value }));

    // Clear error when user starts typing
    if (errors[field]) {
      setErrors((prev) => ({ ...prev, [field]: undefined }));
    }
  };

  const handleBlur = (field: keyof T) => () => {
    setTouched((prev) => ({ ...prev, [field]: true }));
    validateField(field);
  };

  const validateField = (field: keyof T) => {
    const rules = validationRules?.[field];
    if (!rules) return true;

    for (const rule of rules) {
      if (!rule.validate(values[field])) {
        setErrors((prev) => ({ ...prev, [field]: rule.message }));
        return false;
      }
    }

    setErrors((prev) => ({ ...prev, [field]: undefined }));
    return true;
  };

  const validateAll = () => {
    let isValid = true;
    const fields = Object.keys(values) as (keyof T)[];

    for (const field of fields) {
      if (!validateField(field)) {
        isValid = false;
      }
    }

    return isValid;
  };

  const handleSubmit = (onSubmit: (values: T) => void | Promise<void>) =>
    async (e: FormEvent) => {
      e.preventDefault();

      if (validateAll()) {
        await onSubmit(values);
      }
    };

  return {
    values,
    errors,
    touched,
    handleChange,
    handleBlur,
    handleSubmit,
    setValues,
    setErrors,
  };
}
```

**使用範例:**

```typescript
function LoginForm() {
  const { values, errors, touched, handleChange, handleBlur, handleSubmit } =
    useForm(
      { email: '', password: '' },
      {
        email: [
          {
            validate: (value) => value.length > 0,
            message: 'Email is required',
          },
          {
            validate: (value) => /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value),
            message: 'Invalid email format',
          },
        ],
        password: [
          {
            validate: (value) => value.length >= 8,
            message: 'Password must be at least 8 characters',
          },
        ],
      }
    );

  const onSubmit = async (values: typeof values) => {
    await authAPI.login(values.email, values.password);
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <FormField
        label="Email"
        type="email"
        value={values.email}
        onChange={handleChange('email')}
        onBlur={handleBlur('email')}
        error={touched.email ? errors.email : undefined}
        required
      />

      <FormField
        label="Password"
        type="password"
        value={values.password}
        onChange={handleChange('password')}
        onBlur={handleBlur('password')}
        error={touched.password ? errors.password : undefined}
        required
      />

      <Button type="submit">Login</Button>
    </form>
  );
}
```

---

## 最佳實踐總結

1. **TypeScript 嚴格型別** - 所有 Props 使用 interface 定義
2. **Accessibility** - semantic HTML, ARIA labels, keyboard support
3. **Error handling** - 顯示 error states, 提供 retry 機制
4. **Loading states** - 顯示 loading indicators, aria-live
5. **測試** - 元件測試、互動測試、accessibility 測試
6. **CSS Modules** - 避免全域樣式衝突
7. **Container/Presentational** - 分離業務邏輯與 UI 渲染
8. **Custom Hooks** - 抽取可重用邏輯
