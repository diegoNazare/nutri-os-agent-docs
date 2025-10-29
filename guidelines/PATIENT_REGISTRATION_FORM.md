# Patient Registration Form Implementation Guide

## Overview

This document provides comprehensive instructions for implementing the patient registration form system in the Chrome extension repository. The system allows users to register patients with customizable form fields while maintaining a set of standard required fields.

## Key Features

- **Standard Fields**: Always present, cannot be removed (Nome, Email, Telefone, Data de Nascimento, CPF, Gênero)
- **Custom Fields**: Additional fields can be added via form builder
- **Data Storage**: Standard fields stored in patient table columns, custom fields in `custom_fields` JSONB column
- **Form Persistence**: Form configuration stored in `workspace_forms` table
- **Auto-seeding**: Patient registration form automatically created for all workspaces

## Database Requirements

### 1. Patients Table Structure

The `patients` table must have the following columns:

```sql
CREATE TABLE IF NOT EXISTS public.patients (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  workspace_id UUID REFERENCES public.workspaces(id),
  full_name TEXT NOT NULL,
  nickname TEXT NOT NULL,
  email TEXT,
  phone TEXT NOT NULL,
  birth_date DATE NOT NULL,
  cpf TEXT NOT NULL,
  gender TEXT NOT NULL CHECK (gender IN ('masculino', 'feminino', 'outro')),
  custom_fields JSONB DEFAULT NULL,
  stage_id UUID REFERENCES public.stages(id),
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- Add index for custom_fields queries
CREATE INDEX IF NOT EXISTS idx_patients_custom_fields ON public.patients USING GIN (custom_fields);
```

### 2. Workspace Forms Table

The form configuration is stored in `workspace_forms`:

```sql
CREATE TABLE IF NOT EXISTS public.workspace_forms (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  workspace_id UUID NOT NULL REFERENCES public.workspaces(id) ON DELETE CASCADE,
  name TEXT NOT NULL CHECK (length(trim(name)) > 0),
  description TEXT,
  form_schema JSONB NOT NULL,
  form_template_id UUID REFERENCES public.form_templates(id) ON DELETE SET NULL,
  is_active BOOLEAN NOT NULL DEFAULT true,
  created_by UUID REFERENCES auth.users(id),
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  UNIQUE(workspace_id, name)
);
```

### 3. Migration: Auto-seed Patient Registration Form

Create a migration that seeds the patient registration form for all workspaces:

```sql
-- Migration: seed_patient_registration_form.sql
-- Seed patient registration form for all existing workspaces

DO $$
DECLARE
  workspace_record RECORD;
  default_form_schema JSONB := '{
    "id": "patient-registration",
    "name": "Cadastro de Pacientes",
    "description": "Formulário padrão para cadastro de novos pacientes",
    "sections": [
      {
        "id": "section-personal-info",
        "title": "Informações Pessoais",
        "description": "Campos básicos obrigatórios do paciente (não podem ser removidos)",
        "fields": [
          {
            "id": "field-name",
            "type": "text",
            "label": "Nome Completo",
            "placeholder": "Digite o nome completo do paciente",
            "required": true,
            "isStandard": true
          },
          {
            "id": "field-email",
            "type": "email",
            "label": "E-mail",
            "placeholder": "exemplo@email.com",
            "required": true,
            "isStandard": true
          },
          {
            "id": "field-phone",
            "type": "tel",
            "label": "Telefone",
            "placeholder": "(00) 00000-0000",
            "required": true,
            "isStandard": true
          },
          {
            "id": "field-birthdate",
            "type": "date",
            "label": "Data de Nascimento",
            "placeholder": "Selecione a data",
            "required": true,
            "isStandard": true
          },
          {
            "id": "field-cpf",
            "type": "text",
            "label": "CPF",
            "placeholder": "000.000.000-00",
            "required": true,
            "isStandard": true
          },
          {
            "id": "field-gender",
            "type": "select",
            "label": "Gênero",
            "placeholder": "Selecione o gênero",
            "required": true,
            "isStandard": true,
            "options": [
              { "id": "masculino", "label": "Masculino", "value": "masculino" },
              { "id": "feminino", "label": "Feminino", "value": "feminino" },
              { "id": "outro", "label": "Outro", "value": "outro" }
            ]
          }
        ],
        "order": 1
      }
    ],
    "settings": {
      "allowMultipleSubmissions": false,
      "showProgressBar": true,
      "showSectionNumbers": true,
      "submitButtonText": "Cadastrar Paciente",
      "successMessage": "Paciente cadastrado com sucesso!"
    }
  }'::jsonb;
BEGIN
  -- Insert patient registration form for all existing workspaces that don't have it
  FOR workspace_record IN 
    SELECT id FROM public.workspaces
  LOOP
    IF NOT EXISTS (
      SELECT 1 FROM public.workspace_forms 
      WHERE workspace_id = workspace_record.id 
      AND name = '__patient_registration_form__'
    ) THEN
      INSERT INTO public.workspace_forms (
        workspace_id,
        name,
        description,
        form_schema,
        form_template_id,
        is_active,
        created_by
      ) VALUES (
        workspace_record.id,
        '__patient_registration_form__',
        'Formulário de cadastro de pacientes',
        default_form_schema,
        NULL,
        true,
        NULL
      )
      ON CONFLICT (workspace_id, name) DO NOTHING;
    END IF;
  END LOOP;
END $$;
```

