# Forms and Validation Guide

## Overview

This guide covers react-hook-form with Zod validation for type-safe form handling.

## React Hook Form Setup

### Basic Form

```tsx
import { useForm } from 'react-hook-form';

interface FormData {
  name: string;
  email: string;
}

function BasicForm() {
  const {
    register,
    handleSubmit,
    formState: { errors, isSubmitting },
  } = useForm<FormData>();

  const onSubmit = async (data: FormData) => {
    console.log(data);
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input {...register('name', { required: 'Name is required' })} />
      {errors.name && <span>{errors.name.message}</span>}

      <input 
        {...register('email', { 
          required: 'Email is required',
          pattern: {
            value: /^[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,}$/i,
            message: 'Invalid email address',
          },
        })} 
      />
      {errors.email && <span>{errors.email.message}</span>}

      <button type="submit" disabled={isSubmitting}>Submit</button>
    </form>
  );
}
```

## Zod Integration

### Define Schema

```tsx
import { z } from 'zod';

const campaignSchema = z.object({
  name: z.string()
    .min(3, 'Name must be at least 3 characters')
    .max(100, 'Name must be less than 100 characters'),
  
  email: z.string().email('Invalid email address'),
  
  budget: z.number({ invalid_type_error: 'Budget must be a number' })
    .min(100, 'Minimum budget is $100')
    .max(1000000, 'Maximum budget is $1,000,000'),
  
  status: z.enum(['draft', 'active', 'paused'], {
    errorMap: () => ({ message: 'Select a valid status' }),
  }),
  
  startDate: z.date({ required_error: 'Start date is required' }),
  
  website: z.string().url('Invalid URL').optional().or(z.literal('')),
  
  tags: z.array(z.string()).min(1, 'Select at least one tag'),
  
  settings: z.object({
    notifications: z.boolean(),
    autoRenew: z.boolean(),
  }),
});

// Infer TypeScript type from schema
type CampaignFormData = z.infer<typeof campaignSchema>;
```

### Form with Zod Resolver

```tsx
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';
import { TextField, Button, Box } from '@mui/material';

const schema = z.object({
  name: z.string().min(3, 'Name must be at least 3 characters'),
  email: z.string().email('Invalid email address'),
  age: z.number().int().min(18, 'Must be at least 18'),
});

type FormData = z.infer<typeof schema>;

function CampaignForm() {
  const {
    register,
    handleSubmit,
    formState: { errors, isSubmitting, isDirty },
    reset,
  } = useForm<FormData>({
    resolver: zodResolver(schema),
    mode: 'onBlur', // Validate on blur
    defaultValues: {
      name: '',
      email: '',
      age: undefined,
    },
  });

  const onSubmit = async (data: FormData) => {
    try {
      await api.createCampaign(data);
      reset();
    } catch (error) {
      console.error(error);
    }
  };

  return (
    <Box component="form" onSubmit={handleSubmit(onSubmit)} sx={{ display: 'flex', flexDirection: 'column', gap: 2 }}>
      <TextField
        {...register('name')}
        label="Campaign Name"
        error={!!errors.name}
        helperText={errors.name?.message}
      />

      <TextField
        {...register('email')}
        label="Email"
        type="email"
        error={!!errors.email}
        helperText={errors.email?.message}
      />

      <TextField
        {...register('age', { valueAsNumber: true })}
        label="Age"
        type="number"
        error={!!errors.age}
        helperText={errors.age?.message}
      />

      <Button type="submit" variant="contained" disabled={isSubmitting || !isDirty}>
        {isSubmitting ? 'Submitting...' : 'Submit'}
      </Button>
    </Box>
  );
}
```

## Common Validation Patterns

### Conditional Validation

```tsx
const schema = z.object({
  paymentType: z.enum(['credit', 'bank']),
  creditCardNumber: z.string().optional(),
  bankAccount: z.string().optional(),
}).refine((data) => {
  if (data.paymentType === 'credit') {
    return !!data.creditCardNumber;
  }
  if (data.paymentType === 'bank') {
    return !!data.bankAccount;
  }
  return true;
}, {
  message: 'Payment details required',
  path: ['creditCardNumber'], // or 'bankAccount'
});
```

### Password Confirmation

```tsx
const passwordSchema = z.object({
  password: z.string()
    .min(8, 'Password must be at least 8 characters')
    .regex(/[A-Z]/, 'Password must contain uppercase letter')
    .regex(/[0-9]/, 'Password must contain a number'),
  confirmPassword: z.string(),
}).refine((data) => data.password === data.confirmPassword, {
  message: 'Passwords do not match',
  path: ['confirmPassword'],
});
```

### Custom Async Validation

```tsx
const usernameSchema = z.object({
  username: z.string()
    .min(3)
    .refine(async (username) => {
      const response = await fetch(`/api/check-username?name=${username}`);
      const { available } = await response.json();
      return available;
    }, 'Username is already taken'),
});
```

## Form with Multiple Steps (Wizard)

