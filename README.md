# Dynamic Form Builder - MyCover Assessment

A  Nuxt 4 app that renders a dynamic form from a JSON Schema. It supports common field types, validations, conditional logic (simple `if/then/else` and `x-required_if`), dependent dropdowns, and file inputs. This README focuses on getting the project running locally and how to use the schema upload feature.

## Quick summary

- Framework: Nuxt 4 (Vue 3)
- UI - Nuxt UI
- Package manager used in the project: pnpm (recommended)
- Default port: http://localhost:3000

## Prerequisites

- Node.js 18 or later
- pnpm (recommended). If you prefer npm or yarn you can use them, but instructions below use `pnpm`.

Windows PowerShell users: use the same commands in PowerShell; examples below are compatible with PowerShell.

## Install

Open a terminal in the project folder (`mycover-dynamic-form`) and run:

```powershell
# Install dependencies
pnpm install
```

If you don't have `pnpm` installed, install it globally:

```powershell
npm i -g pnpm
```

## Development

Start the dev server:

```powershell
pnpm dev
```

Open http://localhost:3000 in your browser. The app will hot-reload when you change files.

## Build & Preview (production)

```powershell
# Build for production
pnpm build

# Preview the production build locally
pnpm preview
```

## Usage

1. open the main page and use the "Upload schema" control to select a JSON file. After uploading, click the "Load uploaded" button to render the uploaded schema.
2. Fill the form and click Submit. The app performs client-side validation and shows the collected payload below the form.

Notes:
- File inputs store the selected File object in the form state. For API submission you will need to serialize or upload the file separately.
- The `if/then/else` evaluation in the current implementation supports simple `const` checks in `if.properties` (see sample schema). If you need full JSON Schema evaluation, consider integrating a validator like AJV and wiring it into the conditional checks.

## Schema tips

Supported custom `x-*` properties:

- `x-label`: label shown above the input
- `x-description`: placeholder/help text
- `x-order`: numeric ordering of fields
- `x-source`: describes the input type and data for arrays/selects
- `x-required_if`: shows/makes a field required when another field matches a condition

Example `x-required_if` (field `nin` required when `has_nin` === true):

```json
"nin": {
  "x-required_if": {
    "field": "has_nin",
    "value": true,
    "operator": "eq"
  }
}
```

Dependent dropdowns are supported by providing `x-source.data.dependsOn` and an `options` map keyed by the master field values.

## Project structure

```
app/
├── components/
│   ├── DynamicForm.vue        # Main form renderer
│   └── fields/                # Individual field components
├── pages/
│   └── index.vue              # Page with schema upload and form
```

## Troubleshooting

- Unknown CSS at-rule warnings: the project includes a custom stylesheet with a few experimental at-rules. These are non-fatal warnings from the linter; you can ignore them or update `app/assets/css/main.css` to match your linter rules.
- If hot reload doesn't pick up changes, restart the dev server.

## Testing & linting

This project doesn't include a test suite by default. For linting and formatting, follow the tools noted in the repository (ESLint, Prettier) if you choose to add them.



## License

MIT
