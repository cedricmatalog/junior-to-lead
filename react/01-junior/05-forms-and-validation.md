# Forms and Validation

Build robust forms with proper validation and user experience.

## Learning Objectives

By the end of this module, you will:
- Build controlled forms with React state
- Use React Hook Form for complex forms
- Implement schema validation with Zod/Yup
- Display errors effectively to users

## Controlled Forms Basics

```jsx
function SimpleForm() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [errors, setErrors] = useState({});

  function handleSubmit(e) {
    e.preventDefault();

    // Basic validation
    const newErrors = {};
    if (!email.includes('@')) newErrors.email = 'Invalid email';
    if (password.length < 8) newErrors.password = 'Too short';

    if (Object.keys(newErrors).length > 0) {
      setErrors(newErrors);
      return;
    }

    // Submit form
    console.log({ email, password });
  }

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="email"
        value={email}
        onChange={e => setEmail(e.target.value)}
      />
      {errors.email && <span className="error">{errors.email}</span>}

      <input
        type="password"
        value={password}
        onChange={e => setPassword(e.target.value)}
      />
      {errors.password && <span className="error">{errors.password}</span>}

      <button type="submit">Submit</button>
    </form>
  );
}
```

## React Hook Form

Reduces boilerplate and improves performance for complex forms.

```jsx
import { useForm } from 'react-hook-form';

function LoginForm() {
  const {
    register,
    handleSubmit,
    formState: { errors, isSubmitting }
  } = useForm();

  async function onSubmit(data) {
    await submitToAPI(data);
  }

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input
        {...register('email', {
          required: 'Email is required',
          pattern: {
            value: /^[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,}$/i,
            message: 'Invalid email address'
          }
        })}
      />
      {errors.email && <span>{errors.email.message}</span>}

      <input
        type="password"
        {...register('password', {
          required: 'Password is required',
          minLength: {
            value: 8,
            message: 'Must be at least 8 characters'
          }
        })}
      />
      {errors.password && <span>{errors.password.message}</span>}

      <button disabled={isSubmitting}>
        {isSubmitting ? 'Submitting...' : 'Submit'}
      </button>
    </form>
  );
}
```

## Schema Validation with Zod

Type-safe validation with excellent TypeScript support.

```jsx
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';

// Define schema
const schema = z.object({
  email: z.string().email('Invalid email'),
  password: z.string().min(8, 'Too short').max(100),
  confirmPassword: z.string(),
}).refine(data => data.password === data.confirmPassword, {
  message: 'Passwords must match',
  path: ['confirmPassword'],
});

type FormData = z.infer<typeof schema>;

function RegisterForm() {
  const { register, handleSubmit, formState: { errors } } = useForm<FormData>({
    resolver: zodResolver(schema)
  });

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input {...register('email')} />
      {errors.email && <span>{errors.email.message}</span>}

      <input type="password" {...register('password')} />
      {errors.password && <span>{errors.password.message}</span>}

      <input type="password" {...register('confirmPassword')} />
      {errors.confirmPassword && <span>{errors.confirmPassword.message}</span>}

      <button type="submit">Register</button>
    </form>
  );
}
```

## Displaying Errors Effectively

```jsx
function FormField({ label, error, children }) {
  return (
    <div className="form-field">
      <label>{label}</label>
      {children}
      {error && (
        <span className="error" role="alert">
          {error}
        </span>
      )}
    </div>
  );
}

// Accessible error styling
// .error { color: #d32f2f; font-size: 0.875rem; }
// input[aria-invalid="true"] { border-color: #d32f2f; }
```

**Error UX best practices:**
- Show errors inline, near the relevant field
- Use `aria-invalid` and `role="alert"` for accessibility
- Validate on blur, not on every keystroke
- Clear errors when user starts fixing them
- Show all errors at once, not one at a time

## Validation Patterns

```jsx
// Common Zod patterns
const userSchema = z.object({
  // Basic types
  name: z.string().min(1, 'Required'),
  age: z.number().int().positive(),

  // Email and URL
  email: z.string().email(),
  website: z.string().url().optional(),

  // Enums
  role: z.enum(['admin', 'user', 'guest']),

  // Arrays
  tags: z.array(z.string()).min(1, 'Add at least one tag'),

  // Conditional
  phone: z.string().optional(),
  phoneRequired: z.boolean(),
}).refine(data => !data.phoneRequired || data.phone, {
  message: 'Phone is required',
  path: ['phone'],
});
```

## Common Mistakes

1. **Validating on every keystroke** - Distracting, use blur or submit
2. **Not handling async validation** - Check uniqueness, loading states
3. **Clearing form on error** - Preserve user input
4. **Generic error messages** - Be specific about what's wrong

## Practice Exercises

1. Build a multi-step registration form with validation
2. Create a dynamic form that adds/removes fields
3. Implement async validation (e.g., check if username exists)

## Further Reading

- [React Hook Form Documentation](https://react-hook-form.com/)
- [Zod Documentation](https://zod.dev/)
