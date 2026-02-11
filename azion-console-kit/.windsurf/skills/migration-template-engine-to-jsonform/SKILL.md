---
name: migration-template-engine-to-jsonform
description: How to migrate Template Engine schemas to JSONForms format
---

# Migrating Template Engine to JSONForms

This skill describes how to migrate existing Template Engine schemas to the new JSONForms format in Azion Console Kit.

## Overview

The Template Engine uses a custom schema format with `groups` and `fields`. JSONForms uses a standard JSON Schema format with `properties` and `required`. This migration enables:

- Standard JSON Schema validation
- Reusable custom renderers
- Better maintainability
- Consistent field components

## Architecture

```
┌─────────────────────────────────────────────────────────────────────────┐
│                     Template Engine (Legacy)                             │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────────────────────┐  │
│  │   groups    │───▶│   fields    │───▶│  Custom Field Rendering     │  │
│  │  (array)    │    │  (array)    │    │  (per-template logic)       │  │
│  └─────────────┘    └─────────────┘    └─────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────────┘
                              │
                              │ MIGRATION
                              ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                        JSONForms (New)                                   │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────────────────────┐  │
│  │ properties  │───▶│   Tester    │───▶│  Custom Renderer            │  │
│  │  (object)   │    │  (rankWith) │    │  (reusable components)      │  │
│  └─────────────┘    └─────────────┘    └─────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────────┘
```

## Schema Structure Comparison

### Template Engine (Legacy)

```json
{
  "input_schema": {
    "groups": [
      {
        "name": "group_name",
        "label": "Group Label",
        "fields": [
          {
            "name": "field_name",
            "type": "textfield",
            "label": "Field Label",
            "info": "Tooltip info",
            "value": "",
            "hidden": false,
            "description": "Field description",
            "placeholder": "Placeholder text",
            "attrs": { "required": true, "maxLength": 100 },
            "validators": [{ "regex": "pattern", "errorMessage": "Error" }],
            "instantiation_data_path": "envs.[0].value"
          }
        ]
      }
    ]
  }
}
```

### JSONForms (New)

```json
{
  "input_schema": {
    "type": "object",
    "title": "Form Title",
    "properties": {
      "field_name": {
        "type": "string",
        "label": "Field Label",
        "description": "Field description",
        "placeholder": "Placeholder text",
        "maxLength": 100,
        "pattern": "regex_pattern",
        "error": "Custom error message",
        "instantiation_data_path": "envs.[0].value"
      }
    },
    "required": ["field_name"]
  }
}
```

## Field Type Mapping

| Template Engine Type | JSONForms Type | Format/Options | Notes |
|---------------------|----------------|----------------|-------|
| `textfield` | `string` | - | Basic text input |
| `textfield` (password) | `string` | `format: "password"` | Password input |
| `textfield` (multiline) | `string` | `options: { multi: true }` | Textarea |
| `number` | `number` | - | Decimal numbers |
| `number` (integer) | `integer` | - | Whole numbers only |
| `select` / `dropdown` | `string` | `enum: [...]` | Static dropdown |
| `checkbox` | `boolean` | - | Single checkbox |
| `radio` | `string` | `enum: [...]` + custom renderer | Radio group |
| `switch` | `boolean` | - | Toggle switch |

## Property Mapping

| Template Engine | JSONForms | Notes |
|-----------------|-----------|-------|
| `name` | Property key | Becomes the key in `properties` object |
| `type` | `type` | See Field Type Mapping table |
| `label` | `label` | Custom property for renderers |
| `description` | `description` | Standard JSON Schema |
| `placeholder` | `placeholder` | Custom property for renderers |
| `info` | - | Removed (use `description`) |
| `value` | - | Removed (use form defaults) |
| `hidden` | - | Removed (handle in UI logic) |
| `attrs.required` | `required` array | Move to root `required` array |
| `attrs.maxLength` | `maxLength` | Standard JSON Schema |
| `validators[].regex` | `pattern` | Standard JSON Schema |
| `validators[].errorMessage` | `error` | Custom property for renderers |
| `instantiation_data_path` | `instantiation_data_path` | Preserved as-is |

## Step-by-Step Migration Guide

### Step 1: Create the Base Structure

Start with the JSONForms wrapper:

```json
{
  "type": "object",
  "title": "Your Form Title",
  "properties": {},
  "required": []
}
```

### Step 2: Flatten Groups into Properties

Template Engine groups are flattened. The group's `label` becomes the form `title`:

```javascript
// Before: groups[0].label = "Edge Function Starter Kit"
// After: title = "Edge Function Starter Kit"

// Before: groups[0].fields[0].name = "az_name"
// After: properties.az_name = { ... }
```

### Step 3: Convert Each Field

For each field in `groups[*].fields`:

```javascript
// Template Engine field
{
  "name": "az_name",
  "type": "textfield",
  "label": "Application Name",
  "description": "Name of your application",
  "placeholder": "Enter name",
  "attrs": { "required": true, "maxLength": 100 },
  "validators": [{ "regex": "^[A-Za-z]+$", "errorMessage": "Letters only" }],
  "instantiation_data_path": "envs.[0].value"
}

// JSONForms property
"az_name": {
  "type": "string",
  "label": "Application Name",
  "description": "Name of your application",
  "placeholder": "Enter name",
  "maxLength": 100,
  "pattern": "^[A-Za-z]+$",
  "error": "Letters only",
  "instantiation_data_path": "envs.[0].value"
}
```

