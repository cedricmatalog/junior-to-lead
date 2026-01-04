# Forms and Validation

## Week Five: The Job Application

The food delivery feature is live. Orders are flowing. But there's a new priority from the business team.

"We're hiring," Sarah says. "A lot. And HR wants an application portal built into the platform."

You've built forms before—simple inputs, maybe a contact form. But this is different. Multi-step wizard. File uploads. Conditional fields. Real-time validation.

"Forms are deceptively hard," Marcus warns. "They look simple until they're not."

He's about to show you why.

## Mental Models

Before we dive in, here's how to think about the core concepts:

| Concept | Think of it as... |
|---------|-------------------|
| **Controlled inputs** | A secretary taking dictation - every keystroke reported, you dictate what appears |
| **Validation** | A bouncer checking IDs - verify data meets requirements before letting it through |
| **Schema validation** | A DMV form template - defines all rules in one place |

Keep these in mind. They'll click as we build.

---

## Chapter 1: The Controlled Input

"First, understand how React handles form inputs," Sarah starts.

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

"Every keystroke updates state. State controls the input value. That's a controlled input."

It works, but already you can see the problem. Two fields, and there's a lot of boilerplate. The job application has 20+ fields.

---

## Chapter 2: React Hook Form to the Rescue

"Nobody writes forms by hand anymore," Marcus says. "Meet React Hook Form."

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

"See how `register` handles everything? It connects the input, tracks the value, validates, and reports errors."

The boilerplate disappears. The intent becomes clear.

---

## Chapter 3: Schema Validation with Zod

"For the job application, we need serious validation," Sarah says. "That's where Zod comes in."

Zod lets you define your validation rules as a schema—a single source of truth.

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

"The schema is your contract. If it passes Zod, you know exactly what shape the data has."

---

## Chapter 4: Displaying Errors Right

"Forms fail silently too often," Marcus says. "Users type, click submit, and nothing happens. They don't know what's wrong."

