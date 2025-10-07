# React Debugging Steps Documentation

## Overview
This document outlines the debugging process used to identify and resolve bugs in the React demo application. The debugging was performed using static code analysis, build warnings, and React Developer Tools best practices.

## Bugs Identified and Solutions

### 1. HookSideEffects.js - useCallback Missing Dependencies

**Issue Location**: `src/bugs/HookSideEffects.js` Line 186

**Problem Description**:
```javascript
const track = useCallback((analyticsEvent) => {
  setEventBatch((batch) => [...batch, analyticsEvent]);
}); // Missing dependency array
```

**Root Cause**: The `useCallback` hook is missing its dependency array, causing React Hook exhaustive-deps warning.

**Solution**:
```javascript
const track = useCallback((analyticsEvent) => {
  setEventBatch((batch) => [...batch, analyticsEvent]);
}, []); // Add empty dependency array since no external values are used
```

**Debugging Steps**:
1. Run `npm run build` to identify ESLint warnings
2. Locate the specific line mentioned in the warning
3. Identify that `useCallback` is missing dependency array
4. Add appropriate dependency array based on function requirements

---

### 2. PassingProps.js - Unused useEffect Import

**Issue Location**: `src/bugs/PassingProps.js` Line 1

**Problem Description**:
```javascript
import { useEffect, useState } from "react"; // useEffect is never used
```

**Root Cause**: `useEffect` is imported but never used in the component, causing an unused variable warning.

**Solution**:
```javascript
import { useState } from "react"; // Remove unused useEffect import
```

**Debugging Steps**:
1. Run `npm run build` to identify ESLint warnings
2. Search for `useEffect` usage in the file using grep
3. Confirm that `useEffect` is not used anywhere in the component
4. Remove the unused import

---

### 3. UnexpectedState.js - Incorrect useState Usage

**Issue Location**: `src/bugs/UnexpectedState.js` Lines 58-59

**Problem Description**:
```javascript
function BugAttributes({ initialLevel, onLevelChange }) {
  let currentLevel = useState(initialLevel); // ❌ WRONG: useState returns array, not value
  
  const onLevelUp = () => {
    currentLevel += 1; // ❌ WRONG: Trying to modify useState return value directly
    onLevelChange(currentLevel);
  };
}
```

**Root Cause**: 
- `useState` returns an array `[state, setState]`, not just the state value
- The component is trying to modify the state directly instead of using the setter function
- This breaks React's state management and causes the component to not re-render properly

**Solution**:
```javascript
function BugAttributes({ initialLevel, onLevelChange }) {
  const [currentLevel, setCurrentLevel] = useState(initialLevel); // ✅ CORRECT: Destructure array
  
  const onLevelUp = () => {
    const newLevel = currentLevel + 1; // ✅ CORRECT: Calculate new value
    setCurrentLevel(newLevel); // ✅ CORRECT: Use setter function
    onLevelChange(newLevel);
  };
  
  const onLevelDown = () => {
    const newLevel = currentLevel - 1; // ✅ CORRECT: Calculate new value
    setCurrentLevel(newLevel); // ✅ CORRECT: Use setter function
    onLevelChange(newLevel);
  };
}
```

**Debugging Steps**:
1. Examine the component's state management logic
2. Identify that `useState` is not properly destructured
3. Notice that state updates are not using the setter function
4. Understand that direct mutation of state variables won't trigger re-renders
5. Fix the destructuring and use proper setter functions

---

### 4. PassingProps.js - Mismatched Prop Names

**Issue Location**: `src/bugs/PassingProps.js` PurchaseSummary component

**Problem Description**:
```javascript
function PurchaseSummary({ purchaseLevel, purchaseLikability, ...props }) {
  // purchaseLikability is undefined, but component tries to use it
  return (
    <Text data-test="summary" color="text-weak">
      You are purchasing a level {purchaseLevel} {bug.name} that you{" "}
      {purchaseLikability ?? "haven't decided if you like or not"}
    </Text>
  );
}
```

**Root Cause**: The component receives a `liked` prop but tries to use `purchaseLikability` which is undefined.

**Solution**:
```javascript
function PurchaseSummary({ purchaseLevel, liked, ...props }) { // ✅ CORRECT: Use 'liked' prop
  return (
    <Text data-test="summary" color="text-weak">
      You are purchasing a level {purchaseLevel} {bug.name} that you{" "}
      {liked === "like" ? "like" : liked === "dislike" ? "dislike" : "haven't decided if you like or not"}
    </Text>
  );
}
```

**Debugging Steps**:
1. Examine the component props being passed down
2. Identify the mismatch between received props and used props
3. Trace the data flow from parent to child component
4. Fix the prop name to match what's actually being passed

---

## Debugging Process Used

### 1. Static Code Analysis
- **Build Process**: Used `npm run build` to identify ESLint warnings and errors
- **Code Review**: Manually examined key component files for common React anti-patterns
- **Import Analysis**: Checked for unused imports and missing dependencies

### 2. React Developer Tools Best Practices
- **Component Structure**: Analyzed component hierarchy and prop flow
- **State Management**: Identified improper useState usage patterns
- **Hook Usage**: Verified proper hook implementation and dependency arrays

### 3. Testing Framework Analysis
- **Test Files**: Examined the custom testing framework to understand expected behavior
- **Bug Components**: Analyzed the specific bug components to understand their intended functionality

### 4. Common React Issues Identified
- **Hook Rules Violations**: useState not at top level, missing dependencies
- **State Mutation**: Direct modification of state variables instead of using setters
- **Prop Mismatches**: Inconsistent prop naming between parent and child components
- **Unused Imports**: Dead code that should be cleaned up

## Prevention Strategies

### 1. Use ESLint Rules
- Enable `react-hooks/exhaustive-deps` rule
- Enable `no-unused-vars` rule
- Run linting before commits

### 2. Code Review Checklist
- Verify all hooks follow React rules
- Check that state updates use proper setter functions
- Ensure prop names are consistent throughout the component tree
- Remove unused imports and variables

### 3. Testing
- Write unit tests for component behavior
- Use React Testing Library for integration tests
- Test state changes and prop updates

## Conclusion

The debugging process successfully identified 4 critical bugs that would cause runtime errors and unexpected behavior. The main issues were related to:

1. **Hook implementation errors** (useState, useCallback)
2. **State management anti-patterns**
3. **Prop naming inconsistencies**
4. **Code quality issues** (unused imports)

All issues have been documented with specific solutions and can be resolved by following the provided code examples and debugging steps.