## Form Schema Structure

### TypeScript Types

```typescript
export type FieldType = 
  | 'text'
  | 'email'
  | 'number'
  | 'tel'
  | 'url'
  | 'password'
  | 'textarea'
  | 'select'
  | 'radio'
  | 'checkbox'
  | 'date'
  | 'time'
  | 'datetime'
  | 'file'
  | 'rating'
  | 'scale';

export interface FormFieldOption {
  id: string;
  label: string;
  value: string | number;
}

export interface FormField {
  id: string;
  type: FieldType;
  label: string;
  description?: string;
  placeholder?: string;
  required?: boolean;
  defaultValue?: any;
  options?: FormFieldOption[];
  validation?: FormFieldValidation;
  settings?: Record<string, any>;
  isStandard?: boolean; // Indicates if this is a standard field that can't be deleted
}

export interface FormFieldValidation {
  min?: number;
  max?: number;
  minLength?: number;
  maxLength?: number;
  pattern?: string;
  customMessage?: string;
}

export interface FormSection {
  id: string;
  title: string;
  description?: string;
  fields: FormField[];
  order: number;
}

export interface FormSchema {
  id: string;
  name: string;
  description?: string;
  sections: FormSection[];
  settings: FormSettings;
}

export interface FormSettings {
  allowMultipleSubmissions?: boolean;
  showProgressBar?: boolean;
  showSectionNumbers?: boolean;
  submitButtonText?: string;
  successMessage?: string;
  redirectUrl?: string;
}
```

### Standard Fields Definition

The following fields are **always required** and must be present in every patient registration form:

```typescript
const STANDARD_PATIENT_FIELDS = [
  {
    id: 'field-name',
    type: 'text' as const,
    label: 'Nome Completo',
    placeholder: 'Digite o nome completo do paciente',
    required: true,
    isStandard: true,
  },
  {
    id: 'field-email',
    type: 'email' as const,
    label: 'E-mail',
    placeholder: 'exemplo@email.com',
    required: true,
    isStandard: true,
  },
  {
    id: 'field-phone',
    type: 'tel' as const,
    label: 'Telefone',
    placeholder: '(00) 00000-0000',
    required: true,
    isStandard: true,
  },
  {
    id: 'field-birthdate',
    type: 'date' as const,
    label: 'Data de Nascimento',
    placeholder: 'Selecione a data',
    required: true,
    isStandard: true,
  },
  {
    id: 'field-cpf',
    type: 'text' as const,
    label: 'CPF',
    placeholder: '000.000.000-00',
    required: true,
    isStandard: true,
  },
  {
    id: 'field-gender',
    type: 'select' as const,
    label: 'Gênero',
    placeholder: 'Selecione o gênero',
    required: true,
    isStandard: true,
    options: [
      { id: 'masculino', label: 'Masculino', value: 'masculino' },
      { id: 'feminino', label: 'Feminino', value: 'feminino' },
      { id: 'outro', label: 'Outro', value: 'outro' },
    ],
  },
];
```