### Step 4: Build the Required Array

Collect all required fields:

```javascript
// Before: attrs.required = true on each field
// After: required: ["az_name", "other_required_field"]
```

### Step 5: Handle Special Field Types

#### Password Fields

```json
{
  "password_field": {
    "type": "string",
    "format": "password",
    "label": "Password"
  }
}
```

#### Textarea Fields

```json
{
  "description_field": {
    "type": "string",
    "options": { "multi": true },
    "label": "Description"
  }
}
```

#### Number Fields

```json
{
  "quantity": {
    "type": "number",
    "label": "Quantity",
    "minimum": 0,
    "maximum": 100,
    "showButtons": true,
    "useGrouping": false
  }
}
```

#### Integer Fields

```json
{
  "count": {
    "type": "integer",
    "label": "Count",
    "minimum": 1,
    "maximum": 1000
  }
}
```

#### Dropdown/Select Fields

```json
{
  "region": {
    "type": "string",
    "label": "Region",
    "placeholder": "Select a region",
    "enum": ["us-east", "us-west", "eu-west"],
    "enumLabels": ["US East", "US West", "EU West"],
    "error": "Please select a valid region"
  }
}
```

## Migration Examples

### Example 1: Simple Text Field

**Before (Template Engine):**

```json
{
  "groups": [{
    "name": "edge",
    "label": "Edge Function Starter Kit",
    "fields": [{
      "name": "az_name",
      "type": "textfield",
      "label": "Application Name",
      "description": "Name of your edge application",
      "placeholder": "Enter a name for your new Edge Application.",
      "attrs": { "required": true, "maxLength": 100 },
      "validators": [{
        "regex": "^(?:[A-Za-z]+)(?:[A-Za-z0-9 _]*)$",
        "errorMessage": "The use of accents and/or special characters is not allowed."
      }],
      "instantiation_data_path": "envs.[0].value"
    }]
  }]
}
```

**After (JSONForms):**

```json
{
  "type": "object",
  "title": "Edge Function Starter Kit",
  "properties": {
    "az_name": {
      "type": "string",
      "label": "Application Name",
      "description": "Name of your edge application",
      "placeholder": "Enter a name for your new Edge Application.",
      "maxLength": 100,
      "pattern": "^(?:[A-Za-z]+)(?:[A-Za-z0-9 _]*)$",
      "error": "The use of accents and/or special characters is not allowed.",
      "instantiation_data_path": "envs.[0].value"
    }
  },
  "required": ["az_name"]
}
```

### Example 2: Multiple Fields with Optional

**Before (Template Engine):**

```json
{
  "groups": [{
    "name": "firewall_tor_n_bot",
    "label": "Bot Manager and Tor Block Starter Kit",
    "fields": [
      {
        "name": "az_edge_firewall_name",
        "type": "textfield",
        "label": "Edge Firewall Name",
        "description": "Name of the Edge Firewall to be created",
        "placeholder": "Enter a name for your new Edge Firewall",
        "attrs": { "required": true, "maxLength": 255 },
        "validators": [{
          "regex": "(^[A-Za-z0-9\\-]{6,}$)",
          "errorMessage": "Minimum of 6 characters. Only alphanumeric and hyphens."
        }],
        "instantiation_data_path": "envs.[0].value"
      },
      {
        "name": "az_edge_firewall_domains",
        "type": "textfield",
        "label": "Domain ID(s)",
        "description": "Enter the IDs of the domains you want to protect.",
        "placeholder": "Separate multiple numbers using commas",
        "attrs": { "required": false, "maxLength": 255 },
        "instantiation_data_path": "envs.[1].value"
      }
    ]
  }]
}
```

**After (JSONForms):**

```json
{
  "type": "object",
  "title": "Bot Manager and Tor Block Starter Kit",
  "properties": {
    "az_edge_firewall_name": {
      "type": "string",
      "label": "Edge Firewall Name",
      "description": "Name of the Edge Firewall to be created",
      "placeholder": "Enter a name for your new Edge Firewall",
      "maxLength": 255,
      "pattern": "^[A-Za-z0-9\\-]{6,}$",
      "error": "Minimum of 6 characters. Only alphanumeric and hyphens.",
      "instantiation_data_path": "envs.[0].value"
    },
    "az_edge_firewall_domains": {
      "type": "string",
      "label": "Domain ID(s)",
      "description": "Enter the IDs of the domains you want to protect.",
      "placeholder": "Separate multiple numbers using commas",
      "maxLength": 255,
      "instantiation_data_path": "envs.[1].value"
    }
  },
  "required": ["az_edge_firewall_name"]
}
```

### Example 3: Multiple Groups (VCS Integration)

**Before (Template Engine):**

