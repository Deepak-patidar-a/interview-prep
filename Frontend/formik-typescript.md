# Formik + TypeScript Form Validation

## Question
Why did you choose Formik over React Hook Form? How do you integrate TypeScript with Formik? How do you handle complex validation like a field that is only required when another field has a specific value? What is Formik's known performance issue and how do you solve it?

---

## My Answer
We have a lot of forms needing validation. Formik is built on top of React specifically for validations, reducing the need to write custom hooks. We integrate TypeScript with Formik using type-based validation on fields. For conditional fields we use Formik's disable option where we can write true/false logic based on another field's value. Formik has rendering issues in large forms but we use it for smaller forms so we haven't faced major issues.

## What I Got Right
- Correct — Formik reduces form boilerplate in React
- Knowing Formik has rendering issues and being honest about not facing them yet
- TypeScript integration awareness

## What I Got Wrong
- `disabled` is not the answer for conditional required validation. Disabled means the user cannot interact with the field. Conditionally required means the field only becomes mandatory based on another field's value. The correct answer is Yup's `.when()` method.

---

## Model Answer
We use Formik with Yup for form validation. Forms are typed using TypeScript generics on useFormik so all field values are fully typed and safe. For conditional validation we use Yup's `.when()` method — for example making an address field required only when a checkbox is checked. Formik's known issue is re-rendering the entire form on every keystroke since it uses controlled inputs with centralized state. We avoid this by keeping our forms reasonably sized. For very large forms React Hook Form would be a better choice as it uses uncontrolled inputs and refs, avoiding unnecessary re-renders entirely.

---

## Key Things to Remember

**TypeScript with Formik:**
```typescript
interface LoginFormValues {
  email: string
  password: string
  rememberMe: boolean
}

const formik = useFormik<LoginFormValues>({
  initialValues: {
    email: '',
    password: '',
    rememberMe: false
  },
  onSubmit: (values) => {
    // values is fully typed here
  }
})
```

**Yup conditional validation with `.when()`:**
```javascript
const validationSchema = Yup.object({
  hasAddress: Yup.boolean(),
  address: Yup.string()
    .when('hasAddress', {
      is: true,
      then: Yup.string().required('Address is required'),
      otherwise: Yup.string().notRequired()
    })
})
```

**Formik performance problem:**
- Formik stores all field values in one central state object
- Every keystroke in any field causes the entire form to re-render
- Fine for small forms, laggy for large forms with 50+ fields

**Solutions to Formik re-render issue:**
- Use `React.memo` on individual field components
- Switch to React Hook Form for large forms

**Formik vs React Hook Form:**
- Formik — controlled inputs, centralized state, re-renders on every keystroke
- React Hook Form — uncontrolled inputs using refs, no re-renders on keystroke, better performance
- If starting fresh today — evaluate React Hook Form first

---

## Things to Improve
- Confirm if project uses Yup alongside Formik
- Open React DevTools Profiler on a large Formik form and observe re-renders on keystroke
- Try building one small form with React Hook Form to compare