### Form Name Constant

The patient registration form is identified by this special name:

```typescript
const PATIENT_REGISTRATION_FORM_NAME = "__patient_registration_form__";
```

## Implementation Steps

### Step 1: Fetch Patient Registration Form

Load the form from Supabase `workspace_forms` table:

```typescript
// Fetch form from workspace_forms table
const { data: form, error } = await supabase
  .from('workspace_forms')
  .select('*')
  .eq('workspace_id', workspaceId)
  .eq('name', '__patient_registration_form__')
  .eq('is_active', true)
  .single();

// If form doesn't exist, use default schema
const formSchema = form?.form_schema || getDefaultFormSchema();
```

### Step 2: Render Form Fields

Render fields based on the form schema. For each field:

- **Text/Email/Tel**: Use standard input with appropriate `type` attribute
- **Date**: Use date picker component
- **Select**: Use select dropdown with `z-[110]` z-index (critical for modals!)
- **Radio/Checkbox**: Use appropriate input type
- **Textarea**: Use textarea component

**Important**: Select dropdowns must have `z-[110]` z-index to appear above modal overlays (`z-[95]`).

### Step 3: Form Submission Handler

Process form data and map to patient structure:

```typescript
const handleSubmit = async (formData: Record<string, any>) => {
  // Helper to get field value with fallback
  const getFieldValue = (...keys: string[]) => {
    for (const key of keys) {
      if (formData[key] && formData[key].trim()) return formData[key].trim();
    }
    return '';
  };

  // Extract standard fields
  const fullName = getFieldValue('field-name', 'full_name');
  const email = getFieldValue('field-email', 'email');
  const phone = getFieldValue('field-phone', 'phone');
  const birthDate = getFieldValue('field-birthdate', 'birth_date');
  const cpf = getFieldValue('field-cpf', 'cpf');
  const gender = getFieldValue('field-gender', 'gender') || 'outro';

  // Format phone to international format (+55XXXXXXXXXXX)
  let formattedPhone = '';
  if (phone) {
    const digitsOnly = phone.replace(/\D/g, '');
    formattedPhone = phone.startsWith('+') ? phone : `+55${digitsOnly}`;
  }

  // Format birth date to YYYY-MM-DD
  let formattedBirthDate = '';
  if (birthDate) {
    if (birthDate.match(/^\d{4}-\d{2}-\d{2}$/)) {
      formattedBirthDate = birthDate;
    } else {
      const date = new Date(birthDate);
      if (!isNaN(date.getTime())) {
        formattedBirthDate = date.toISOString().split('T')[0];
      }
    }
  }

  // Format CPF - remove all non-digits, ensure 11 digits
  let formattedCpf = '';
  if (cpf && cpf.trim()) {
    const cpfDigits = cpf.replace(/\D/g, '');
    formattedCpf = cpfDigits.length === 11 
      ? cpfDigits 
      : cpfDigits.padStart(11, '0');
  }

  // Separate standard fields from custom fields
  const standardFieldIds = [
    'field-name',
    'field-email',
    'field-phone',
    'field-birthdate',
    'field-cpf',
    'field-gender'
  ];

  const customFieldsData: Record<string, any> = {};

  // Process custom fields with metadata
  formSchema.sections.forEach(section => {
    section.fields.forEach(field => {
      if (!standardFieldIds.includes(field.id) && formData[field.id] !== undefined) {
        customFieldsData[field.id] = {
          type: field.type,
          label: field.label,
          value: formData[field.id],
          required: field.required || false,
          description: field.description || null,
          placeholder: field.placeholder || null,
          options: field.options || null,
          validation: field.validation || null,
        };
      }
    });
  });

  // Build patient data object
  const patientData = {
    full_name: fullName,
    nickname: fullName?.split(' ')[0] || '',
    email: email,
    phone: formattedPhone,
    birth_date: formattedBirthDate,
    gender: gender as 'masculino' | 'feminino' | 'outro',
    cpf: formattedCpf,
    workspace_id: workspaceId,
    custom_fields: Object.keys(customFieldsData).length > 0 ? customFieldsData : null,
  };

  // Validate required fields
  const requiredFields = [
    { field: 'full_name', value: patientData.full_name, label: 'Nome completo' },
    { field: 'email', value: patientData.email, label: 'E-mail' },
    { field: 'phone', value: patientData.phone, label: 'Telefone' },
    { field: 'birth_date', value: patientData.birth_date, label: 'Data de nascimento' },
    { field: 'cpf', value: patientData.cpf, label: 'CPF' },
    { field: 'gender', value: patientData.gender, label: 'Gênero' },
  ];

  const missingFields = requiredFields.filter(f => !f.value);
  if (missingFields.length > 0) {
    throw new Error(`Campos obrigatórios: ${missingFields.map(f => f.label).join(', ')}`);
  }

  // Validate formats
  if (patientData.phone && !patientData.phone.match(/^\+[0-9]{10,15}$/)) {
    throw new Error('Telefone inválido');
  }

  if (patientData.cpf && !patientData.cpf.match(/^[0-9]{11}$/)) {
    throw new Error('CPF inválido');
  }

  if (patientData.email && !patientData.email.match(/^[^\s@]+@[^\s@]+\.[^\s@]+$/)) {
    throw new Error('E-mail inválido');
  }

  return patientData;
};
```