He shows you a better pattern:

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
```

**Error UX best practices:**
- Show errors inline, near the relevant field
- Use `aria-invalid` and `role="alert"` for accessibility
- Validate on blur, not on every keystroke
- Clear errors when user starts fixing them
- Show all errors at once, not one at a time

---

## Chapter 5: The Job Application Schema

Time to build the real thing. The job application is complex:

```jsx
const applicationSchema = z.object({
  // Personal Info
  firstName: z.string().min(1, 'First name is required'),
  lastName: z.string().min(1, 'Last name is required'),
  email: z.string().email('Invalid email address'),
  phone: z.string().regex(/^\+?[\d\s-]{10,}$/, 'Invalid phone number'),

  // Work Experience
  experiences: z.array(z.object({
    company: z.string().min(1, 'Company name is required'),
    title: z.string().min(1, 'Job title is required'),
    startDate: z.string().min(1, 'Start date is required'),
    endDate: z.string().optional(),
    current: z.boolean(),
    description: z.string().max(500, 'Keep it under 500 characters'),
  })).min(1, 'Add at least one work experience'),

  // Additional
  linkedinUrl: z.string().url('Invalid URL').optional().or(z.literal('')),
  portfolioUrl: z.string().url('Invalid URL').optional().or(z.literal('')),
  salary: z.number().min(0, 'Salary must be positive'),
  startAvailability: z.enum(['immediately', '2weeks', '1month', 'other']),
  customAvailability: z.string().optional(),
  referralSource: z.string().min(1, 'Please tell us how you heard about us'),

  // Legal
  authorizedToWork: z.boolean().refine(val => val === true, 'Required'),
  agreedToTerms: z.boolean().refine(val => val === true, 'You must agree to continue'),
}).refine(
  data => data.startAvailability !== 'other' || data.customAvailability,
  { message: 'Please specify your availability', path: ['customAvailability'] }
);
```

"Notice the `.refine()` calls," Sarah points out. "That's how you add custom validation logic—like requiring a custom availability when 'other' is selected."

---

## Chapter 6: Multi-Step Wizard

A 20-field form is overwhelming. You break it into steps:

```jsx
function JobApplicationForm({ jobId, onSubmit }) {
  const [step, setStep] = useState(1);

  const {
    register,
    handleSubmit,
    control,
    watch,
    trigger,
    formState: { errors, isSubmitting }
  } = useForm({
    resolver: zodResolver(applicationSchema),
    defaultValues: {
      experiences: [{ company: '', title: '', startDate: '', current: false, description: '' }],
      startAvailability: 'immediately',
    }
  });

  const { fields, append, remove } = useFieldArray({ control, name: 'experiences' });
  const availability = watch('startAvailability');

  // Validate current step before proceeding
  const nextStep = async () => {
    const fieldsToValidate = {
      1: ['firstName', 'lastName', 'email', 'phone'],
      2: ['experiences'],
      3: ['salary', 'startAvailability', 'customAvailability', 'referralSource'],
    }[step];

    const isValid = await trigger(fieldsToValidate);
    if (isValid) setStep(s => s + 1);
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)} className="application-form">
      <ProgressBar current={step} total={4} />

      {step === 1 && (
        <section>
          <h2>Personal Information</h2>
          <div className="form-row">
            <FormField label="First Name" error={errors.firstName?.message}>
              <input {...register('firstName')} />
            </FormField>
            <FormField label="Last Name" error={errors.lastName?.message}>
              <input {...register('lastName')} />
            </FormField>
          </div>
          <FormField label="Email" error={errors.email?.message}>
            <input type="email" {...register('email')} />
          </FormField>
          <FormField label="Phone" error={errors.phone?.message}>
            <input type="tel" {...register('phone')} placeholder="+1 555 123 4567" />
          </FormField>
        </section>
      )}

      {step === 2 && (
        <section>
          <h2>Work Experience</h2>
          {fields.map((field, index) => (
            <ExperienceCard
              key={field.id}
              index={index}
              register={register}
              control={control}
              errors={errors.experiences?.[index]}
              onRemove={() => remove(index)}
              canRemove={fields.length > 1}
            />
          ))}
          <button type="button" onClick={() => append({ company: '', title: '', startDate: '', current: false, description: '' })}>
            + Add Another Position
          </button>
        </section>
      )}

      {step === 3 && (
        <section>
          <h2>Additional Information</h2>
          <FormField label="Expected Salary (USD)" error={errors.salary?.message}>
            <Controller
              name="salary"
              control={control}
              render={({ field }) => (
                <input
                  type="number"
                  {...field}
                  onChange={e => field.onChange(parseInt(e.target.value) || 0)}
                />
              )}
            />
          </FormField>

          <FormField label="When can you start?" error={errors.startAvailability?.message}>
            <select {...register('startAvailability')}>
              <option value="immediately">Immediately</option>
              <option value="2weeks">2 weeks notice</option>
              <option value="1month">1 month notice</option>
              <option value="other">Other</option>
            </select>
          </FormField>

          {availability === 'other' && (
            <FormField label="Please specify" error={errors.customAvailability?.message}>
              <input {...register('customAvailability')} />
            </FormField>
          )}

          <FormField label="How did you hear about us?" error={errors.referralSource?.message}>
            <input {...register('referralSource')} />
          </FormField>
        </section>
      )}

      {step === 4 && (
        <section>
          <h2>Review & Submit</h2>
          <ApplicationSummary control={control} />

          <FormField error={errors.authorizedToWork?.message}>
            <label className="checkbox">
              <input type="checkbox" {...register('authorizedToWork')} />
              I am authorized to work in this country
            </label>
          </FormField>

          <FormField error={errors.agreedToTerms?.message}>
            <label className="checkbox">
              <input type="checkbox" {...register('agreedToTerms')} />
              I agree to the terms and conditions
            </label>
          </FormField>
        </section>
      )}

      <div className="form-actions">
        {step > 1 && <button type="button" onClick={() => setStep(s => s - 1)}>Back</button>}
        {step < 4 && <button type="button" onClick={nextStep}>Continue</button>}
        {step === 4 && <button type="submit" disabled={isSubmitting}>
          {isSubmitting ? 'Submitting...' : 'Submit Application'}
        </button>}
      </div>
    </form>
  );
}
```

---

## Chapter 7: The Patterns in Action

By Friday, the job application portal is live. Sarah walks through what you've built:

**Multi-step form with step validation** - Users can't proceed until current fields are valid.

**Dynamic field arrays** - Work experience section grows and shrinks as needed.

**Conditional fields** - "Other" availability shows a text input.

**Complex validation** - Zod schema handles all rules in one place.

**Controller for non-standard inputs** - Number inputs that need type conversion.

**Progress indicator and navigation** - Users know where they are and can go back.

"This is production-quality form work," she says. "Most developers struggle with forms their whole career. You just learned the right patterns."

---

## Chapter 8: File Uploads

Monday morning. HR has a new request.

"Applicants need to upload their resume," Sarah says. "PDF or Word, max 5MB. And a profile photo."

File uploads add complexity—you're dealing with binary data, not just strings.

### Basic File Upload

```jsx
function ResumeUpload({ control, errors }) {
  const [preview, setPreview] = useState(null);

  return (
    <Controller
      name="resume"
      control={control}
      rules={{
        required: 'Resume is required',
        validate: {
          fileSize: (file) =>
            !file || file.size <= 5 * 1024 * 1024 || 'File must be under 5MB',
          fileType: (file) =>
            !file ||
            ['application/pdf', 'application/msword', 'application/vnd.openxmlformats-officedocument.wordprocessingml.document']
              .includes(file.type) ||
            'Only PDF and Word documents allowed',
        },
      }}
      render={({ field: { onChange, value } }) => (
        <div className="file-upload">
          <input
            type="file"
            accept=".pdf,.doc,.docx"
            onChange={(e) => {
              const file = e.target.files?.[0];
              onChange(file);
              setPreview(file ? file.name : null);
            }}
          />
          {preview && <span className="file-name">{preview}</span>}
          {errors.resume && <span className="error">{errors.resume.message}</span>}
        </div>
      )}
    />
  );
}
```

### Image Upload with Preview

```jsx
function ProfilePhotoUpload({ control, errors }) {
  const [preview, setPreview] = useState(null);

  const handleFileChange = (file, onChange) => {
    onChange(file);

    if (file && file.type.startsWith('image/')) {
      const reader = new FileReader();
      reader.onload = (e) => setPreview(e.target.result);
      reader.readAsDataURL(file);
    } else {
      setPreview(null);
    }
  };

  return (
    <Controller
      name="profilePhoto"
      control={control}
      rules={{
        validate: {
          fileSize: (file) =>
            !file || file.size <= 2 * 1024 * 1024 || 'Image must be under 2MB',
          fileType: (file) =>
            !file ||
            ['image/jpeg', 'image/png', 'image/webp'].includes(file.type) ||
            'Only JPEG, PNG, or WebP images allowed',
        },
      }}
      render={({ field: { onChange } }) => (
        <div className="photo-upload">
          <div className="preview-container">
            {preview ? (
              <img src={preview} alt="Preview" className="photo-preview" />
            ) : (
              <div className="photo-placeholder">
                <span>No photo</span>
              </div>
            )}
          </div>

          <label className="upload-button">
            Choose Photo
            <input
              type="file"
              accept="image/jpeg,image/png,image/webp"
              onChange={(e) => handleFileChange(e.target.files?.[0], onChange)}
              hidden
            />
          </label>

          {errors.profilePhoto && (
            <span className="error">{errors.profilePhoto.message}</span>
          )}
        </div>
      )}
    />
  );
}
```

### Drag and Drop Upload

```jsx
function DragDropUpload({ onFileSelect, accept, maxSize }) {
  const [isDragging, setIsDragging] = useState(false);
  const [error, setError] = useState(null);

  const handleDrag = (e) => {
    e.preventDefault();
    e.stopPropagation();
  };

  const handleDragIn = (e) => {
    handleDrag(e);
    setIsDragging(true);
  };

  const handleDragOut = (e) => {
    handleDrag(e);
    setIsDragging(false);
  };

  const validateAndSelect = (file) => {
    setError(null);

    if (maxSize && file.size > maxSize) {
      setError(`File must be under ${Math.round(maxSize / 1024 / 1024)}MB`);
      return;
    }

    onFileSelect(file);
  };

  const handleDrop = (e) => {
    handleDrag(e);
    setIsDragging(false);

    const file = e.dataTransfer.files?.[0];
    if (file) validateAndSelect(file);
  };

  return (
    <div
      className={`drop-zone ${isDragging ? 'dragging' : ''}`}
      onDragEnter={handleDragIn}
      onDragLeave={handleDragOut}
      onDragOver={handleDrag}
      onDrop={handleDrop}
    >
      <p>Drag file here or click to browse</p>
      <input
        type="file"
        accept={accept}
        onChange={(e) => {
          const file = e.target.files?.[0];
          if (file) validateAndSelect(file);
        }}
      />
      {error && <span className="error">{error}</span>}
    </div>
  );
}
```

---

## Chapter 9: Saving Progress

"What if someone gets interrupted?" Marcus asks. "They've filled out three steps, then their browser crashes. They have to start over?"

You add automatic progress saving.

### Auto-Save to localStorage

```jsx
function useFormPersistence(formKey, { watch, reset }) {
  // Load saved data on mount
  useEffect(() => {
    const saved = localStorage.getItem(formKey);
    if (saved) {
      try {
        const parsed = JSON.parse(saved);
        reset(parsed);
      } catch (e) {
        localStorage.removeItem(formKey);
      }
    }
  }, [formKey, reset]);

  // Save on every change (debounced)
  const formValues = watch();

  useEffect(() => {
    const timeoutId = setTimeout(() => {
      localStorage.setItem(formKey, JSON.stringify(formValues));
    }, 500); // Debounce saves

    return () => clearTimeout(timeoutId);
  }, [formKey, formValues]);

  // Clear saved data
  const clearSavedData = () => {
    localStorage.removeItem(formKey);
  };

  return { clearSavedData };
}
```

### Using the Persistence Hook

```jsx
function JobApplicationForm({ jobId }) {
  const formMethods = useForm({
    resolver: zodResolver(applicationSchema),
    defaultValues: {
      firstName: '',
      lastName: '',
      email: '',
      experiences: [{ company: '', title: '' }],
    },
  });

  const { clearSavedData } = useFormPersistence(
    `job-application-${jobId}`,
    formMethods
  );

  const onSubmit = async (data) => {
    await submitApplication(data);
    clearSavedData(); // Clear saved data on successful submit
  };

  return (
    <FormProvider {...formMethods}>
      <form onSubmit={formMethods.handleSubmit(onSubmit)}>
        {/* Form fields */}
        <p className="auto-save-notice">
          Your progress is automatically saved
        </p>
      </form>
    </FormProvider>
  );
}
```

### Restore Draft Prompt

```jsx
function DraftPrompt({ formKey, onRestore, onDiscard }) {
  const [hasDraft, setHasDraft] = useState(false);

  useEffect(() => {
    const saved = localStorage.getItem(formKey);
    if (saved) setHasDraft(true);
  }, [formKey]);

  if (!hasDraft) return null;

  return (
    <div className="draft-prompt" role="alert">
      <p>You have a saved draft. Would you like to continue?</p>
      <button onClick={onRestore}>Continue Draft</button>
      <button onClick={() => {
        localStorage.removeItem(formKey);
        setHasDraft(false);
        onDiscard();
      }}>
        Start Fresh
      </button>
    </div>
  );
}
```

"Now users can close the tab, come back tomorrow, and pick up right where they left off," Sarah approves.

---

## Validation Patterns Reference

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

  // Conditional validation
  phone: z.string().optional(),
  phoneRequired: z.boolean(),
}).refine(data => !data.phoneRequired || data.phone, {
  message: 'Phone is required',
  path: ['phone'],
});
```

## Common Mistakes

1. **Validating on every keystroke** - Distracting; use blur or submit
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

---

**Navigation**: [← Previous Module](./04-state-management-basics.md) | [Next Module →](./06-styling-approaches.md)
