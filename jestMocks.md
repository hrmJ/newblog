Build a table of test types vs import types

### Basic case 1

- code in same project
- import a named export / single func

(= this is a mock)

```typescript
import * as mainServices from "services/mainServices";

//......

describe("...", () => {
  it("...", () => {
    jest.spyOn(mainServices, "getEventTypeTags").mockResolvedValue(sampleTags);
  });
});
```

### Basic case 2

Mocking a default export and just recording (= mock, not stub)

```typescript
import axios from "axios"
jest.mock("axios");

describe("...", () => {
  it("...", () => {
    expect(axios.get).toHaveBeenCalledWith(...)
  });
});


```

Note: also: mocking a default export in your own project

### Mocking a class

- example: `window.resizeObserver`

```typescript
window.ResizeObserver = jest.fn().mockImplementation(() => ({
  observe: jest.fn(),
  unobserver: jest.fn(),
  disconnect: jest.fn(),
}));
```

### Caveats

Order... When mocking a middleware --> before imports evaluated