### Step 4: Save to Supabase

Insert patient into database:

```typescript
const savePatient = async (patientData: PatientData) => {
  const { data, error } = await supabase
    .from('patients')
    .insert([patientData])
    .select()
    .single();

  if (error) {
    console.error('Error creating patient:', error);
    throw error;
  }

  return data;
};
```

## Data Mapping Reference

### Standard Field IDs → Database Columns

| Form Field ID | Database Column | Format | Required |
|--------------|----------------|--------|----------|
| `field-name` | `full_name` | String (trimmed) | ✅ Yes |
| `field-email` | `email` | Email format | ✅ Yes |
| `field-phone` | `phone` | `+55XXXXXXXXXXX` (international format) | ✅ Yes |
| `field-birthdate` | `birth_date` | `YYYY-MM-DD` | ✅ Yes |
| `field-cpf` | `cpf` | 11 digits (numbers only) | ✅ Yes |
| `field-gender` | `gender` | `'masculino' \| 'feminino' \| 'outro'` | ✅ Yes |

### Custom Fields → `custom_fields` JSONB

All non-standard fields are stored in `custom_fields` with metadata:

```json
{
  "field-id": {
    "type": "field_type",
    "label": "Field Label",
    "value": "user_input_value",
    "required": false,
    "description": "Field description",
    "placeholder": "Field placeholder",
    "options": [...],
    "validation": {...}
  }
}
```

## Validation Rules

### Phone Validation
- Format: International format starting with `+`
- Pattern: `/^\+[0-9]{10,15}$/`
- Default: Add `+55` prefix for Brazilian numbers

### CPF Validation
- Format: Exactly 11 digits (numbers only)
- Pattern: `/^[0-9]{11}$/`
- Processing: Remove all non-digits, pad with zeros if needed

### Email Validation
- Format: Standard email format
- Pattern: `/^[^\s@]+@[^\s@]+\.[^\s@]+$/`

### Date Validation
- Format: `YYYY-MM-DD`
- Processing: Convert various date formats to ISO date string

## UI Components Required

### Form Renderer Components

You'll need components to render each field type:

1. **Text Input** (`text`, `email`, `tel`, `url`, `password`)
2. **Number Input** (`number`)
3. **Textarea** (`textarea`)
4. **Select Dropdown** (`select`) - **Critical**: Must have `z-[110]` z-index!
5. **Radio Group** (`radio`)
6. **Checkbox Group** (`checkbox`)
7. **Date Picker** (`date`, `datetime`)
8. **Time Picker** (`time`)
9. **File Upload** (`file`)
10. **Rating** (`rating`)
11. **Scale/Slider** (`scale`)

### Modal/Dialog Component

For Chrome extension, use a modal overlay with:
- Overlay z-index: `z-[95]`
- Content z-index: `z-[100]`
- Select dropdown z-index: `z-[110]` (must be above modal content)

## Default Form Schema

If the form doesn't exist in the database, use this default:

