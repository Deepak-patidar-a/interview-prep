# 04 - Multi-Step Form State Management in React

## ❓ Interview Question
> "How would you manage state for a multi-step form in React? For example, a form with 3-4 steps where each step collects different information — walk me through how you'd store the data across steps, handle navigation between steps, and manage validation."

---

## 🙋 My Answer (What I Said)
- Use Redux to store form data across steps temporarily
- Validate all fields in each step before allowing Next
- Disable "Save & Next" until validation passes
- Each step has Prev / Save / Next buttons (contextually enabled)
- On final step, show confirmation popup before calling POST API
- On going back, pre-fill form from Redux store
- If unsaved changes when navigating back, show "do you want to save?" popup

---

## ✅ What I Got Right
- Redux for cross-step state management
- Not calling API until final step
- Validation before proceeding to next step
- Disabling Next until valid
- Per-step button logic (Prev/Save/Next)
- Confirmation popup before final submit
- Pre-filling form from store when going back
- Unsaved changes warning popup — excellent edge case!

## ❌ What Was Wrong / Missing
- Redux can be overkill for simple forms — `useState` or `useReducer` at parent level is often sufficient
- Didn't describe Redux slice shape
- Missing: `currentStep` state to track and render active step
- Missing: progress indicator / stepper UI
- Missing: browser back button handling

---

## 💡 Model Answer

### Simple Approach — useState at Parent Level

```jsx
import { useState } from "react";

const MultiStepForm = () => {
  const [currentStep, setCurrentStep] = useState(1);
  const [formData, setFormData] = useState({
    step1: { name: "", email: "" },
    step2: { address: "", city: "" },
    step3: { cardNumber: "", expiry: "" },
  });

  const updateStep = (step, data) => {
    setFormData((prev) => ({ ...prev, [step]: data }));
  };

  const handleNext = () => setCurrentStep((prev) => prev + 1);
  const handlePrev = () => setCurrentStep((prev) => prev - 1);

  const handleSubmit = async () => {
    const res = await fetch("/api/submit", {
      method: "POST",
      body: JSON.stringify(formData),
    });
    // handle response
  };

  return (
    <div>
      {/* Progress Indicator */}
      <StepIndicator currentStep={currentStep} totalSteps={3} />

      {currentStep === 1 && (
        <Step1
          data={formData.step1}
          onUpdate={(data) => updateStep("step1", data)}
          onNext={handleNext}
        />
      )}
      {currentStep === 2 && (
        <Step2
          data={formData.step2}
          onUpdate={(data) => updateStep("step2", data)}
          onNext={handleNext}
          onPrev={handlePrev}
        />
      )}
      {currentStep === 3 && (
        <Step3
          data={formData.step3}
          onUpdate={(data) => updateStep("step3", data)}
          onPrev={handlePrev}
          onSubmit={handleSubmit}
        />
      )}
    </div>
  );
};
```

### Redux Approach (for large/complex apps)

```js
// formSlice.js
import { createSlice } from "@reduxjs/toolkit";

const formSlice = createSlice({
  name: "multiStepForm",
  initialState: {
    currentStep: 1,
    step1: { name: "", email: "" },
    step2: { address: "", city: "" },
    step3: { cardNumber: "", expiry: "" },
  },
  reducers: {
    updateStep1: (state, action) => { state.step1 = action.payload; },
    updateStep2: (state, action) => { state.step2 = action.payload; },
    updateStep3: (state, action) => { state.step3 = action.payload; },
    setCurrentStep: (state, action) => { state.currentStep = action.payload; },
    resetForm: () => initialState,
  },
});
```

### Step with Validation Example

```jsx
const Step1 = ({ data, onUpdate, onNext }) => {
  const [fields, setFields] = useState(data);
  const [errors, setErrors] = useState({});

  const validate = () => {
    const newErrors = {};
    if (!fields.name.trim()) newErrors.name = "Name is required";
    if (!fields.email.trim()) newErrors.email = "Email is required";
    setErrors(newErrors);
    return Object.keys(newErrors).length === 0;
  };

  const handleNext = () => {
    if (!validate()) return;
    onUpdate(fields);
    onNext();
  };

  return (
    <div>
      <input
        value={fields.name}
        onChange={(e) => setFields({ ...fields, name: e.target.value })}
        placeholder="Name"
      />
      {errors.name && <p style={{ color: "red" }}>{errors.name}</p>}

      <input
        value={fields.email}
        onChange={(e) => setFields({ ...fields, email: e.target.value })}
        placeholder="Email"
      />
      {errors.email && <p style={{ color: "red" }}>{errors.email}</p>}

      <button onClick={handleNext}>Save & Next</button>
    </div>
  );
};
```

### Step Indicator Component

```jsx
const StepIndicator = ({ currentStep, totalSteps }) => (
  <div style={{ display: "flex", gap: "10px", marginBottom: "20px" }}>
    {Array.from({ length: totalSteps }, (_, i) => (
      <div
        key={i}
        style={{
          width: 30,
          height: 30,
          borderRadius: "50%",
          background: i + 1 <= currentStep ? "blue" : "gray",
          color: "white",
          display: "flex",
          alignItems: "center",
          justifyContent: "center",
        }}
      >
        {i + 1}
      </div>
    ))}
  </div>
);
```

---

## 🔑 Key Concepts to Remember
| Concept | Detail |
|---|---|
| Simple state | `useState` at parent for small forms — pass data down as props |
| Redux | Better for complex apps where form data is needed globally |
| currentStep | Track active step with `useState(1)` |
| Validation | Validate per step before calling `onNext()` |
| Pre-fill on back | Pass stored data back into step component as initial state |
| Final submit | Only call API at last step with all accumulated data |
| Progress UI | Show stepper/indicator so user knows where they are |
