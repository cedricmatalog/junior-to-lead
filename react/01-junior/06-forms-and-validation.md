# Forms and Validation

> **Last reviewed**: 2026-01-06


## Week Six: The Job Application

The food delivery feature is live. Orders are flowing. But there's a new priority from the business team.

"You're hiring," Sarah says. "A lot. And HR wants an application portal built into the platform."

You've built forms before—simple inputs, maybe a contact form. But this is different. Multi-step wizard. File uploads. Conditional fields. Real-time validation.

"Forms are deceptively hard," Marcus warns. "They look simple until they're not."

He's about to show you why.

## Mental Models

Before you dive in, here's how to think about the core concepts:

| Concept | Think of it as... |
|---------|-------------------|
| **Controlled inputs** | A secretary taking dictation - every keystroke reported, you dictate what appears |
| **Validation** | A bouncer checking IDs - verify data meets requirements before letting it through |
| **Schema validation** | A DMV form template - defines all rules in one place |

Keep these in mind. They'll click as you build.

---

## Prerequisites

Junior Module 04 (State Management Basics) - Understanding state management, useReducer, and Context patterns.

---

## Learning Objectives

By the end of this module, you'll be able to:

- [ ] Build controlled form inputs with React Hook Form for minimal boilerplate
- [ ] Implement schema-based validation using Zod with type safety
- [ ] Display validation errors inline with proper accessibility attributes
- [ ] Create multi-step forms with step-by-step validation
- [ ] Handle file uploads with validation and preview functionality
- [ ] Implement auto-save to localStorage to prevent data loss

---

## Time Estimate

- **Reading**: 60-75 minutes
- **Exercises**: 4-5 hours
- **Mastery**: Practice form patterns over 3-4 weeks across different projects

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

You wire up the form once so you stop fighting state.

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

"For the job application, you need serious validation," Sarah says. "That's where Zod comes in."

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

// TypeScript users: infer types from schema
type FormData = z.infer<typeof schema>;

