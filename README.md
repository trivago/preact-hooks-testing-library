# DEPRECATED: Moved to [@testing-library/preact-hooks](https://github.com/testing-library/preact-hooks-testing-library)

# preact-hooks-testing-library

preact port of the the [@testing-library/react-hooks](https://github.com/testing-library/react-hooks-testing-library) library.

## Why not `@testing-library/react-hooks`?

Currently, due to the use of `react-test-renderer`, the react hooks testing library most likely will never be compatible with preact.

## Why not another library?

At the time of writing, a library did not exist to test preact hooks. 

## When to use this library

1. You're writing a library with one or more custom hooks that are not directly tied to a component
2. You have a complex hook that is difficult to test through component interactions

## When not to use this library

1. Your hook is defined alongside a component and is only used there
2. Your hook is easy to test by just testing the components using it

## Installation

Install with your favorite package manager

```
yarn add -D @trivago/preact-hooks-testing-library
OR
npm install --save-dev @trivago/preact-hooks-testing-library
```

## Example #1: Basic
---

### `useCounter.ts`

```typescript
import { useState, useCallback } from 'react'; // aliased to preact

const useCounter = () => {
    const [count, setCount] = useState(0);

    const increment = useCallback(() => setCount(c => c + 1));

    return {
        count,
        increment
    }
}

export default useCounter;
```

### `useCounter.test.ts`

```typescript
import { renderHook, act } from 'preact-hooks-testing-library';
import useCounter from './useCounter';

test('should increment counter', () => {
  const { result } = renderHook(() => useCounter());

  act(() => {
    result.current.increment();
  });

  expect(result.current.count).toBe(1);
});

```

## Example #2: Wrapped Components

Sometimes, hooks may need access to values or functionality outside of itself that are provided by a context provider or some other HOC.

```typescript jsx
import { createContext } from 'preact'
import { useState, useCallback, useContext } from 'preact/hooks'

const CounterStepContext = createContext(1)
export const CounterStepProvider = ({ step, children }) => (
  <CounterStepContext.Provider value={step}>{children}</CounterStepContext.Provider>
)
export function useCounter(initialValue = 0) {
  const [count, setCount] = useState(initialValue)
  const step = useContext(CounterStepContext)
  const increment = useCallback(() => setCount((x) => x + step), [step])
  const reset = useCallback(() => setCount(initialValue), [initialValue])
  return { count, increment, reset }
}

```

In our test, we simply use CoounterStepProvider as the wrapper when rendering the hook:

```typescript
import { renderHook, act } from 'preact-hooks-testing-library'
import { CounterStepProvider, useCounter } from './counter'

test('should use custom step when incrementing', () => {
  const wrapper = ({ children }) => <CounterStepProvider step={2}>{children}</CounterStepProvider>
  const { result } = renderHook(() => useCounter(), { wrapper })
  act(() => {
    result.current.increment()
  })
  expect(result.current.count).toBe(2)
})
```

### TODO

- [ ] remove `@ts-nocheck` flag from tests
- [ ] fix disabled auto clean up tests
