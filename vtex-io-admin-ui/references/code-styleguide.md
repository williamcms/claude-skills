# Admin UI code styleguide

Source: https://admin-ui.vercel.app/documentation/developing/code-styleguide

Conventions the Admin UI team applies to component code. Follow them in admin apps for consistency.

## Component typing

`React.FunctionComponent` (`React.FC`) is discouraged. Type `children` explicitly with `React.ReactNode`.

Recommended:

```typescript
interface Props {
  title: string
  children?: React.ReactNode
}

function Component(props: Props) {
  // ...
}
```

Avoid:

```typescript
const Component: React.FC<Props> = (props) => {}
```

## Destructure in the body, not in parameters

```typescript
// Good
function Component(props: Props) {
  const { title } = props
}

// Avoid
function Component({ title }: Props) {}
```

## Explicit prop types, no inline types

```typescript
// Good
interface Props {
  title: string
}
function Component(props: Props) {}

// Avoid
function Component({ title }: { title: string }) {}
```

## Extend types in the declaration, not in parameters

```typescript
// Good
interface Props extends SomeTypeA {
  title: string
}

// Also good
type Props = SomeTypeA & SomeTypeB

// Avoid
function Component(props: Props & SomeTypeA) {}
```

## Exports and structure

- Named exports are preferred over default exports.
- Avoid deep directory nesting.

```typescript
// Good
export function Component() {}

// Avoid
export default function Component() {}
```

Standard component file structure:

```
component/
├── tests/
│   ├── component.test.tsx
│   └── component-composite.test.tsx
├── stories/
│   ├── component.stories.tsx
│   └── component-composite.stories.tsx
├── index.tsx
├── component.tsx
├── component-composite.tsx
├── component.context.tsx
├── component.style.ts
└── component.state.ts
```

Tests follow `*.test.(ts|tsx)` (never `index.test.*`) in a `tests/` directory; Storybook stories use the `*.stories.tsx` suffix in a `stories/` directory.

## Styling rules

Always prefer tokens over hardcoded values:

```typescript
// Good
<div className={csx({ bg: '$primary', padding: '$xs' })} />

// Avoid
<div className={csx({ bg: '#fff', padding: '8px' })} />
```

Prefer shorthand/composed properties:

```typescript
// Preferred
<div className={csx({ bg: '$primary', paddingX: '$xs', size: '20rem' })} />

// Can be improved
<div
  className={csx({
    backgroundColor: '$primary',
    paddingLeft: '$xs',
    paddingRight: '$xs',
    height: '20rem',
    width: '20rem',
  })}
/>
```

Responsive design: use responsive aliases, not arrays or custom media queries:

```typescript
// Good
<div
  className={csx({
    padding: '$xs',
    '@tablet': { padding: '$s' },
    '@desktop': { padding: '$m' },
  })}
/>

// Avoid
<div className={csx({ padding: ['$xs', '$s', '$m'] })} />
<div className={csx({ '@media screen and (min-width: 40em)': { padding: '$s' } })} />
```

Do not style child elements from parent selectors:

```typescript
// Good
<div className={csx({ padding: '$m' })}>
  <h1 className={csx({ color: '$primary' })}>Title</h1>
</div>

// Avoid
<div className={csx({ padding: '$m', h1: { color: '$primary' } })}>
  <h1>Title</h1>
</div>
```

## Naming conventions

Critical/error state props follow this convention across the library:

```typescript
{
  error: boolean
  errorText: string
}
```

Error messages appear after user actions and should provide context and a way forward.