```tsx
import { useForm, FormProvider, useFormContext } from 'react-hook-form';

// Step 1 Schema
const step1Schema = z.object({
  name: z.string().min(1),
  email: z.string().email(),
});

// Step 2 Schema
const step2Schema = z.object({
  address: z.string().min(1),
  city: z.string().min(1),
});

// Combined Schema
const fullSchema = step1Schema.merge(step2Schema);

function WizardForm() {
  const [step, setStep] = useState(1);
  
  const methods = useForm({
    resolver: zodResolver(fullSchema),
    mode: 'onBlur',
  });

  const onSubmit = async (data: z.infer<typeof fullSchema>) => {
    await api.submit(data);
  };

  return (
    <FormProvider {...methods}>
      <form onSubmit={methods.handleSubmit(onSubmit)}>
        {step === 1 && <Step1 onNext={() => setStep(2)} />}
        {step === 2 && <Step2 onBack={() => setStep(1)} />}
      </form>
    </FormProvider>
  );
}

function Step1({ onNext }) {
  const { trigger } = useFormContext();
  
  const handleNext = async () => {
    const isValid = await trigger(['name', 'email']);
    if (isValid) onNext();
  };

  return (
    <div>
      <FormField name="name" />
      <FormField name="email" />
      <Button onClick={handleNext}>Next</Button>
    </div>
  );
}
```

## Field Arrays

```tsx
import { useFieldArray, useForm } from 'react-hook-form';

const schema = z.object({
  contacts: z.array(z.object({
    name: z.string().min(1),
    email: z.string().email(),
  })).min(1, 'Add at least one contact'),
});

function ContactsForm() {
  const { control, register, handleSubmit, formState: { errors } } = useForm({
    resolver: zodResolver(schema),
    defaultValues: {
      contacts: [{ name: '', email: '' }],
    },
  });

  const { fields, append, remove } = useFieldArray({
    control,
    name: 'contacts',
  });

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      {fields.map((field, index) => (
        <Box key={field.id} sx={{ display: 'flex', gap: 2, mb: 2 }}>
          <TextField
            {...register(`contacts.${index}.name`)}
            label="Name"
            error={!!errors.contacts?.[index]?.name}
          />
          <TextField
            {...register(`contacts.${index}.email`)}
            label="Email"
            error={!!errors.contacts?.[index]?.email}
          />
          <IconButton onClick={() => remove(index)}>
            <DeleteIcon />
          </IconButton>
        </Box>
      ))}
      
      <Button onClick={() => append({ name: '', email: '' })}>
        Add Contact
      </Button>
      
      {errors.contacts?.root && (
        <Typography color="error">{errors.contacts.root.message}</Typography>
      )}
      
      <Button type="submit">Submit</Button>
    </form>
  );
}
```

## Form with React Query Mutation

```tsx
import { useForm } from 'react-hook-form';
import { useMutation, useQueryClient } from '@tanstack/react-query';
import { zodResolver } from '@hookform/resolvers/zod';

function CampaignForm({ campaignId }: { campaignId?: string }) {
  const queryClient = useQueryClient();
  const { t } = useTranslation('campaigns');

  const { data: campaign } = useQuery({
    queryKey: ['campaign', campaignId],
    queryFn: () => campaignsApi.getById(campaignId!),
    enabled: !!campaignId,
  });

  const mutation = useMutation({
    mutationFn: campaignId 
      ? (data: FormData) => campaignsApi.update(campaignId, data)
      : campaignsApi.create,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['campaigns'] });
      navigate('/campaigns');
    },
    onError: (error) => {
      setError('root', { message: error.message });
    },
  });

  const { register, handleSubmit, formState: { errors }, setError, reset } = useForm({
    resolver: zodResolver(campaignSchema),
    values: campaign, // Populate form when data loads
  });

  const onSubmit = (data: FormData) => {
    mutation.mutate(data);
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      {errors.root && <Alert severity="error">{errors.root.message}</Alert>}
      
      <TextField {...register('name')} label={t('form.name.label')} />
      
      <LoadingButton 
        type="submit" 
        loading={mutation.isPending}
      >
        {campaignId ? t('actions.update') : t('actions.create')}
      </LoadingButton>
    </form>
  );
}
```

## Reusable Form Components

```tsx
// src/components/forms/FormTextField.tsx
import { useFormContext, Controller } from 'react-hook-form';
import { TextField, TextFieldProps } from '@mui/material';

interface FormTextFieldProps extends Omit<TextFieldProps, 'name'> {
  name: string;
}

export function FormTextField({ name, ...props }: FormTextFieldProps) {
  const { control, formState: { errors } } = useFormContext();

  return (
    <Controller
      name={name}
      control={control}
      render={({ field }) => (
        <TextField
          {...field}
          {...props}
          error={!!errors[name]}
          helperText={errors[name]?.message as string}
        />
      )}
    />
  );
}

// Usage
<FormTextField name="email" label="Email" type="email" />
```

## Best Practices

1. **Always use Zod schemas** for consistent validation between client and server
2. **Use `mode: 'onBlur'`** for better UX (don't show errors while typing)
3. **Show loading states** during async validation
4. **Clear form after successful submission** using `reset()`
5. **Handle server errors** by setting them with `setError('root', ...)`