```typescript
const getDefaultFormSchema = (): FormSchema => ({
  id: 'patient-registration',
  name: 'Cadastro de Pacientes',
  description: 'Formulário padrão para cadastro de novos pacientes',
  sections: [
    {
      id: 'section-personal-info',
      title: 'Informações Pessoais',
      description: 'Campos básicos obrigatórios do paciente (não podem ser removidos)',
      fields: STANDARD_PATIENT_FIELDS, // Use the standard fields array above
      order: 1,
    },
  ],
  settings: {
    allowMultipleSubmissions: false,
    showProgressBar: true,
    showSectionNumbers: true,
    submitButtonText: 'Cadastrar Paciente',
    successMessage: 'Paciente cadastrado com sucesso!',
  },
});
```

## Implementation Checklist

- [ ] Create database migration for `custom_fields` column (if not exists)
- [ ] Create database migration to seed patient registration form
- [ ] Implement form schema TypeScript types
- [ ] Create form fetcher function (load from `workspace_forms`)
- [ ] Implement form renderer component
- [ ] Implement field renderers for all field types
- [ ] Implement form submission handler with data mapping
- [ ] Add validation logic (phone, CPF, email, date)
- [ ] Implement Supabase patient insert function
- [ ] Handle custom fields storage in JSONB
- [ ] Test with standard fields only
- [ ] Test with custom fields added
- [ ] Verify z-index hierarchy (modal overlay < content < select dropdown)
- [ ] Test form loading from database
- [ ] Test fallback to default schema when form doesn't exist

## Common Pitfalls

1. **Z-Index Issues**: Select dropdowns must have `z-[110]` to appear above modals
2. **Phone Format**: Must be international format (`+55XXXXXXXXXXX`)
3. **CPF Format**: Must be exactly 11 digits (strip formatting)
4. **Date Format**: Must be `YYYY-MM-DD` for database
5. **Custom Fields**: Must include metadata (type, label, etc.) not just values
6. **Field ID Mapping**: Standard fields use specific IDs (`field-name`, `field-email`, etc.)
7. **Workspace Context**: Always require `workspace_id` when creating patients

## Testing Examples

### Test Case 1: Standard Fields Only

```typescript
const formData = {
  'field-name': 'João Silva',
  'field-email': 'joao@example.com',
  'field-phone': '(11) 98765-4321',
  'field-birthdate': '1990-01-15',
  'field-cpf': '123.456.789-00',
  'field-gender': 'masculino'
};

// Expected output:
{
  full_name: 'João Silva',
  nickname: 'João',
  email: 'joao@example.com',
  phone: '+5511987654321',
  birth_date: '1990-01-15',
  cpf: '12345678900',
  gender: 'masculino',
  custom_fields: null
}
```

### Test Case 2: With Custom Fields

```typescript
const formData = {
  'field-name': 'Maria Santos',
  'field-email': 'maria@example.com',
  'field-phone': '(21) 99876-5432',
  'field-birthdate': '1995-05-20',
  'field-cpf': '987.654.321-00',
  'field-gender': 'feminino',
  'field-observations': 'Paciente com alergia a lactose'
};

// Expected output:
{
  full_name: 'Maria Santos',
  nickname: 'Maria',
  email: 'maria@example.com',
  phone: '+5521998765432',
  birth_date: '1995-05-20',
  cpf: '98765432100',
  gender: 'feminino',
  custom_fields: {
    'field-observations': {
      type: 'textarea',
      label: 'Observações',
      value: 'Paciente com alergia a lactose',
      required: false,
      // ... other metadata
    }
  }
}
```

## Additional Notes

- The form is automatically created for all workspaces via database migration
- Standard fields cannot be removed (enforced by `isStandard: true` flag)
- Custom fields are stored with full metadata for form builder compatibility
- The system supports both create and edit modes
- Patients are automatically assigned to the first stage of their workspace (via database triggers)

## Related Files in Main Repository

- `src/pages/CadastrosPacientes.tsx` - Form builder page
- `src/components/patient/PatientRegistrationModal.tsx` - Registration modal
- `src/components/forms/FormRenderer.tsx` - Form renderer component
- `src/types/form.ts` - TypeScript type definitions
- `supabase/migrations/20251029233900_seed_patient_registration_form.sql` - Migration file
- `src/hooks/usePatients.ts` - Patient CRUD operations

