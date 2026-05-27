# 09 — Forms & Validation in React (Deep Dive)

> **Course:** React — From Beginner to Production
> **Topic:** Forms & Validation
> **Level:** Beginner → Advanced
> **Prerequisites:** State & useState (Module 04), Event Handling (Module 06), useEffect (Module 05)

---

## Table of Contents

1. [Forms in React — The Big Picture](#1-forms-in-react--the-big-picture)
2. [Controlled Inputs — Complete Guide](#2-controlled-inputs--complete-guide)
3. [Uncontrolled Inputs & useRef](#3-uncontrolled-inputs--useref)
4. [Generic Change Handler Pattern](#4-generic-change-handler-pattern)
5. [Form Validation — Strategies & Patterns](#5-form-validation--strategies--patterns)
6. [Validation Timing — When to Validate](#6-validation-timing--when-to-validate)
7. [Building a useForm Custom Hook](#7-building-a-useform-custom-hook)
8. [React Hook Form — Production Standard](#8-react-hook-form--production-standard)
9. [Schema Validation with Zod](#9-schema-validation-with-zod)
10. [React Hook Form + Zod Integration](#10-react-hook-form--zod-integration)
11. [Complex Form Patterns](#11-complex-form-patterns)
12. [Accessibility in Forms](#12-accessibility-in-forms)
13. [Code Examples (Beginner → Advanced)](#13-code-examples-beginner--advanced)
14. [Real-World Use Cases](#14-real-world-use-cases)
15. [Best Practices](#15-best-practices)
16. [Common Mistakes](#16-common-mistakes)
17. [Performance Considerations](#17-performance-considerations)
18. [Interview Questions](#18-interview-questions)
19. [Practice Tasks](#19-practice-tasks)
20. [Summary](#20-summary)

---

## 1. Forms in React — The Big Picture

### Why Forms are Different in React

In plain HTML, a form manages its own state — the browser tracks what the user typed in each field. When you submit, you read the values directly from the DOM.

React introduces a different model: **you** manage the form state in JavaScript, and the inputs display whatever state you give them. This gives you full control — you can validate in real time, transform input, conditionally show/hide fields, and submit data in any format.

### The Three Approaches

| Approach | When to Use | Complexity |
|----------|------------|------------|
| **Controlled (vanilla React)** | Small/medium forms, full control needed | Medium |
| **Custom useForm hook** | Reusable logic across many forms | Medium |
| **React Hook Form** | Large/complex forms, production apps | Low (after learning) |

### The Mental Model

```
User types → onChange fires → setState updates → React re-renders → input shows new value

                    ┌─────────────────────────────────┐
                    │         React State              │
                    │  { email: "user@example.com" }  │
                    └──────────────┬──────────────────┘
                                   │ value={email}
                    ┌──────────────▼──────────────────┐
                    │    <input type="email" />        │
                    └──────────────┬──────────────────┘
                                   │ onChange → setEmail
                    ┌──────────────▼──────────────────┐
                    │         User types               │
                    └─────────────────────────────────┘
```

---

## 2. Controlled Inputs — Complete Guide

### Text Input

```jsx
const [name, setName] = useState('');

<input
  type="text"
  value={name}
  onChange={e => setName(e.target.value)}
  placeholder="Enter your name"
/>
```

### Email, Password, Number

```jsx
const [email, setEmail]       = useState('');
const [password, setPassword] = useState('');
const [age, setAge]           = useState('');

<input type="email"    value={email}    onChange={e => setEmail(e.target.value)} />
<input type="password" value={password} onChange={e => setPassword(e.target.value)} />
<input
  type="number"
  value={age}
  onChange={e => setAge(e.target.value)}
  min={0}
  max={120}
/>
```

### Textarea

```jsx
const [bio, setBio] = useState('');

<textarea
  value={bio}
  onChange={e => setBio(e.target.value)}
  rows={4}
  maxLength={500}
  placeholder="Tell us about yourself..."
/>
<p>{bio.length}/500</p>
```

### Select (Single)

```jsx
const [country, setCountry] = useState('');

<select value={country} onChange={e => setCountry(e.target.value)}>
  <option value="">-- Select Country --</option>
  <option value="IN">India</option>
  <option value="US">United States</option>
  <option value="UK">United Kingdom</option>
</select>
```

### Select (Multiple)

```jsx
const [selectedSkills, setSelectedSkills] = useState([]);

function handleMultiSelect(e) {
  const selected = Array.from(e.target.selectedOptions, opt => opt.value);
  setSelectedSkills(selected);
}

<select multiple value={selectedSkills} onChange={handleMultiSelect}>
  <option value="react">React</option>
  <option value="node">Node.js</option>
  <option value="python">Python</option>
  <option value="sql">SQL</option>
</select>
<p>Selected: {selectedSkills.join(', ')}</p>
```

### Checkbox (Single)

```jsx
const [agreed, setAgreed] = useState(false);

<label>
  <input
    type="checkbox"
    checked={agreed}
    onChange={e => setAgreed(e.target.checked)}  // use .checked not .value
  />
  I agree to the Terms and Conditions
</label>
```

### Checkbox Group (Multiple)

```jsx
const [interests, setInterests] = useState([]);

const INTEREST_OPTIONS = ['Sports', 'Music', 'Travel', 'Technology', 'Art'];

function handleInterestChange(e) {
  const { value, checked } = e.target;
  setInterests(prev =>
    checked
      ? [...prev, value]
      : prev.filter(i => i !== value)
  );
}

<fieldset>
  <legend>Your Interests</legend>
  {INTEREST_OPTIONS.map(option => (
    <label key={option}>
      <input
        type="checkbox"
        value={option}
        checked={interests.includes(option)}
        onChange={handleInterestChange}
      />
      {option}
    </label>
  ))}
</fieldset>
```

### Radio Buttons

```jsx
const [gender, setGender] = useState('');

const GENDER_OPTIONS = [
  { value: 'male',   label: 'Male'   },
  { value: 'female', label: 'Female' },
  { value: 'other',  label: 'Other'  },
];

<fieldset>
  <legend>Gender</legend>
  {GENDER_OPTIONS.map(opt => (
    <label key={opt.value}>
      <input
        type="radio"
        name="gender"
        value={opt.value}
        checked={gender === opt.value}
        onChange={e => setGender(e.target.value)}
      />
      {opt.label}
    </label>
  ))}
</fieldset>
```

### Range Slider

```jsx
const [volume, setVolume] = useState(50);

<label>
  Volume: {volume}%
  <input
    type="range"
    min={0}
    max={100}
    step={5}
    value={volume}
    onChange={e => setVolume(Number(e.target.value))}
  />
</label>
```

### File Input (Special Case)

File inputs are **always uncontrolled** — you cannot set their `value` from React. Use a `ref` to read files:

```jsx
const [fileName, setFileName] = useState('');
const fileRef = useRef(null);

function handleFileChange(e) {
  const file = e.target.files[0];
  if (file) setFileName(file.name);
}

<input
  ref={fileRef}
  type="file"
  accept=".pdf,.docx"
  onChange={handleFileChange}
/>
{fileName && <p>Selected: {fileName}</p>}
```

---

## 3. Uncontrolled Inputs & useRef

### What Are Uncontrolled Inputs?

Uncontrolled inputs let the DOM manage the value. You read the value via a `ref` when needed (e.g., on submit). There are no `onChange` handlers or state updates on every keystroke.

```jsx
import { useRef } from 'react';

function SimpleLoginForm() {
  const emailRef    = useRef(null);
  const passwordRef = useRef(null);

  function handleSubmit(e) {
    e.preventDefault();
    const email    = emailRef.current.value;
    const password = passwordRef.current.value;
    console.log({ email, password });
  }

  return (
    <form onSubmit={handleSubmit}>
      <input ref={emailRef}    type="email"    defaultValue="" placeholder="Email" />
      <input ref={passwordRef} type="password" defaultValue="" placeholder="Password" />
      <button type="submit">Login</button>
    </form>
  );
}
```

### defaultValue vs value

```jsx
// Controlled — React drives value, reflects state on every render
<input value={name} onChange={e => setName(e.target.value)} />

// Uncontrolled — DOM drives value, React sets it only once (on mount)
<input defaultValue="Initial Value" ref={inputRef} />

// ❌ Don't mix — you'll get a React warning
<input value={name} defaultValue="Initial" />
```

### When to Use Uncontrolled

- Integrating with non-React code or third-party libraries that directly manipulate DOM inputs
- Very simple one-off forms where real-time validation isn't needed
- File inputs (always uncontrolled)
- Performance-critical forms where you want zero re-renders on every keystroke

---

## 4. Generic Change Handler Pattern

Managing separate state and `onChange` for every field is repetitive. A **generic handler** uses the input's `name` attribute as a dynamic key:

```jsx
function RegistrationForm() {
  const [form, setForm] = useState({
    firstName:   '',
    lastName:    '',
    email:       '',
    password:    '',
    phone:       '',
    dateOfBirth: '',
    gender:      '',
    country:     '',
    newsletter:  false,
  });

  // Generic handler — works for text, email, select, and checkbox
  function handleChange(e) {
    const { name, value, type, checked } = e.target;
    setForm(prev => ({
      ...prev,
      [name]: type === 'checkbox' ? checked : value,
    }));
  }

  return (
    <form>
      <input name="firstName"   value={form.firstName}   onChange={handleChange} placeholder="First Name" />
      <input name="lastName"    value={form.lastName}    onChange={handleChange} placeholder="Last Name" />
      <input name="email"       value={form.email}       onChange={handleChange} type="email" />
      <input name="password"    value={form.password}    onChange={handleChange} type="password" />
      <input name="phone"       value={form.phone}       onChange={handleChange} type="tel" />
      <input name="dateOfBirth" value={form.dateOfBirth} onChange={handleChange} type="date" />

      <select name="gender" value={form.gender} onChange={handleChange}>
        <option value="">Select Gender</option>
        <option value="male">Male</option>
        <option value="female">Female</option>
      </select>

      <select name="country" value={form.country} onChange={handleChange}>
        <option value="">Select Country</option>
        <option value="IN">India</option>
        <option value="US">United States</option>
      </select>

      <label>
        <input
          type="checkbox"
          name="newsletter"
          checked={form.newsletter}
          onChange={handleChange}
        />
        Subscribe to newsletter
      </label>
    </form>
  );
}
```

---

## 5. Form Validation — Strategies & Patterns

### Strategy 1: Inline Validation Functions

Write dedicated validation functions that return error messages or null:

```jsx
// Individual field validators
function validateEmail(value) {
  if (!value.trim()) return 'Email is required';
  if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value)) return 'Invalid email format';
  return null;
}

function validatePassword(value) {
  if (!value) return 'Password is required';
  if (value.length < 8) return 'Password must be at least 8 characters';
  if (!/[A-Z]/.test(value)) return 'Must contain at least one uppercase letter';
  if (!/[0-9]/.test(value)) return 'Must contain at least one number';
  if (!/[!@#$%^&*]/.test(value)) return 'Must contain at least one special character';
  return null;
}

function validateName(value, fieldName = 'Name') {
  if (!value.trim()) return `${fieldName} is required`;
  if (value.trim().length < 2) return `${fieldName} must be at least 2 characters`;
  if (value.trim().length > 50) return `${fieldName} must be under 50 characters`;
  if (!/^[a-zA-Z\s'-]+$/.test(value)) return `${fieldName} can only contain letters`;
  return null;
}

function validatePhone(value) {
  if (!value.trim()) return 'Phone number is required';
  if (!/^[6-9]\d{9}$/.test(value.replace(/\s/g, '')))
    return 'Enter a valid 10-digit Indian mobile number';
  return null;
}

function validateAge(value) {
  const age = Number(value);
  if (!value) return 'Date of birth is required';
  const birthDate = new Date(value);
  const today = new Date();
  const years = today.getFullYear() - birthDate.getFullYear();
  if (years < 18) return 'You must be at least 18 years old';
  if (years > 120) return 'Please enter a valid date';
  return null;
}
```

### Strategy 2: Schema-Based Validation Object

```jsx
// Centralize all field rules in one object
const VALIDATION_SCHEMA = {
  firstName: (v) => {
    if (!v.trim()) return 'First name is required';
    if (v.trim().length < 2) return 'Minimum 2 characters';
    return null;
  },
  email: (v) => {
    if (!v.trim()) return 'Email is required';
    if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(v)) return 'Invalid email';
    return null;
  },
  password: (v) => {
    if (!v) return 'Password is required';
    if (v.length < 8) return 'Minimum 8 characters';
    return null;
  },
  confirmPassword: (v, formValues) => {
    if (!v) return 'Please confirm your password';
    if (v !== formValues.password) return 'Passwords do not match';
    return null;
  },
};

// Validate entire form
function validateForm(values) {
  const errors = {};
  for (const [field, validator] of Object.entries(VALIDATION_SCHEMA)) {
    const error = validator(values[field], values);
    if (error) errors[field] = error;
  }
  return errors;
}
```

### Strategy 3: Real-Time Password Strength

```jsx
function getPasswordStrength(password) {
  if (!password) return { score: 0, label: '', color: '' };

  let score = 0;
  const checks = {
    length:    password.length >= 8,
    uppercase: /[A-Z]/.test(password),
    lowercase: /[a-z]/.test(password),
    number:    /[0-9]/.test(password),
    special:   /[!@#$%^&*(),.?":{}|<>]/.test(password),
    long:      password.length >= 12,
  };

  score = Object.values(checks).filter(Boolean).length;

  if (score <= 2) return { score, label: 'Weak',   color: '#ef4444', width: '25%'  };
  if (score <= 3) return { score, label: 'Fair',   color: '#f97316', width: '50%'  };
  if (score <= 4) return { score, label: 'Good',   color: '#eab308', width: '75%'  };
  return           { score, label: 'Strong', color: '#22c55e', width: '100%' };
}

function PasswordStrengthBar({ password }) {
  const strength = getPasswordStrength(password);
  if (!password) return null;

  return (
    <div className="password-strength">
      <div className="strength-bar">
        <div
          className="strength-bar__fill"
          style={{ width: strength.width, backgroundColor: strength.color, transition: 'all 0.3s' }}
        />
      </div>
      <span style={{ color: strength.color, fontSize: '12px' }}>
        {strength.label}
      </span>
    </div>
  );
}
```

---

## 6. Validation Timing — When to Validate

Different validation strategies create different user experiences:

### On Submit Only

```jsx
// Simplest — validate only when user clicks submit
function handleSubmit(e) {
  e.preventDefault();
  const errors = validateForm(form);
  setErrors(errors);
  if (Object.keys(errors).length > 0) return;
  submitForm(form);
}
```

**UX:** Clean form with no distraction while typing. User only sees errors after trying to submit.

### On Blur (After Leaving Field)

```jsx
const [touched, setTouched] = useState({});

function handleBlur(e) {
  const { name } = e.target;
  setTouched(prev => ({ ...prev, [name]: true }));
}

// Show error only if field has been touched
function getFieldError(name) {
  if (!touched[name]) return null;
  return validateField(name, form[name]);
}
```

**UX:** User gets feedback after they've finished with a field — not while actively typing.

### On Change (Real-Time)

```jsx
function handleChange(e) {
  const { name, value } = e.target;
  setForm(prev => ({ ...prev, [name]: value }));

  // Validate immediately on every keystroke
  const error = validateField(name, value);
  setErrors(prev => ({ ...prev, [name]: error }));
}
```

**UX:** Instant feedback — great for password strength but can feel aggressive for email.

### Hybrid Strategy (Best UX — Industry Standard)

```jsx
// Validate on blur (first time), then validate on change (after first touch)
function handleChange(e) {
  const { name, value } = e.target;
  setForm(prev => ({ ...prev, [name]: value }));

  // Only show real-time errors after the field has been touched
  if (touched[name]) {
    const error = validateField(name, value);
    setErrors(prev => ({ ...prev, [name]: error }));
  }
}

function handleBlur(e) {
  const { name, value } = e.target;
  setTouched(prev => ({ ...prev, [name]: true }));
  const error = validateField(name, value);
  setErrors(prev => ({ ...prev, [name]: error }));
}
```

**UX:** No errors shown while typing for the first time. After blur, errors shown + updated in real-time.

---

## 7. Building a useForm Custom Hook

A reusable hook that encapsulates all form state, change handling, blur tracking, and validation:

```jsx
// File: hooks/useForm.js
import { useState, useCallback } from 'react';

function useForm(initialValues, validationSchema = {}) {
  const [values, setValues]   = useState(initialValues);
  const [errors, setErrors]   = useState({});
  const [touched, setTouched] = useState({});
  const [isSubmitting, setIsSubmitting] = useState(false);

  // Validate a single field
  const validateField = useCallback((name, value) => {
    const validator = validationSchema[name];
    if (!validator) return null;
    return validator(value, values);
  }, [validationSchema, values]);

  // Validate all fields
  const validateAll = useCallback(() => {
    const newErrors = {};
    for (const [name, validator] of Object.entries(validationSchema)) {
      const error = validator(values[name], values);
      if (error) newErrors[name] = error;
    }
    return newErrors;
  }, [validationSchema, values]);

  // Generic change handler
  const handleChange = useCallback((e) => {
    const { name, value, type, checked } = e.target;
    const newValue = type === 'checkbox' ? checked : value;

    setValues(prev => ({ ...prev, [name]: newValue }));

    // Update error if field already touched
    if (touched[name]) {
      const error = validationSchema[name]?.(newValue, { ...values, [name]: newValue });
      setErrors(prev => ({ ...prev, [name]: error || '' }));
    }
  }, [touched, validationSchema, values]);

  // Blur handler — mark field as touched and validate
  const handleBlur = useCallback((e) => {
    const { name, value } = e.target;
    setTouched(prev => ({ ...prev, [name]: true }));
    const error = validateField(name, value);
    setErrors(prev => ({ ...prev, [name]: error || '' }));
  }, [validateField]);

  // Submit handler factory
  const handleSubmit = useCallback((onSubmit) => async (e) => {
    e.preventDefault();

    // Mark all fields as touched
    const allTouched = Object.keys(initialValues).reduce(
      (acc, key) => ({ ...acc, [key]: true }), {}
    );
    setTouched(allTouched);

    // Validate all
    const validationErrors = validateAll();
    setErrors(validationErrors);

    if (Object.keys(validationErrors).length > 0) return;

    setIsSubmitting(true);
    try {
      await onSubmit(values);
    } finally {
      setIsSubmitting(false);
    }
  }, [initialValues, validateAll, values]);

  // Reset form to initial state
  const reset = useCallback(() => {
    setValues(initialValues);
    setErrors({});
    setTouched({});
    setIsSubmitting(false);
  }, [initialValues]);

  // Set a field value programmatically
  const setValue = useCallback((name, value) => {
    setValues(prev => ({ ...prev, [name]: value }));
  }, []);

  // Set multiple values at once
  const setMultipleValues = useCallback((newValues) => {
    setValues(prev => ({ ...prev, ...newValues }));
  }, []);

  // Helper: is a field showing an error?
  const getFieldProps = useCallback((name) => ({
    name,
    value: values[name],
    onChange: handleChange,
    onBlur:   handleBlur,
    'aria-invalid': !!(touched[name] && errors[name]),
    'aria-describedby': errors[name] ? `${name}-error` : undefined,
  }), [values, errors, touched, handleChange, handleBlur]);

  const isValid = Object.keys(validationSchema).every(
    name => !validateField(name, values[name])
  );

  return {
    values,
    errors,
    touched,
    isSubmitting,
    isValid,
    handleChange,
    handleBlur,
    handleSubmit,
    reset,
    setValue,
    setMultipleValues,
    getFieldProps,
  };
}

export default useForm;
```

### Using the useForm Hook

```jsx
import useForm from '../hooks/useForm';

const INITIAL_VALUES = {
  name: '',
  email: '',
  password: '',
  confirmPassword: '',
};

const VALIDATION = {
  name: (v) => !v.trim() ? 'Name is required' : v.length < 2 ? 'Too short' : null,
  email: (v) => !v.trim() ? 'Email required' : !/\S+@\S+\.\S+/.test(v) ? 'Invalid email' : null,
  password: (v) => !v ? 'Password required' : v.length < 8 ? 'Min 8 characters' : null,
  confirmPassword: (v, all) => !v ? 'Required' : v !== all.password ? 'Passwords do not match' : null,
};

function SignupForm() {
  const {
    values, errors, touched, isSubmitting,
    getFieldProps, handleSubmit, reset,
  } = useForm(INITIAL_VALUES, VALIDATION);

  async function onSubmit(formValues) {
    await registerUser(formValues);
    alert('Account created!');
    reset();
  }

  return (
    <form onSubmit={handleSubmit(onSubmit)} noValidate>
      <div className="form-group">
        <label htmlFor="name">Full Name</label>
        <input id="name" type="text" {...getFieldProps('name')} />
        {touched.name && errors.name && (
          <p id="name-error" className="error" role="alert">{errors.name}</p>
        )}
      </div>

      <div className="form-group">
        <label htmlFor="email">Email</label>
        <input id="email" type="email" {...getFieldProps('email')} />
        {touched.email && errors.email && (
          <p id="email-error" className="error" role="alert">{errors.email}</p>
        )}
      </div>

      <div className="form-group">
        <label htmlFor="password">Password</label>
        <input id="password" type="password" {...getFieldProps('password')} />
        <PasswordStrengthBar password={values.password} />
        {touched.password && errors.password && (
          <p id="password-error" className="error" role="alert">{errors.password}</p>
        )}
      </div>

      <div className="form-group">
        <label htmlFor="confirmPassword">Confirm Password</label>
        <input id="confirmPassword" type="password" {...getFieldProps('confirmPassword')} />
        {touched.confirmPassword && errors.confirmPassword && (
          <p id="confirmPassword-error" className="error" role="alert">{errors.confirmPassword}</p>
        )}
      </div>

      <button type="submit" disabled={isSubmitting}>
        {isSubmitting ? 'Creating Account...' : 'Create Account'}
      </button>
    </form>
  );
}
```

---

## 8. React Hook Form — Production Standard

React Hook Form (RHF) is the industry-standard form library for React. It uses **uncontrolled inputs by default** (via refs) to minimize re-renders, making it extremely performant.

```bash
npm install react-hook-form
```

### Core Concepts

| Concept | Description |
|---------|-------------|
| `useForm()` | Main hook — returns register, handleSubmit, formState, etc. |
| `register` | Connects an input to RHF — spreads ref, onChange, onBlur, name |
| `handleSubmit` | Wraps your submit fn — validates before calling |
| `formState` | Object with `errors`, `isSubmitting`, `isDirty`, `isValid`, `touchedFields` |
| `watch` | Subscribe to field value changes |
| `setValue` | Programmatically set a field value |
| `reset` | Reset form to defaults |
| `Controller` | Wrapper for controlled third-party components (Select, DatePicker) |

### Basic Usage

```jsx
import { useForm } from 'react-hook-form';

function LoginForm() {
  const {
    register,
    handleSubmit,
    formState: { errors, isSubmitting },
  } = useForm({
    defaultValues: {
      email: '',
      password: '',
    },
  });

  async function onSubmit(data) {
    // data = { email: '...', password: '...' }
    // Only called when validation passes
    await loginUser(data);
  }

  return (
    <form onSubmit={handleSubmit(onSubmit)} noValidate>
      <div className="form-group">
        <label htmlFor="email">Email</label>
        <input
          id="email"
          type="email"
          {...register('email', {
            required: 'Email is required',
            pattern: {
              value: /^[^\s@]+@[^\s@]+\.[^\s@]+$/,
              message: 'Enter a valid email address',
            },
          })}
          aria-invalid={!!errors.email}
          aria-describedby={errors.email ? 'email-error' : undefined}
        />
        {errors.email && (
          <p id="email-error" role="alert" className="error">
            {errors.email.message}
          </p>
        )}
      </div>

      <div className="form-group">
        <label htmlFor="password">Password</label>
        <input
          id="password"
          type="password"
          {...register('password', {
            required: 'Password is required',
            minLength: { value: 8, message: 'Minimum 8 characters' },
          })}
          aria-invalid={!!errors.password}
        />
        {errors.password && (
          <p role="alert" className="error">{errors.password.message}</p>
        )}
      </div>

      <button type="submit" disabled={isSubmitting}>
        {isSubmitting ? 'Signing in...' : 'Sign In'}
      </button>
    </form>
  );
}
```

### RHF Validation Rules

```jsx
register('fieldName', {
  required: 'This field is required',                    // boolean or string message
  minLength: { value: 3, message: 'Min 3 chars' },
  maxLength: { value: 100, message: 'Max 100 chars' },
  min: { value: 0, message: 'Must be 0 or more' },
  max: { value: 100, message: 'Must be 100 or less' },
  pattern: { value: /regex/, message: 'Invalid format' },
  validate: (value) => value !== 'bad' || 'This value is not allowed',
  // Custom validate with multiple checks:
  validate: {
    notEmpty:     v => v.trim().length > 0 || 'Cannot be empty',
    noSpaces:     v => !/\s/.test(v) || 'No spaces allowed',
    asyncCheck:   async v => {
      const taken = await checkUsernameTaken(v);
      return !taken || 'Username already taken';
    },
  },
})
```

### Watching Field Values

```jsx
const { register, watch } = useForm();

const password = watch('password'); // live value of password field
const allValues = watch();          // watch all fields

// Use watched value for dependent validation
<input
  {...register('confirmPassword', {
    validate: v => v === password || 'Passwords do not match',
  })}
/>
```

### Controller for Third-Party Inputs

```jsx
import { Controller } from 'react-hook-form';
import DatePicker from 'react-datepicker';
import Select from 'react-select';

function AdvancedForm() {
  const { control, handleSubmit } = useForm();

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      {/* DatePicker — uses Controller because it's a controlled component */}
      <Controller
        name="birthDate"
        control={control}
        rules={{ required: 'Date of birth is required' }}
        render={({ field, fieldState }) => (
          <div>
            <DatePicker
              selected={field.value}
              onChange={field.onChange}
              onBlur={field.onBlur}
              placeholderText="Date of Birth"
            />
            {fieldState.error && <p className="error">{fieldState.error.message}</p>}
          </div>
        )}
      />

      {/* React Select */}
      <Controller
        name="skills"
        control={control}
        rules={{ required: 'Select at least one skill' }}
        render={({ field }) => (
          <Select
            {...field}
            isMulti
            options={skillOptions}
            placeholder="Select skills..."
          />
        )}
      />
    </form>
  );
}
```

### useFieldArray — Dynamic Fields

```jsx
import { useForm, useFieldArray } from 'react-hook-form';

function ExperienceForm() {
  const { register, control, handleSubmit } = useForm({
    defaultValues: {
      experiences: [{ company: '', role: '', years: '' }],
    },
  });

  const { fields, append, remove, move } = useFieldArray({
    control,
    name: 'experiences',
  });

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <h2>Work Experience</h2>

      {fields.map((field, index) => (
        <div key={field.id} className="experience-entry">
          <input
            {...register(`experiences.${index}.company`, { required: 'Company required' })}
            placeholder="Company Name"
          />
          <input
            {...register(`experiences.${index}.role`, { required: 'Role required' })}
            placeholder="Your Role"
          />
          <input
            {...register(`experiences.${index}.years`, { required: 'Years required' })}
            type="number"
            placeholder="Years"
          />
          <button type="button" onClick={() => remove(index)}>Remove</button>
          {index > 0 && (
            <button type="button" onClick={() => move(index, index - 1)}>Move Up</button>
          )}
        </div>
      ))}

      <button
        type="button"
        onClick={() => append({ company: '', role: '', years: '' })}
      >
        + Add Experience
      </button>

      <button type="submit">Save</button>
    </form>
  );
}
```

---

## 9. Schema Validation with Zod

Zod is a TypeScript-first schema validation library. It lets you define data shapes declaratively and validate at runtime.

```bash
npm install zod
```

### Defining Schemas

```jsx
import { z } from 'zod';

// Basic schema
const loginSchema = z.object({
  email:    z.string().min(1, 'Email required').email('Invalid email format'),
  password: z.string().min(8, 'Password must be at least 8 characters'),
});

// Complex registration schema
const registrationSchema = z.object({
  firstName: z
    .string()
    .min(1, 'First name is required')
    .min(2, 'Must be at least 2 characters')
    .max(50, 'Must be under 50 characters')
    .regex(/^[a-zA-Z\s'-]+$/, 'Only letters allowed'),

  lastName: z
    .string()
    .min(1, 'Last name is required')
    .min(2, 'Must be at least 2 characters'),

  email: z
    .string()
    .min(1, 'Email is required')
    .email('Enter a valid email address'),

  password: z
    .string()
    .min(8, 'Minimum 8 characters')
    .regex(/[A-Z]/, 'Must contain at least one uppercase letter')
    .regex(/[0-9]/, 'Must contain at least one number')
    .regex(/[!@#$%^&*]/, 'Must contain at least one special character'),

  confirmPassword: z.string().min(1, 'Please confirm your password'),

  age: z
    .number({ invalid_type_error: 'Age must be a number' })
    .min(18, 'Must be at least 18 years old')
    .max(120, 'Invalid age'),

  role: z.enum(['admin', 'editor', 'viewer'], {
    errorMap: () => ({ message: 'Select a valid role' }),
  }),

  website: z
    .string()
    .url('Enter a valid URL')
    .optional()
    .or(z.literal('')),

  agreedToTerms: z
    .boolean()
    .refine(v => v === true, 'You must agree to the terms'),

}).refine(
  // Cross-field validation — passwords must match
  data => data.password === data.confirmPassword,
  {
    message: 'Passwords do not match',
    path: ['confirmPassword'], // which field gets the error
  }
);

// TypeScript type inference
type RegistrationFormValues = z.infer<typeof registrationSchema>;
```

### Using Zod Without RHF

```jsx
function validateWithZod(schema, values) {
  const result = schema.safeParse(values);
  if (result.success) return {};

  return result.error.errors.reduce((acc, err) => {
    const path = err.path.join('.');
    if (!acc[path]) acc[path] = err.message;
    return acc;
  }, {});
}

// Usage
const errors = validateWithZod(registrationSchema, formValues);
```

---

## 10. React Hook Form + Zod Integration

This is the **production standard** combination for form management in React:

```bash
npm install react-hook-form zod @hookform/resolvers
```

```jsx
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';

// 1. Define schema
const profileSchema = z.object({
  displayName: z
    .string()
    .min(2, 'Display name must be at least 2 characters')
    .max(30, 'Display name must be under 30 characters'),

  bio: z
    .string()
    .max(160, 'Bio must be under 160 characters')
    .optional(),

  email: z
    .string()
    .min(1, 'Email is required')
    .email('Invalid email address'),

  website: z
    .string()
    .url('Enter a valid URL (include https://)')
    .optional()
    .or(z.literal('')),

  notifications: z.object({
    email:    z.boolean(),
    push:     z.boolean(),
    sms:      z.boolean(),
  }),
});

type ProfileFormValues = z.infer<typeof profileSchema>;

// 2. Use in component
function EditProfileForm({ user, onSave }) {
  const {
    register,
    handleSubmit,
    formState: { errors, isSubmitting, isDirty, dirtyFields },
    reset,
    watch,
  } = useForm({
    resolver: zodResolver(profileSchema), // Zod validates instead of RHF rules
    defaultValues: {
      displayName: user.displayName ?? '',
      bio:         user.bio ?? '',
      email:       user.email ?? '',
      website:     user.website ?? '',
      notifications: {
        email: user.notifications?.email ?? true,
        push:  user.notifications?.push ?? false,
        sms:   user.notifications?.sms ?? false,
      },
    },
  });

  const bioValue = watch('bio', '');

  async function onSubmit(data) {
    await onSave(data);
    reset(data); // reset to new values (clears isDirty)
  }

  return (
    <form onSubmit={handleSubmit(onSubmit)} noValidate>
      {/* Display Name */}
      <div className="form-group">
        <label htmlFor="displayName">Display Name *</label>
        <input
          id="displayName"
          {...register('displayName')}
          aria-invalid={!!errors.displayName}
          aria-describedby={errors.displayName ? 'displayName-error' : undefined}
        />
        {errors.displayName && (
          <p id="displayName-error" role="alert" className="field-error">
            {errors.displayName.message}
          </p>
        )}
      </div>

      {/* Bio with character counter */}
      <div className="form-group">
        <label htmlFor="bio">Bio</label>
        <textarea
          id="bio"
          {...register('bio')}
          rows={3}
          aria-invalid={!!errors.bio}
        />
        <p className="char-count" aria-live="polite">
          {bioValue?.length ?? 0}/160
        </p>
        {errors.bio && (
          <p role="alert" className="field-error">{errors.bio.message}</p>
        )}
      </div>

      {/* Email */}
      <div className="form-group">
        <label htmlFor="email">Email *</label>
        <input
          id="email"
          type="email"
          {...register('email')}
          aria-invalid={!!errors.email}
        />
        {errors.email && (
          <p role="alert" className="field-error">{errors.email.message}</p>
        )}
      </div>

      {/* Website */}
      <div className="form-group">
        <label htmlFor="website">Website</label>
        <input
          id="website"
          type="url"
          {...register('website')}
          placeholder="https://your-site.com"
        />
        {errors.website && (
          <p role="alert" className="field-error">{errors.website.message}</p>
        )}
      </div>

      {/* Notifications */}
      <fieldset>
        <legend>Notification Preferences</legend>
        {['email', 'push', 'sms'].map(type => (
          <label key={type} className="checkbox-label">
            <input
              type="checkbox"
              {...register(`notifications.${type}`)}
            />
            {type.charAt(0).toUpperCase() + type.slice(1)} notifications
          </label>
        ))}
      </fieldset>

      <div className="form-actions">
        <button
          type="button"
          onClick={() => reset()}
          disabled={!isDirty || isSubmitting}
          className="btn btn--outline"
        >
          Discard Changes
        </button>
        <button
          type="submit"
          disabled={!isDirty || isSubmitting}
          className="btn btn--primary"
        >
          {isSubmitting ? 'Saving...' : 'Save Profile'}
        </button>
      </div>
    </form>
  );
}
```

---

## 11. Complex Form Patterns

### 11.1 Multi-Step Form with Validation per Step

```jsx
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';
import { useState } from 'react';

const step1Schema = z.object({
  firstName: z.string().min(1, 'First name required'),
  lastName:  z.string().min(1, 'Last name required'),
  email:     z.string().email('Invalid email'),
});

const step2Schema = z.object({
  phone:   z.string().regex(/^[6-9]\d{9}$/, 'Invalid phone number'),
  address: z.string().min(5, 'Enter a complete address'),
  city:    z.string().min(1, 'City required'),
  pincode: z.string().regex(/^\d{6}$/, 'Enter a valid 6-digit PIN'),
});

const step3Schema = z.object({
  cardNumber: z.string().regex(/^\d{16}$/, 'Enter 16-digit card number'),
  expiry:     z.string().regex(/^(0[1-9]|1[0-2])\/\d{2}$/, 'Format: MM/YY'),
  cvv:        z.string().regex(/^\d{3,4}$/, 'Invalid CVV'),
});

const STEPS = [
  { title: 'Personal Info', schema: step1Schema },
  { title: 'Address',       schema: step2Schema },
  { title: 'Payment',       schema: step3Schema },
];

function MultiStepCheckout() {
  const [currentStep, setCurrentStep] = useState(0);
  const [completedData, setCompletedData] = useState({});

  const {
    register,
    handleSubmit,
    formState: { errors, isValid },
    trigger,
  } = useForm({
    resolver: zodResolver(STEPS[currentStep].schema),
    mode: 'onChange', // validate on every change
  });

  async function handleNext(data) {
    // Validate current step before proceeding
    const isStepValid = await trigger();
    if (!isStepValid) return;

    setCompletedData(prev => ({ ...prev, ...data }));
    setCurrentStep(prev => prev + 1);
  }

  function handleBack() {
    setCurrentStep(prev => prev - 1);
  }

  async function handleFinalSubmit(data) {
    const allData = { ...completedData, ...data };
    await submitOrder(allData);
  }

  const isLastStep = currentStep === STEPS.length - 1;

  return (
    <div className="multistep-form">
      {/* Progress indicator */}
      <div className="steps-indicator">
        {STEPS.map((step, i) => (
          <div
            key={step.title}
            className={`step-dot ${i < currentStep ? 'completed' : ''} ${i === currentStep ? 'active' : ''}`}
          >
            {i < currentStep ? '✓' : i + 1}
            <span>{step.title}</span>
          </div>
        ))}
      </div>

      <form onSubmit={handleSubmit(isLastStep ? handleFinalSubmit : handleNext)}>
        <h2>{STEPS[currentStep].title}</h2>

        {/* Step content rendered per current step */}
        {currentStep === 0 && (
          <>
            <input {...register('firstName')} placeholder="First Name" />
            {errors.firstName && <p className="error">{errors.firstName.message}</p>}
            <input {...register('lastName')} placeholder="Last Name" />
            {errors.lastName && <p className="error">{errors.lastName.message}</p>}
            <input {...register('email')} type="email" placeholder="Email" />
            {errors.email && <p className="error">{errors.email.message}</p>}
          </>
        )}

        {currentStep === 1 && (
          <>
            <input {...register('phone')} placeholder="Phone" />
            {errors.phone && <p className="error">{errors.phone.message}</p>}
            <input {...register('address')} placeholder="Address" />
            {errors.address && <p className="error">{errors.address.message}</p>}
            <input {...register('city')} placeholder="City" />
            {errors.city && <p className="error">{errors.city.message}</p>}
            <input {...register('pincode')} placeholder="PIN Code" />
            {errors.pincode && <p className="error">{errors.pincode.message}</p>}
          </>
        )}

        {currentStep === 2 && (
          <>
            <input {...register('cardNumber')} placeholder="Card Number (16 digits)" maxLength={16} />
            {errors.cardNumber && <p className="error">{errors.cardNumber.message}</p>}
            <input {...register('expiry')} placeholder="MM/YY" maxLength={5} />
            {errors.expiry && <p className="error">{errors.expiry.message}</p>}
            <input {...register('cvv')} placeholder="CVV" maxLength={4} type="password" />
            {errors.cvv && <p className="error">{errors.cvv.message}</p>}
          </>
        )}

        <div className="form-navigation">
          {currentStep > 0 && (
            <button type="button" onClick={handleBack} className="btn btn--outline">
              ← Back
            </button>
          )}
          <button type="submit" className="btn btn--primary">
            {isLastStep ? 'Place Order' : 'Next →'}
          </button>
        </div>
      </form>
    </div>
  );
}
```

### 11.2 Async Validation (Username Availability Check)

```jsx
import { useForm } from 'react-hook-form';
import { useState } from 'react';

// Debounce helper
function useDebounce(value, delay) {
  const [debounced, setDebounced] = useState(value);
  useEffect(() => {
    const timer = setTimeout(() => setDebounced(value), delay);
    return () => clearTimeout(timer);
  }, [value, delay]);
  return debounced;
}

function UsernameForm() {
  const { register, handleSubmit, watch, formState: { errors } } = useForm({
    mode: 'onChange',
  });

  const username = watch('username', '');
  const debouncedUsername = useDebounce(username, 500);

  const [usernameStatus, setUsernameStatus] = useState('idle'); // idle | checking | available | taken

  useEffect(() => {
    if (!debouncedUsername || debouncedUsername.length < 3) {
      setUsernameStatus('idle');
      return;
    }
    setUsernameStatus('checking');
    checkUsernameAvailability(debouncedUsername).then(available => {
      setUsernameStatus(available ? 'available' : 'taken');
    });
  }, [debouncedUsername]);

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <div className="form-group">
        <label htmlFor="username">Username</label>
        <div className="input-with-status">
          <input
            id="username"
            {...register('username', {
              required: 'Username is required',
              minLength: { value: 3, message: 'Minimum 3 characters' },
              maxLength: { value: 20, message: 'Maximum 20 characters' },
              pattern: {
                value: /^[a-z0-9_]+$/,
                message: 'Only lowercase letters, numbers, and underscores',
              },
            })}
          />
          {/* Status indicator */}
          {usernameStatus === 'checking'   && <span className="status-check">⌛ Checking...</span>}
          {usernameStatus === 'available'  && <span className="status-ok">✅ Available!</span>}
          {usernameStatus === 'taken'      && <span className="status-err">❌ Already taken</span>}
        </div>
        {errors.username && <p className="error">{errors.username.message}</p>}
      </div>
      <button type="submit" disabled={usernameStatus === 'taken' || usernameStatus === 'checking'}>
        Register
      </button>
    </form>
  );
}
```

---

## 12. Accessibility in Forms

Every form should meet WCAG 2.1 AA standards. Here are the key requirements:

### 12.1 Always Associate Labels with Inputs

```jsx
// ✅ Method 1 — htmlFor + id (preferred)
<label htmlFor="email">Email Address</label>
<input id="email" type="email" />

// ✅ Method 2 — Wrap input inside label
<label>
  Email Address
  <input type="email" />
</label>

// ❌ Placeholder only — NOT a substitute for a label
<input type="email" placeholder="Email" />  // Not accessible
```

### 12.2 Use fieldset and legend for Groups

```jsx
<fieldset>
  <legend>Shipping Address</legend>
  <label htmlFor="street">Street</label>
  <input id="street" name="street" />
  <label htmlFor="city">City</label>
  <input id="city" name="city" />
</fieldset>
```

### 12.3 ARIA Attributes for Validation

```jsx
<input
  id="email"
  type="email"
  aria-required="true"
  aria-invalid={!!errors.email}                      // signals invalid state to screen readers
  aria-describedby={errors.email ? 'email-error' : 'email-hint'}  // points to description
/>
{/* Hint (always visible) */}
<p id="email-hint" className="hint">We'll never share your email.</p>
{/* Error (shown conditionally) */}
{errors.email && (
  <p id="email-error" role="alert" className="error">
    {errors.email}
  </p>
)}
```

### 12.4 Loading State Accessibility

```jsx
<button
  type="submit"
  disabled={isSubmitting}
  aria-busy={isSubmitting}          // signals loading to screen readers
  aria-label={isSubmitting ? 'Submitting form, please wait' : 'Submit form'}
>
  {isSubmitting ? (
    <>
      <span aria-hidden="true">⌛</span>
      <span className="sr-only">Submitting...</span>
    </>
  ) : 'Submit'}
</button>
```

### 12.5 Focus Management on Error

```jsx
// Move focus to first error field on submit
function handleSubmitWithFocus(e) {
  e.preventDefault();
  const errors = validateForm(form);
  setErrors(errors);

  const firstErrorField = Object.keys(errors)[0];
  if (firstErrorField) {
    document.getElementById(firstErrorField)?.focus();
  }
}
```

---

## 13. Code Examples (Beginner → Advanced)

### Example 1 — Beginner: Newsletter Signup

```jsx
// File: components/NewsletterForm/NewsletterForm.jsx
import { useState } from 'react';

function NewsletterForm() {
  const [email, setEmail]         = useState('');
  const [error, setError]         = useState('');
  const [isSubmitted, setIsSubmitted] = useState(false);
  const [isLoading, setIsLoading] = useState(false);

  function validate(value) {
    if (!value.trim()) return 'Email address is required';
    if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value)) return 'Please enter a valid email';
    return '';
  }

  async function handleSubmit(e) {
    e.preventDefault();
    const err = validate(email);
    if (err) { setError(err); return; }

    setIsLoading(true);
    try {
      await subscribeToNewsletter(email);
      setIsSubmitted(true);
    } catch {
      setError('Something went wrong. Please try again.');
    } finally {
      setIsLoading(false);
    }
  }

  if (isSubmitted) {
    return (
      <div className="newsletter-success" role="status">
        <p>🎉 You're subscribed! Check your inbox for a confirmation.</p>
      </div>
    );
  }

  return (
    <form onSubmit={handleSubmit} className="newsletter-form" noValidate>
      <h3>Stay Updated</h3>
      <p>Get the latest articles delivered to your inbox.</p>

      <div className="newsletter-form__field">
        <label htmlFor="newsletter-email" className="sr-only">
          Email address
        </label>
        <input
          id="newsletter-email"
          type="email"
          value={email}
          onChange={e => {
            setEmail(e.target.value);
            if (error) setError(validate(e.target.value));
          }}
          onBlur={() => setError(validate(email))}
          placeholder="Enter your email"
          aria-invalid={!!error}
          aria-describedby={error ? 'newsletter-error' : undefined}
          className={error ? 'input--error' : ''}
        />
        <button type="submit" disabled={isLoading}>
          {isLoading ? '...' : 'Subscribe'}
        </button>
      </div>

      {error && (
        <p id="newsletter-error" role="alert" className="newsletter-form__error">
          {error}
        </p>
      )}
    </form>
  );
}

export default NewsletterForm;
```

---

### Example 2 — Intermediate: Complete Registration Form (Custom Hook)

```jsx
// File: components/RegisterForm/RegisterForm.jsx
import useForm from '../../hooks/useForm';
import PasswordStrengthBar from '../PasswordStrengthBar/PasswordStrengthBar';

const INITIAL_VALUES = {
  firstName: '', lastName: '',   email: '',
  password: '',  confirmPassword: '', phone: '',
  dateOfBirth: '', gender: '',   country: '',
  agreedToTerms: false,
};

const VALIDATION = {
  firstName:       v => !v.trim() ? 'Required' : v.length < 2 ? 'Min 2 characters' : null,
  lastName:        v => !v.trim() ? 'Required' : null,
  email:           v => !v.trim() ? 'Required' : !/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(v) ? 'Invalid email' : null,
  password:        v => !v ? 'Required' : v.length < 8 ? 'Min 8 chars' :
                          !/[A-Z]/.test(v) ? 'Need uppercase' : !/[0-9]/.test(v) ? 'Need number' : null,
  confirmPassword: (v, all) => !v ? 'Required' : v !== all.password ? "Passwords don't match" : null,
  phone:           v => !v ? 'Required' : !/^[6-9]\d{9}$/.test(v) ? 'Invalid phone' : null,
  dateOfBirth:     v => {
    if (!v) return 'Required';
    const age = (new Date() - new Date(v)) / (365.25 * 24 * 3600 * 1000);
    return age < 18 ? 'Must be 18+' : null;
  },
  gender:          v => !v ? 'Please select your gender' : null,
  country:         v => !v ? 'Please select your country' : null,
  agreedToTerms:   v => !v ? 'You must agree to the terms' : null,
};

function RegisterForm({ onSuccess }) {
  const { values, errors, touched, isSubmitting, getFieldProps, handleSubmit, reset } =
    useForm(INITIAL_VALUES, VALIDATION);

  async function onSubmit(data) {
    await registerUser(data);
    onSuccess?.();
    reset();
  }

  function Field({ name, label, type = 'text', ...rest }) {
    const props = getFieldProps(name);
    const showError = touched[name] && errors[name];
    return (
      <div className={`form-group ${showError ? 'form-group--error' : ''}`}>
        <label htmlFor={name}>{label}</label>
        <input id={name} type={type} className="form-input" {...props} {...rest} />
        {showError && <p id={`${name}-error`} role="alert" className="field-error">{errors[name]}</p>}
      </div>
    );
  }

  return (
    <form onSubmit={handleSubmit(onSubmit)} noValidate className="register-form">
      <h2>Create Your Account</h2>

      <div className="form-row">
        <Field name="firstName" label="First Name *" />
        <Field name="lastName"  label="Last Name *"  />
      </div>

      <Field name="email" label="Email Address *" type="email" />
      <Field name="phone" label="Phone Number *"  type="tel" placeholder="+91XXXXXXXXXX" />
      <Field name="dateOfBirth" label="Date of Birth *" type="date" />

      {/* Password with strength */}
      <div className={`form-group ${touched.password && errors.password ? 'form-group--error' : ''}`}>
        <label htmlFor="password">Password *</label>
        <input id="password" type="password" className="form-input" {...getFieldProps('password')} />
        <PasswordStrengthBar password={values.password} />
        {touched.password && errors.password && (
          <p id="password-error" role="alert" className="field-error">{errors.password}</p>
        )}
      </div>

      <Field name="confirmPassword" label="Confirm Password *" type="password" />

      {/* Gender */}
      <div className="form-group">
        <fieldset>
          <legend>Gender *</legend>
          {['male', 'female', 'other'].map(g => (
            <label key={g} className="radio-label">
              <input type="radio" {...getFieldProps('gender')} value={g} checked={values.gender === g} />
              {g.charAt(0).toUpperCase() + g.slice(1)}
            </label>
          ))}
        </fieldset>
        {touched.gender && errors.gender && (
          <p role="alert" className="field-error">{errors.gender}</p>
        )}
      </div>

      {/* Country */}
      <div className={`form-group ${touched.country && errors.country ? 'form-group--error' : ''}`}>
        <label htmlFor="country">Country *</label>
        <select id="country" className="form-input" {...getFieldProps('country')}>
          <option value="">-- Select Country --</option>
          <option value="IN">India</option>
          <option value="US">United States</option>
          <option value="UK">United Kingdom</option>
        </select>
        {touched.country && errors.country && (
          <p role="alert" className="field-error">{errors.country}</p>
        )}
      </div>

      {/* Terms */}
      <div className={`form-group ${touched.agreedToTerms && errors.agreedToTerms ? 'form-group--error' : ''}`}>
        <label className="checkbox-label">
          <input type="checkbox" {...getFieldProps('agreedToTerms')} checked={values.agreedToTerms} />
          I agree to the <a href="/terms" target="_blank">Terms of Service</a> and{' '}
          <a href="/privacy" target="_blank">Privacy Policy</a> *
        </label>
        {touched.agreedToTerms && errors.agreedToTerms && (
          <p role="alert" className="field-error">{errors.agreedToTerms}</p>
        )}
      </div>

      <button type="submit" disabled={isSubmitting} className="btn btn--primary btn--full">
        {isSubmitting ? 'Creating Account...' : 'Create Account'}
      </button>
    </form>
  );
}

export default RegisterForm;
```

---

## 14. Real-World Use Cases

### 14.1 Server-Side Validation Errors

```jsx
function LoginForm() {
  const { register, handleSubmit, setError, formState: { errors } } = useForm();

  async function onSubmit(data) {
    try {
      await loginUser(data);
    } catch (apiError) {
      // Map server errors back to form fields
      if (apiError.code === 'INVALID_CREDENTIALS') {
        setError('password', { message: 'Invalid email or password' });
      } else if (apiError.code === 'ACCOUNT_LOCKED') {
        setError('root', { message: 'Account locked. Contact support.' });
      } else if (apiError.fieldErrors) {
        // Multiple field errors from server
        apiError.fieldErrors.forEach(({ field, message }) => {
          setError(field, { message });
        });
      }
    }
  }

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      {errors.root && (
        <div role="alert" className="form-error-banner">{errors.root.message}</div>
      )}
      {/* ... fields */}
    </form>
  );
}
```

### 14.2 Auto-Save Draft Form

```jsx
function ArticleEditor({ articleId }) {
  const { register, watch, handleSubmit } = useForm({
    defaultValues: { title: '', content: '', tags: '' },
  });

  const formValues = watch();
  const [saveStatus, setSaveStatus] = useState('saved'); // 'saved' | 'saving' | 'unsaved'

  // Auto-save on changes (debounced)
  useEffect(() => {
    setSaveStatus('unsaved');
    const timer = setTimeout(async () => {
      setSaveStatus('saving');
      try {
        await saveDraft(articleId, formValues);
        setSaveStatus('saved');
      } catch {
        setSaveStatus('unsaved');
      }
    }, 1500);
    return () => clearTimeout(timer);
  }, [JSON.stringify(formValues)]); // eslint-disable-line

  const statusMessages = {
    saved:   '✅ All changes saved',
    saving:  '⌛ Saving...',
    unsaved: '● Unsaved changes',
  };

  return (
    <form onSubmit={handleSubmit(publishArticle)}>
      <div className="editor-toolbar">
        <span className="save-status" aria-live="polite">{statusMessages[saveStatus]}</span>
        <button type="submit">Publish</button>
      </div>
      <input {...register('title')} placeholder="Article Title" />
      <textarea {...register('content')} rows={20} placeholder="Write your article..." />
      <input {...register('tags')} placeholder="Tags (comma-separated)" />
    </form>
  );
}
```

---

## 15. Best Practices

### 15.1 Always Use `noValidate` on forms

Disable the browser's native validation to fully control it yourself:

```jsx
<form noValidate onSubmit={handleSubmit}>
```

### 15.2 Always `e.preventDefault()` on Submit

```jsx
<form onSubmit={(e) => { e.preventDefault(); handleSubmit(); }}>
// Or with RHF — handleSubmit() does this automatically
<form onSubmit={handleSubmit(onSubmit)}>
```

### 15.3 Use RHF + Zod for Any Non-Trivial Form

For forms with more than 3-4 fields, the RHF + Zod combination saves time and gives you full type safety.

### 15.4 Validate at the Right Time

Use the **hybrid strategy**: validate on blur first time, then real-time after first blur. This gives feedback without being annoying.

### 15.5 Handle All Form States

```jsx
// Always handle: idle, submitting, success, error
{isSuccess && <SuccessBanner />}
{serverError && <ErrorBanner message={serverError} />}
<button disabled={isSubmitting}>{isSubmitting ? 'Sending...' : 'Submit'}</button>
```

### 15.6 Clear Errors When User Starts Fixing

```jsx
function handleChange(e) {
  const { name, value } = e.target;
  setForm(prev => ({ ...prev, [name]: value }));
  if (errors[name]) {
    setErrors(prev => ({ ...prev, [name]: '' })); // clear error on re-type
  }
}
```

### 15.7 Use Semantic HTML

- `<form>` not `<div>` for forms
- `<fieldset>` + `<legend>` for related groups
- `<label>` for every input
- `type="submit"` button for form submission
- `type="button"` for non-submit buttons inside forms

---

## 16. Common Mistakes

### Mistake 1: Missing e.preventDefault()

```jsx
// ❌ Page reloads on submit
<form onSubmit={handleSubmit}>...</form>
function handleSubmit() { /* no preventDefault */ }

// ✅
function handleSubmit(e) { e.preventDefault(); ... }
```

### Mistake 2: Using value Without onChange (Unresponsive Input)

```jsx
// ❌ Input is frozen — user can't type
<input value={email} />

// ✅ Always pair value with onChange
<input value={email} onChange={e => setEmail(e.target.value)} />
```

### Mistake 3: Validating Only on Submit

```jsx
// ❌ User fills 10 fields, submits, and sees 5 errors they could have fixed earlier

// ✅ Use hybrid validation — onBlur first, then onChange
```

### Mistake 4: Not Handling Server Errors

```jsx
// ❌ Ignores API errors
async function handleSubmit(data) {
  await submitForm(data); // What if this throws?
}

// ✅ Always handle server-side errors
async function handleSubmit(data) {
  try {
    await submitForm(data);
    setSuccess(true);
  } catch (err) {
    setServerError(err.message);
  }
}
```

### Mistake 5: Using Array Index as Key in Dynamic Form Arrays

```jsx
// ❌ Index keys break when items are reordered
{fields.map((field, index) => (
  <input key={index} {...register(`items.${index}.name`)} />
))}

// ✅ React Hook Form's useFieldArray provides stable field.id
{fields.map((field, index) => (
  <input key={field.id} {...register(`items.${index}.name`)} />
))}
```

### Mistake 6: Forgetting to Disable Submit Button While Submitting

```jsx
// ❌ User can click submit multiple times — duplicate submissions
<button type="submit">Submit</button>

// ✅ Disable during submission
<button type="submit" disabled={isSubmitting}>
  {isSubmitting ? 'Submitting...' : 'Submit'}
</button>
```

### Mistake 7: Storing Passwords in State Longer Than Needed

```jsx
// ✅ Clear sensitive data after use
async function handleSubmit(data) {
  await login(data);
  reset(); // Clears password from form state
}
```

---

## 17. Performance Considerations

### 17.1 React Hook Form Over Controlled Inputs for Large Forms

RHF uses refs (uncontrolled inputs), so typing doesn't trigger re-renders. A 50-field form has zero re-renders per keystroke vs. 50 re-renders in a controlled approach.

### 17.2 Memoize Validation Schema Outside Component

```jsx
// ❌ Schema recreated on every render
function MyForm() {
  const schema = z.object({ ... }); // new object each render
}

// ✅ Define outside component — stable reference
const mySchema = z.object({ ... });

function MyForm() {
  const { register } = useForm({ resolver: zodResolver(mySchema) });
}
```

### 17.3 Debounce Async Validation

Never fire API validation calls on every keystroke. Debounce by 300-500ms:

```jsx
const debouncedUsername = useDebounce(watch('username'), 500);
useEffect(() => {
  if (debouncedUsername) checkUsername(debouncedUsername);
}, [debouncedUsername]);
```

### 17.4 Use watch() Selectively

Watching all form values re-renders on every change. Watch only what you need:

```jsx
// ❌ Re-renders on ANY field change
const allValues = watch();

// ✅ Re-renders only when 'password' changes
const password = watch('password');
```

---

## 18. Interview Questions

### Q1: What is the difference between a controlled and uncontrolled form input in React?

**Answer:** A controlled input has its value driven entirely by React state — `value` is set from a state variable and every change updates state via `onChange`. React is always the source of truth. An uncontrolled input stores its value in the DOM itself — you use a `ref` to read the value only when needed (usually on submit). Controlled inputs allow real-time validation, conditional rendering, and programmatic reset. Uncontrolled inputs have fewer re-renders. React recommends controlled inputs for most cases. File inputs are always uncontrolled — their value cannot be set programmatically.

---

### Q2: Why should you never set `value` on an input without providing `onChange`?

**Answer:** When you set `value` on an input without an `onChange` handler, React "controls" the input's value but there's no way to update the state driving it. The input becomes read-only — the user's keystrokes are rejected because React always re-renders with the same state value. You'll see a React warning about this. Either provide both `value` and `onChange` for a controlled input, or use `defaultValue` without `onChange` for an uncontrolled one.

---

### Q3: What is the generic change handler pattern and why is it useful?

**Answer:** The generic change handler uses the input's `name` attribute as a dynamic key to update a single state object covering all form fields. One `handleChange` function handles all inputs: `const { name, value, type, checked } = e.target; setForm(prev => ({ ...prev, [name]: type === 'checkbox' ? checked : value }))`. This eliminates writing separate `onChange` and state for every field, making it easy to add new fields and reducing boilerplate significantly.

---

### Q4: What is the best validation timing strategy for forms?

**Answer:** The industry-standard hybrid strategy: validate on **blur** the first time (when the user leaves a field), then switch to **real-time validation** (onChange) for that field after it's been touched. This means no errors shown while the user is actively typing for the first time, but immediate feedback once they've visited a field. On submit, validate all fields at once and mark all as touched. This balances non-intrusive UX with useful feedback.

---

### Q5: What is React Hook Form and why is it preferred over manual controlled forms?

**Answer:** React Hook Form is a library that manages form state using uncontrolled inputs (refs) instead of controlled state. Benefits: (1) **Performance** — no re-renders per keystroke; (2) **Less boilerplate** — `register()` replaces value/onChange/onBlur props; (3) **Built-in validation** — validation rules in `register()`; (4) **Integration** — works with Zod, Yup, Joi via resolvers; (5) **Dynamic fields** — `useFieldArray` for arrays. The trade-off is it requires learning its API. For forms with 3+ fields or complex validation, RHF is significantly better than manual state management.

---

### Q6: What is Zod and how does it improve form validation?

**Answer:** Zod is a TypeScript-first schema declaration and validation library. You define the shape and constraints of your data declaratively (`.string().email().min(5)`) and Zod validates any value against that schema at runtime. Benefits: (1) Type inference — you get TypeScript types from your schema automatically; (2) Composable — schemas can be combined, extended, refined; (3) Detailed error messages — errors include path and message; (4) Cross-field validation — `.refine()` can validate relationships between fields. Combined with `@hookform/resolvers/zod`, your schema drives both TypeScript types and runtime validation.

---

### Q7: How do you handle server-side validation errors in a React form?

**Answer:** After a failed API call, use `setError()` from React Hook Form (or manually update error state) to map server errors back to specific fields. Structure your API error response to include field names and messages. Use `setError('fieldName', { message: 'Error from server' })` to show the error under the corresponding field. For general form errors not tied to a specific field, use `setError('root', { message: '...' })` and display it as a form-level error banner. Always reset errors on retry.

---

### Q8: How do you build a multi-step form with per-step validation?

**Answer:** Use a separate Zod schema or validation rules for each step. On "Next", validate only the current step's fields using RHF's `trigger()` (which runs validation for specified fields without submitting). If validation passes, store the step's data and advance `currentStep`. On the final step, merge all collected data and submit. Key points: use `reset()` carefully between steps to not lose previous step data, and track all data in a parent state object that accumulates data from each completed step.

---

### Q9: What is `useFieldArray` in React Hook Form?

**Answer:** `useFieldArray` is a hook for managing dynamic arrays of form fields — like a list of work experiences, education entries, or product line items. It returns `fields` (the current array with stable `id`s for keys), `append` (add item), `remove` (remove by index), `prepend`, `insert`, `move`, and `swap`. Critically, each field has a stable `field.id` that should be used as the React `key` — not the array index — to prevent state bugs on reorder or remove.

---

### Q10: What accessibility requirements must every form meet?

**Answer:** Key requirements: (1) Every input must have a visible `<label>` associated via `htmlFor`/`id` — placeholder text is not a substitute; (2) Group related inputs with `<fieldset>` and `<legend>`; (3) Use `aria-invalid="true"` on inputs with errors; (4) Use `aria-describedby` to link inputs to their error messages; (5) Error messages should have `role="alert"` for immediate screen reader announcement; (6) Submit button must be a proper `<button type="submit">` or `<input type="submit">`; (7) Disable submit button during submission with `aria-busy="true"`; (8) On error, move focus to the first invalid field programmatically.

---

## 19. Practice Tasks

### Task 1 — Beginner: Feedback Form

Build a `FeedbackForm` component.

**Requirements:**
- Fields: Name (text), Email (email), Category (select: Bug/Feature/General), Rating (radio: 1–5), Message (textarea, max 300 chars)
- Use a single state object with generic `handleChange`
- Validate on submit only:
  - Name: required, min 2 chars
  - Email: required, valid format
  - Category: required
  - Rating: required
  - Message: required, min 10 chars, max 300 chars
- Show field-level errors after attempted submit
- Show character counter for message: "127/300"
- Disable submit during loading
- Show a success message on submit
- Show reset button after success

---

### Task 2 — Intermediate: Job Application Form (Custom Hook)

Build a `JobApplicationForm` using your `useForm` custom hook.

**Requirements:**
- Fields: Full Name, Email, Phone, LinkedIn URL, Years of Experience (number), Skills (checkboxes: React, Node.js, Python, SQL, AWS), Cover Letter (textarea), Resume (file input), Salary Expectation (range: ₹3L–₹50L), Start Date (date), Referral (radio: Yes/No), Referral Name (shown only if Referral = Yes)
- Hybrid validation (blur then onChange)
- LinkedIn URL: valid URL format starting with `https://linkedin.com`
- Phone: 10-digit Indian number
- Years of experience: number between 0 and 40
- Skills: at least 2 required
- Salary: display as formatted label above range slider
- Start date: must be in the future
- Referral name: required only when referral = Yes (conditional validation)
- Show a summary page before submit with "Edit" and "Confirm Submit" buttons
- Progress indicator showing % of required fields filled

---

### Task 3 — Advanced: Full Settings Page (React Hook Form + Zod)

Build a `SettingsPage` with multiple form sections using RHF + Zod.

**Requirements:**

**Section 1 — Profile:**
- Display name, username (async uniqueness check debounced), bio (160 chars), website URL, avatar upload with preview
- Username: lowercase letters, numbers, underscores only, async uniqueness check with loading indicator

**Section 2 — Security:**
- Current password, new password (strength indicator), confirm new password
- Cross-field validation: new ≠ current, confirm = new
- "Show/Hide password" toggle on all password fields

**Section 3 — Notifications:**
- Checkboxes for: email notifications, push notifications, SMS alerts, weekly digest, marketing emails
- Select frequency for digest: Daily/Weekly/Monthly (shown only when weekly digest is checked)

**Section 4 — Danger Zone:**
- "Delete Account" — requires typing "DELETE" in an input to enable the delete button
- Red confirm dialog before proceeding

**General requirements:**
- Each section has its own schema and submit button
- Show "Unsaved changes" indicator per section when form is dirty
- Server error handling: display API errors under correct fields
- All sections fully accessible with correct ARIA
- Auto-save draft for Profile section (debounced 2s)
- Toast notification on successful save per section

---

## 20. Summary

### Key Takeaways

| Concept | Key Point |
|---------|-----------|
| Controlled input | `value` + `onChange` — React owns the value |
| Uncontrolled input | `defaultValue` + `ref` — DOM owns the value |
| File input | Always uncontrolled — read via `e.target.files` |
| Generic handler | One `handleChange` using `e.target.name` as key |
| Validation timing | Hybrid: onBlur first, then onChange after touch |
| Custom useForm | Encapsulates values, errors, touched, handlers |
| React Hook Form | Ref-based, zero re-renders, production standard |
| Zod | Schema-based validation with TypeScript types |
| RHF + Zod | Best combination for production forms |
| useFieldArray | Dynamic form arrays with stable IDs |
| Server errors | `setError()` maps API errors to fields |
| Accessibility | label, fieldset, aria-invalid, aria-describedby, role="alert" |
| noValidate | Always add to `<form>` — disables browser validation |

### Form Approach Decision Guide

```
How complex is the form?
  ├── 1–2 fields (search bar, newsletter) → Local useState + manual validate
  │
  ├── 3–8 fields (login, contact, signup) → Custom useForm hook
  │                                          OR React Hook Form (basic)
  │
  └── 8+ fields OR complex validation     → React Hook Form + Zod resolver
      (multi-step, dynamic arrays,               (production standard)
       async validation, server errors)
```

### Validation Strategy Quick Reference

```
On Submit Only   → Simple forms, low interactivity
On Blur          → After user finishes with each field
On Change        → Real-time, best for passwords/username
Hybrid (Best)    → Blur first, then onChange after first touch
Async            → Debounce 300–500ms, show loading indicator
Server-side      → Map errors to fields with setError()
```

---

> **Next Topic:** `10-useRef-and-DOM.md` — Mastering the useRef hook: accessing DOM elements directly, persisting mutable values without triggering re-renders, managing focus, measurements, animations, and integration with third-party libraries.