```json
{
  "groups": [
    {
      "name": "gh_connection",
      "label": "GitHub Connection",
      "fields": [{
        "name": "platform_feature__vcs_integration__uuid",
        "type": "textfield",
        "label": "Git Scope",
        "description": "Select the scope for this template.",
        "attrs": { "required": true, "maxLength": 100 },
        "instantiation_data_path": ""
      }]
    },
    {
      "name": "astro",
      "label": "Astro Boilerplate",
      "fields": [{
        "name": "az_name",
        "type": "textfield",
        "label": "Application Name",
        "description": "Give a unique and easy-to-remember name.",
        "placeholder": "Enter a name for your new Edge Application.",
        "attrs": { "required": true, "maxLength": 100 },
        "validators": [{
          "regex": "(^[A-Za-z0-9\\-]{6,}$)",
          "errorMessage": "Minimum of 6 characters. Only alphanumeric and hyphens."
        }],
        "instantiation_data_path": "envs.[0].value"
      }]
    }
  ]
}
```

**After (JSONForms):**

```json
{
  "type": "object",
  "title": "Astro Boilerplate",
  "properties": {
    "platform_feature__vcs_integration__uuid": {
      "type": "string",
      "label": "Git Scope",
      "description": "Select the scope for this template.",
      "placeholder": "",
      "maxLength": 100,
      "instantiation_data_path": ""
    },
    "az_name": {
      "type": "string",
      "label": "Application Name",
      "description": "Give a unique and easy-to-remember name.",
      "placeholder": "Enter a name for your new Edge Application.",
      "maxLength": 100,
      "pattern": "^[A-Za-z0-9\\-]{6,}$",
      "error": "Minimum of 6 characters. Only alphanumeric and hyphens.",
      "instantiation_data_path": "envs.[0].value"
    }
  },
  "required": ["platform_feature__vcs_integration__uuid", "az_name"]
}
```

## Number Field Options

| Property | Type | Description |
|----------|------|-------------|
| `showButtons` | `boolean` | Show increment/decrement buttons (default: `true`) |
| `useGrouping` | `boolean` | Use thousand separators (default: `true`) |
| `minimum` | `number` | Minimum allowed value |
| `maximum` | `number` | Maximum allowed value |
| `multipleOf` | `number` | Step value for increment/decrement |

## Custom Renderers Required

The following custom renderers need to be created to support all Template Engine field types. See the `create-jsonform-custom-render` skill for implementation details.

| Renderer | Status | Tester Condition |
|----------|--------|------------------|
| Input Text | ✅ Implemented | `isStringControl` |
| Input Number | ✅ Implemented | `isNumberControl \|\| isIntegerControl` |
| Input Password | ✅ Implemented | `isStringControl && format === 'password'` |
| Textarea | ✅ Implemented | `isStringControl && options.multi` |
| Dropdown | ✅ Implemented | `isEnumControl` |
| Dynamic Select | ❌ Pending | Custom schema property |
| Checkbox | ❌ Pending | `isBooleanControl` |
| Radio Group | ❌ Pending | Custom schema property |
| Switch Toggle | ❌ Pending | `isBooleanControl` + custom property |

## Best Practices

### 1. Preserve Field Names
Keep the original `name` values as property keys to maintain compatibility with `instantiation_data_path`.

### 2. Use Standard JSON Schema Properties
Prefer standard properties (`maxLength`, `pattern`, `minimum`, `maximum`) over custom ones.

### 3. Custom Properties for UI
Use custom properties (`label`, `placeholder`, `error`, `showButtons`) for UI-specific behavior.

### 4. Required Array at Root
Always define required fields in the root `required` array, not inline.

### 5. Consistent Error Messages
Always provide an `error` property when using `pattern` for user-friendly validation messages.

### 6. Test Migrations
After migration, test:
- All fields render correctly
- Validation works as expected
- Form submission includes all data
- `instantiation_data_path` mappings are correct

## Troubleshooting

### Field Not Rendering

1. **Check type mapping** - Ensure the JSONForms type matches a registered renderer
2. **Verify property key** - Must be a valid JavaScript identifier
3. **Check required array** - Field name must match property key exactly

### Validation Not Working

1. **Pattern syntax** - JSONForms uses JavaScript regex, remove outer parentheses if present
2. **Error not showing** - Ensure `error` property is defined in schema
3. **Required not enforced** - Check field is in `required` array

### Value Not Submitting

1. **Check `instantiation_data_path`** - Must match expected backend format
2. **Verify property key** - Must match form data structure

### Regex Pattern Issues

Template Engine wraps patterns in parentheses. Remove them for JSONForms:

```javascript
// Template Engine
"regex": "(^[A-Za-z0-9\\-]{6,}$)"

// JSONForms
"pattern": "^[A-Za-z0-9\\-]{6,}$"
```

## References

- [JSONForms Documentation](https://jsonforms.io/docs)
- [JSON Schema Specification](https://json-schema.org/)
- [JSONForms UI Schema Controls](https://jsonforms.io/docs/uischema/controls)
- [JSONForms Renderer Sets](https://jsonforms.io/docs/renderer-sets)
- See also: `create-jsonform-custom-render` skill for creating custom renderers