function RegisterForm() {
  const { register, handleSubmit, formState: { errors } } = useForm<FormData>({
    resolver: zodResolver(schema)
  });

  function onSubmit(data: FormData) {
    console.log(data);
  }

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

Here's the validation flow:

```
┌──────────────────────────────────────────────────────────┐
│           Form Validation Flow                           │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  User types in form                                      │
│       │                                                  │
│       ↓                                                  │
│  React Hook Form captures value                          │
│       │                                                  │
│       ↓                                                  │
│  On blur or submit...                                    │
│       │                                                  │
│       ↓                                                  │
│  ┌──────────────────────────────┐                        │
│  │    Zod Schema Validation     │                        │
│  │  z.object({ ... })           │                        │
│  └──────────────────────────────┘                        │
│       │                                                  │
│       ├─ Valid? ──────────┐                              │
│       │                   │                              │
│      YES                 NO                              │
│       │                   │                              │
│       ↓                   ↓                              │
│  ┌─────────┐        ┌──────────┐                         │
│  │ Submit  │        │  Errors  │                         │
│  │  data   │        │ Object:  │                         │
│  └─────────┘        │ {        │                         │
│       │             │  email:  │                         │
│       ↓             │   "..."  │                         │
│  API call           │ }        │                         │
│       │             └──────────┘                         │
│       │                   │                              │
│       │                   ↓                              │
│       │            Display errors inline                 │
│       │                   │                              │
│       │                   ↓                              │
│       │            Focus first error field               │
│       │                   │                              │
│       ↓                   │                              │
│  Success/Error      User fixes and retries               │
│  feedback                 │                              │
│       │                   │                              │
│       └───────────────────┘                              │
│                                                          │
│  Key: Validation happens BEFORE submission               │
│  Errors shown immediately with clear messages            │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

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

Sarah says, "Schemas keep complex forms from becoming chaos."

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
  salary: z.coerce.number().min(0, 'Salary must be positive'),
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

Marcus says, "Steps reduce cognitive load."

```
┌──────────────────────────────────────────────────────────┐
│          Multi-Step Wizard Flow                          │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  ┌─────────┐    ┌─────────┐    ┌──────────┐    ┌──────┐  │
│  │ Step 1  │ →  │ Step 2  │ →  │ Step 3   │ →  │ Step │  │
│  │Personal │    │Experience│   │Additional│    │  4   │  │
│  │  Info   │    │         │    │          │    │Review│  │
│  └─────────┘    └─────────┘    └──────────┘    └──────┘  │
│      │              │               │              │     │
│      ↓              ↓               ↓              ↓     │
│  Validate      Validate       Validate        Validate   │
│  fields        fields         fields          all +      │
│  for step      for step       for step        submit     │
│      │              │               │              │     │
│      ├─ Valid? ─────┤───── Valid? ──┤──── Valid? ──┤     │
│      │              │               │              │     │
│     YES             YES            YES            YES    │
│      ↓              ↓               ↓              ↓     │
│  Continue       Continue        Continue        Submit   │
│  to Step 2      to Step 3       to Step 4       data     │
│      │              │               │              │     │
│     NO              NO             NO             NO     │
│      ↓              ↓               ↓              ↓     │
│  Show errors   Show errors    Show errors    Show        │
│  Stay on       Stay on        Stay on        errors      │
│  Step 1        Step 2         Step 3         Don't       │
│                                               submit     │
│                                                          │
│  Progress saved in state:                                │
│  { step1Data, step2Data, step3Data, step4Data }          │
│                                                          │
│  Back button:                                            │
│  • Saves current step data                               │
│  • Moves to previous step                                │
│  • Loads previous step data                              │
│                                                          │
│  Auto-save to localStorage (optional):                   │
│  • Saves after each step completion                      │
│  • Restores on page reload                               │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

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

  // useFieldArray manages dynamic form arrays (add, remove, reorder items)
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

### File Uploads

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

### Saving Progress

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


Example:
```jsx
// ❌ Ad-hoc validation scattered in handlers
if (!email.includes('@')) {
  setEmailError('Invalid email');
}

// ✅ Centralized schema validation
const schema = z.object({ email: z.string().email() });
```

---

## Debug This Code

Before moving to the exercises, test your debugging skills. Each snippet has bugs - can you spot and fix them?

<details>
<summary>Challenge 1: Uncontrolled vs Controlled Input</summary>

```jsx
function LoginForm() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');

  const handleSubmit = (e) => {
    e.preventDefault();
    console.log({ email, password });
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="email"
        placeholder="Email"
        onChange={(e) => setEmail(e.target.value)}
      />
      <input
        type="password"
        value={password}
        placeholder="Password"
      />
      <button type="submit">Login</button>
    </form>
  );
}
```

**How many bugs can you find?** (Answer: 2 bugs)

<details>
<summary>Hint</summary>

Check which inputs are controlled vs uncontrolled. Are all inputs properly connected to state?

</details>

<details>
<summary>Solution</summary>

**Bug 1**: Email input is uncontrolled - it has `onChange` but no `value` prop, so React doesn't control its value.

**Bug 2**: Password input is read-only - it has `value` but no `onChange`, so users can't type in it.

**Fixed code:**

```jsx
function LoginForm() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');

  const handleSubmit = (e) => {
    e.preventDefault();
    console.log({ email, password });
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
        placeholder="Email"
      />
      <input
        type="password"
        value={password}
        onChange={(e) => setPassword(e.target.value)}
        placeholder="Password"
      />
      <button type="submit">Login</button>
    </form>
  );
}
```

**Why it matters**: Controlled inputs need both `value` and `onChange`. Without `value`, React can't control the input. Without `onChange`, users can't modify it.

</details>

</details>

<details>
<summary>Challenge 2: React Hook Form Registration</summary>

```jsx
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';

const schema = z.object({
  username: z.string().min(3, 'Username must be at least 3 characters'),
  email: z.string().email('Invalid email'),
});

function SignupForm() {
  const { handleSubmit, formState: { errors } } = useForm({
    resolver: zodResolver(schema),
  });

  const onSubmit = (data) => {
    console.log(data);
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input type="text" placeholder="Username" />
      {errors.username && <span>{errors.username.message}</span>}

      <input type="email" placeholder="Email" />
      {errors.email && <span>{errors.email.message}</span>}

      <button type="submit">Sign Up</button>
    </form>
  );
}
```

**How many bugs can you find?** (Answer: 1 bug)

<details>
<summary>Hint</summary>

React Hook Form needs to know which inputs to manage. How does it track them?

</details>

<details>
<summary>Solution</summary>

**Bug**: Inputs are not registered with React Hook Form - missing `{...register('fieldName')}` spread on each input.

**Fixed code:**

```jsx
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';

const schema = z.object({
  username: z.string().min(3, 'Username must be at least 3 characters'),
  email: z.string().email('Invalid email'),
});

function SignupForm() {
  const { register, handleSubmit, formState: { errors } } = useForm({
    resolver: zodResolver(schema),
  });

  const onSubmit = (data) => {
    console.log(data);
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input
        type="text"
        placeholder="Username"
        {...register('username')}
      />
      {errors.username && <span>{errors.username.message}</span>}

      <input
        type="email"
        placeholder="Email"
        {...register('email')}
      />
      {errors.email && <span>{errors.email.message}</span>}

      <button type="submit">Sign Up</button>
    </form>
  );
}
```

**Why it matters**: React Hook Form's `register` function connects inputs to the form state. Without it, RHF doesn't know these inputs exist and won't validate or collect their values.

</details>

</details>

<details>
<summary>Challenge 3: Zod Schema Validation Logic</summary>

```jsx
import { z } from 'zod';

const passwordSchema = z.object({
  password: z
    .string()
    .min(8, 'Password must be at least 8 characters')
    .regex(/[A-Z]/, 'Password must contain an uppercase letter')
    .regex(/[0-9]/, 'Password must contain a number'),
  confirmPassword: z.string(),
}).refine(data => data.password === data.confirmPassword, {
  message: 'Passwords must match',
  path: 'confirmPassword',
});

function PasswordForm() {
  const { register, handleSubmit, formState: { errors } } = useForm({
    resolver: zodResolver(passwordSchema),
  });

  return (
    <form onSubmit={handleSubmit(console.log)}>
      <input type="password" {...register('password')} />
      {errors.password && <span>{errors.password.message}</span>}

      <input type="password" {...register('confirmPassword')} />
      {errors.confirmPassword && <span>{errors.confirmPassword.message}</span>}

      <button type="submit">Submit</button>
    </form>
  );
}
```

**How many bugs can you find?** (Answer: 1 bug)

<details>
<summary>Hint</summary>

Look at the refine error path. Is it pointing to the right field?

</details>

<details>
<summary>Solution</summary>

**Bug**: The `path` in the refine should be an array, not a string.

**Fixed code:**

```jsx
import { z } from 'zod';

const passwordSchema = z.object({
  password: z
    .string()
    .min(8, 'Password must be at least 8 characters')
    .regex(/[A-Z]/, 'Password must contain an uppercase letter')
    .regex(/[0-9]/, 'Password must contain a number'),
  confirmPassword: z.string(),
}).refine(data => data.password === data.confirmPassword, {
  message: 'Passwords must match',
  path: ['confirmPassword'],  // ✅ Array, not string
});

function PasswordForm() {
  const { register, handleSubmit, formState: { errors } } = useForm({
    resolver: zodResolver(passwordSchema),
  });

  return (
    <form onSubmit={handleSubmit(console.log)}>
      <input type="password" {...register('password')} />
      {errors.password && <span>{errors.password.message}</span>}

      <input type="password" {...register('confirmPassword')} />
      {errors.confirmPassword && <span>{errors.confirmPassword.message}</span>}

      <button type="submit">Submit</button>
    </form>
  );
}
```

**Why it matters**: Zod's `path` in refinements should be an array indicating which field the error belongs to. A string might work in some cases but isn't the correct type and can cause issues.

</details>

</details>

---

## Practice Exercises

1. Build a multi-step registration form with validation
2. Create a dynamic form that adds/removes fields
3. Implement async validation (e.g., check if username exists)

### Solutions

<details>
<summary>Exercise 1: Multi-Step Registration Form</summary>

```jsx
import { useCallback, useEffect, useMemo, useState } from 'react';
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';

// Step schemas
const step1Schema = z.object({
  email: z.string().email('Invalid email address'),
  password: z.string().min(8, 'Password must be at least 8 characters'),
  confirmPassword: z.string(),
}).refine(data => data.password === data.confirmPassword, {
  message: "Passwords don't match",
  path: ['confirmPassword'],
});

const step2Schema = z.object({
  firstName: z.string().min(1, 'First name is required'),
  lastName: z.string().min(1, 'Last name is required'),
  dateOfBirth: z.string().min(1, 'Date of birth is required'),
});

const step3Schema = z.object({
  address: z.string().min(1, 'Address is required'),
  city: z.string().min(1, 'City is required'),
  zipCode: z.string().regex(/^\d{5}$/, 'Must be 5 digits'),
  terms: z.boolean().refine(val => val === true, 'You must accept the terms'),
});

function MultiStepRegistration() {
  const [step, setStep] = useState(1);
  const [formData, setFormData] = useState({});

  const schemas = {
    1: step1Schema,
    2: step2Schema,
    3: step3Schema,
  };

  const {
    register,
    handleSubmit,
    formState: { errors },
    trigger,
  } = useForm({
    resolver: zodResolver(schemas[step]),
    defaultValues: formData,
  });

  const onNext = async (data) => {
    const isValid = await trigger();
    if (isValid) {
      setFormData(prev => ({ ...prev, ...data }));
      setStep(s => s + 1);
    }
  };

  const onBack = (data) => {
    setFormData(prev => ({ ...prev, ...data }));
    setStep(s => s - 1);
  };

  const onSubmit = async (data) => {
    const finalData = { ...formData, ...data };
    console.log('Submitting:', finalData);
    // Submit to API
  };

  return (
    <div className="registration-form">
      <div className="progress-bar">
        <div className="progress" style={{ width: `${(step / 3) * 100}%` }} />
      </div>

      <h2>Step {step} of 3</h2>

      {step === 1 && (
        <form onSubmit={handleSubmit(onNext)}>
          <div>
            <label htmlFor="email">Email</label>
            <input id="email" type="email" {...register('email')} />
            {errors.email && <span className="error">{errors.email.message}</span>}
          </div>

          <div>
            <label htmlFor="password">Password</label>
            <input id="password" type="password" {...register('password')} />
            {errors.password && <span className="error">{errors.password.message}</span>}
          </div>

          <div>
            <label htmlFor="confirmPassword">Confirm Password</label>
            <input id="confirmPassword" type="password" {...register('confirmPassword')} />
            {errors.confirmPassword && <span className="error">{errors.confirmPassword.message}</span>}
          </div>

          <button type="submit">Next</button>
        </form>
      )}

      {step === 2 && (
        <form onSubmit={handleSubmit(onNext)}>
          <div>
            <label htmlFor="firstName">First Name</label>
            <input id="firstName" {...register('firstName')} />
            {errors.firstName && <span className="error">{errors.firstName.message}</span>}
          </div>

          <div>
            <label htmlFor="lastName">Last Name</label>
            <input id="lastName" {...register('lastName')} />
            {errors.lastName && <span className="error">{errors.lastName.message}</span>}
          </div>

          <div>
            <label htmlFor="dateOfBirth">Date of Birth</label>
            <input id="dateOfBirth" type="date" {...register('dateOfBirth')} />
            {errors.dateOfBirth && <span className="error">{errors.dateOfBirth.message}</span>}
          </div>

          <button type="button" onClick={handleSubmit(onBack)}>Back</button>
          <button type="submit">Next</button>
        </form>
      )}

      {step === 3 && (
        <form onSubmit={handleSubmit(onSubmit)}>
          <div>
            <label htmlFor="address">Address</label>
            <input id="address" {...register('address')} />
            {errors.address && <span className="error">{errors.address.message}</span>}
          </div>

          <div>
            <label htmlFor="city">City</label>
            <input id="city" {...register('city')} />
            {errors.city && <span className="error">{errors.city.message}</span>}
          </div>

          <div>
            <label htmlFor="zipCode">ZIP Code</label>
            <input id="zipCode" {...register('zipCode')} />
            {errors.zipCode && <span className="error">{errors.zipCode.message}</span>}
          </div>

          <div>
            <label>
              <input type="checkbox" {...register('terms')} />
              I agree to the terms and conditions
            </label>
            {errors.terms && <span className="error">{errors.terms.message}</span>}
          </div>

          <button type="button" onClick={handleSubmit(onBack)}>Back</button>
          <button type="submit">Submit</button>
        </form>
      )}
    </div>
  );
}
```

**Key points:**
- Separate schemas for each step allow progressive validation
- Form data accumulates across steps in state
- trigger() validates current step before proceeding
- Progress bar provides visual feedback
- Back button preserves entered data

</details>

<details>
<summary>Exercise 2: Dynamic Form with Add/Remove Fields</summary>

```jsx
import { useForm, useFieldArray } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';

const schema = z.object({
  teamName: z.string().min(1, 'Team name is required'),
  members: z.array(
    z.object({
      name: z.string().min(1, 'Name is required'),
      email: z.string().email('Invalid email'),
      role: z.enum(['developer', 'designer', 'manager'], {
        errorMap: () => ({ message: 'Please select a role' }),
      }),
    })
  ).min(1, 'At least one team member is required'),
});

function TeamForm() {
  const {
    register,
    control,
    handleSubmit,
    formState: { errors },
  } = useForm({
    resolver: zodResolver(schema),
    defaultValues: {
      teamName: '',
      members: [{ name: '', email: '', role: 'developer' }],
    },
  });

  const { fields, append, remove } = useFieldArray({
    control,
    name: 'members',
  });

  const onSubmit = (data) => {
    console.log('Team data:', data);
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <div>
        <label htmlFor="teamName">Team Name</label>
        <input id="teamName" {...register('teamName')} />
        {errors.teamName && <span className="error">{errors.teamName.message}</span>}
      </div>

      <h3>Team Members</h3>
      {errors.members?.root && (
        <span className="error">{errors.members.root.message}</span>
      )}

      {fields.map((field, index) => (
        <div key={field.id} className="member-card">
          <h4>Member {index + 1}</h4>

          <div>
            <label htmlFor={`members.${index}.name`}>Name</label>
            <input
              id={`members.${index}.name`}
              {...register(`members.${index}.name`)}
            />
            {errors.members?.[index]?.name && (
              <span className="error">{errors.members[index].name.message}</span>
            )}
          </div>

          <div>
            <label htmlFor={`members.${index}.email`}>Email</label>
            <input
              id={`members.${index}.email`}
              type="email"
              {...register(`members.${index}.email`)}
            />
            {errors.members?.[index]?.email && (
              <span className="error">{errors.members[index].email.message}</span>
            )}
          </div>

          <div>
            <label htmlFor={`members.${index}.role`}>Role</label>
            <select
              id={`members.${index}.role`}
              {...register(`members.${index}.role`)}
            >
              <option value="">Select role...</option>
              <option value="developer">Developer</option>
              <option value="designer">Designer</option>
              <option value="manager">Manager</option>
            </select>
            {errors.members?.[index]?.role && (
              <span className="error">{errors.members[index].role.message}</span>
            )}
          </div>

          {fields.length > 1 && (
            <button type="button" onClick={() => remove(index)}>
              Remove Member
            </button>
          )}
        </div>
      ))}

      <button
        type="button"
        onClick={() => append({ name: '', email: '', role: 'developer' })}
      >
        + Add Team Member
      </button>

      <button type="submit">Create Team</button>
    </form>
  );
}
```

**Key points:**
- useFieldArray manages dynamic arrays of form fields
- Each field gets unique ID for proper React reconciliation
- Remove button disabled when only one member remains
- Validation applies to each array item individually
- Array-level validation ensures at least one member exists

</details>

<details>
<summary>Exercise 3: Async Validation (Username Availability)</summary>

```jsx
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';
import { useState } from 'react';

// Simulate API call
const checkUsernameAvailability = async (username) => {
  await new Promise(resolve => setTimeout(resolve, 500));
  const taken = ['admin', 'user', 'test'];
  return !taken.includes(username.toLowerCase());
};

const schema = z.object({
  username: z
    .string()
    .min(3, 'Username must be at least 3 characters')
    .regex(/^[a-zA-Z0-9_]+$/, 'Only letters, numbers, and underscores allowed'),
  email: z.string().email('Invalid email address'),
  password: z.string().min(8, 'Password must be at least 8 characters'),
});

function RegistrationForm() {
  const [usernameChecking, setUsernameChecking] = useState(false);
  const [usernameAvailable, setUsernameAvailable] = useState(null);

  const {
    register,
    handleSubmit,
    formState: { errors, isSubmitting },
    setError,
    clearErrors,
  } = useForm({
    resolver: zodResolver(schema),
  });

  // Debounced async username check
  const checkUsername = useCallback(async (value) => {
    if (!value || value.length < 3) {
      setUsernameAvailable(null);
      return;
    }

    setUsernameChecking(true);
    clearErrors('username');

    try {
      const isAvailable = await checkUsernameAvailability(value);

      if (isAvailable) {
        setUsernameAvailable(true);
      } else {
        setUsernameAvailable(false);
        setError('username', {
          type: 'manual',
          message: 'Username is already taken',
        });
      }
    } catch (error) {
      setError('username', {
        type: 'manual',
        message: 'Failed to check username availability',
      });
    } finally {
      setUsernameChecking(false);
    }
  }, [clearErrors, setError]);

  // Debounce helper
  const debounce = (func, delay) => {
    let timeoutId;
    const debounced = (...args) => {
      clearTimeout(timeoutId);
      timeoutId = setTimeout(() => func(...args), delay);
    };
    debounced.cancel = () => clearTimeout(timeoutId);
    return debounced;
  };

  const debouncedCheck = useMemo(
    () => debounce(checkUsername, 500),
    [checkUsername]
  );

  useEffect(() => {
    return () => debouncedCheck.cancel();
  }, [debouncedCheck]);

  const onSubmit = async (data) => {
    // Final check before submission
    if (!usernameAvailable) {
      setError('username', {
        type: 'manual',
        message: 'Please choose an available username',
      });
      return;
    }

    console.log('Submitting:', data);
    // Submit to API
  };

  const usernameField = register('username');

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <div>
        <label htmlFor="username">Username</label>
        <div className="input-with-status">
          <input
            id="username"
            {...usernameField}
            onChange={(e) => {
              usernameField.onChange(e);
              debouncedCheck(e.target.value);
            }}
          />
          {usernameChecking && <span className="checking">Checking...</span>}
          {usernameAvailable === true && !usernameChecking && (
            <span className="available">✓ Available</span>
          )}
        </div>
        {errors.username && <span className="error">{errors.username.message}</span>}
      </div>

      <div>
        <label htmlFor="email">Email</label>
        <input id="email" type="email" {...register('email')} />
        {errors.email && <span className="error">{errors.email.message}</span>}
      </div>

      <div>
        <label htmlFor="password">Password</label>
        <input id="password" type="password" {...register('password')} />
        {errors.password && <span className="error">{errors.password.message}</span>}
      </div>

      <button type="submit" disabled={isSubmitting || usernameChecking}>
        {isSubmitting ? 'Registering...' : 'Register'}
      </button>
    </form>
  );
}
```

**Key points:**
- Debouncing prevents excessive API calls while typing
- Visual feedback shows checking/available/taken states
- setError allows manual validation errors from async checks
- Final validation on submit ensures username is still available
- Disable submit while checking to prevent race conditions

</details>

---

## What You Learned

This module covered:

- **React Hook Form**: Simplifies form state management with register, handleSubmit, and formState
- **Zod Validation**: Type-safe schema validation with clear error messages and type inference
- **Error Display**: Inline error messages with aria-invalid and role="alert" for accessibility
- **Multi-Step Forms**: Validating each step independently before allowing progression
- **File Uploads**: Handling binary data with validation, previews, and proper error states
- **Form Persistence**: Auto-saving to localStorage with debouncing to prevent data loss

**Key takeaway**: React Hook Form + Zod creates a powerful, type-safe form solution with minimal boilerplate and excellent developer experience.

---

## Real-World Application

This week at work, you might use these concepts to:

- Build a job application form with resume upload and multi-step progression
- Create a user registration flow with real-time password strength validation
- Implement a settings page with auto-save functionality
- Design a checkout form with address validation and payment processing
- Build a survey form with conditional questions based on previous answers

---

## Further Reading

- [React Hook Form Documentation](https://react-hook-form.com/)
- [Zod Documentation](https://zod.dev/)

---

**Navigation**: [← Previous Module](./05-routing-and-navigation.md) | [Next Module →](./07-styling-approaches.md)
