# TypeScript typings for schema authoring and custom widgets

Source: community pattern, [`gblcintra/schema-ui`](https://github.com/gblcintra/schema-ui) (`react/typings/global.d.ts`).

When a project's content schema uses a custom `ui:widget` function, `WidgetProps`, or `Registry` (see [schema-patterns.md](schema-patterns.md)), its `react/` builder needs global type declarations for them — otherwise every custom widget function is typed `any`. Drop the file below **verbatim** at `react/typings/schema.d.ts` in the project being customized.

```ts
import type { ComponentType } from 'react'

declare global {

  /* =========================
   JSON Schema Types
  ========================= */

  type JSONSchemaProperty = {
    type?: 'string' | 'number' | 'boolean' | 'object' | 'array' | 'null' | 'integer'
    title?: string
    description?: string
    properties?: Record<string, JSONSchemaProperty>
    items?: JSONSchemaProperty
    enum?: unknown[]
    default?: unknown
    minimum?: number
    maximum?: number
    multipleOf?: number
    format?: string
    readOnly?: boolean
  }

  type JSONSchemaDependencyCondition = {
    properties: Record<string, JSONSchemaProperty>
  }

  type JSONSchemaDependency = {
    oneOf?: JSONSchemaDependencyCondition[]
  }

  type JSONSchema = {
    properties?: Record<string, JSONSchemaProperty>
    dependencies?: Record<string, JSONSchemaDependency>
  }

  /* =========================
   Component Props
  ========================= */

  interface SchemaUiProps {
    titleItem: string
    activeItem: boolean
    [key: string]: unknown
  }

  /* =========================
   RJSF Core Types
  ========================= */

  type OnChange = (val: unknown) => void

  type SchemaFieldProps = {
    name: string
    schema: unknown
    uiSchema?: unknown
    formData?: unknown
    registry: Registry
    onChange?: (value: unknown) => void
  }

  interface Registry {
    fields: {
      SchemaField: ComponentType<SchemaFieldProps>
    }
  }

  /* =========================
   External API Types
  ========================= */

  type Category = {
    id: number
    name: string
    hasChildren: boolean
    children?: Category[]
  }

  /* =========================
   Widget Props
  ========================= */

  type WidgetProps<T = unknown> = {
    schema: {
      title?: string
      description?: string
      enum?: string[]
      items?: {
        enum?: string[]
      }
    }
    value: T
    onChange: OnChange
    registry: Registry
  }
}
```

These are `declare global` types — no import needed at the call site once the file exists anywhere under the project's TypeScript `include` (i.e. inside `react/`). `Registry` here is deliberately narrower than rjsf's own (only `fields.SchemaField`); widen it locally if a custom widget needs `registry.widgets` too.

## Rule of thumb

The first time a task in a project writes a custom `ui:widget` function or references `WidgetProps`/`Registry`, copy this file into that project's `react/typings/schema.d.ts` as part of the same task — don't wait to be asked, and don't skip it because the project already has some other `typings/global.d.ts` (this file is additive, not a replacement).
