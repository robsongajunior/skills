---
name: theme-management
description: How to work with theme management in Azion Console Kit
---

# Theme Management Skill

This document describes the theme management architecture in Azion Console Kit, including how to read, set, and react to theme changes.

## Architecture Overview

Theme management is handled by a dedicated Pinia store (`useThemeStore`) that is completely decoupled from account/user data. This ensures theme persistence across sessions and prevents theme state from being lost during logout/login flows.

## Key Files

| File | Purpose |
|------|---------|
| `src/stores/theme.js` | Pinia store for theme state, exports `useThemeStore`, `getSystemTheme`, `DARK_SCHEME_QUERY` |
| `src/helpers/theme-apply.js` | Helper to apply theme to DOM (adds/removes CSS classes on `:root`) |
| `src/router/hooks/guards/themeGuard.js` | Router guard that initializes theme on app load |
| `src/App.vue` | Watches theme changes and listens for OS theme preference changes |

## Theme Store (`src/stores/theme.js`)

### Exports

```javascript
// Constants
export const DARK_SCHEME_QUERY = '(prefers-color-scheme: dark)'

// Helper function - returns 'dark' or 'light' based on OS preference
export const getSystemTheme = () => {
  return window.matchMedia(DARK_SCHEME_QUERY).matches ? 'dark' : 'light'
}

// Pinia store
export const useThemeStore = defineStore({
  id: 'theme',
  state: () => ({ theme: getInitialTheme() }),
  getters: {
    currentTheme: (state) => state.theme  // Returns 'light', 'dark', or 'system'
  },
  actions: {
    setTheme(theme) {
      this.theme = theme
      localStorage.setItem('theme', theme)
    }
  }
})
```

### Theme Values

- `'light'` - Light theme
- `'dark'` - Dark theme  
- `'system'` - Follow OS preference

### Persistence

Theme is persisted in `localStorage` under the key `'theme'`.

## Reading Theme in Components

### Using `storeToRefs` (Recommended for reactivity)

```javascript
import { useThemeStore } from '@/stores/theme'
import { storeToRefs } from 'pinia'

const themeStore = useThemeStore()
const { currentTheme } = storeToRefs(themeStore)

// Use in template or watch
watch(currentTheme, (theme) => {
  console.log('Theme changed to:', theme)
})
```

### Direct access (for non-reactive reads)

```javascript
import { useThemeStore } from '@/stores/theme'

const themeStore = useThemeStore()
const theme = themeStore.currentTheme
```

### For Monaco Editor theme

```javascript
import { useThemeStore } from '@/stores/theme'
import { computed } from 'vue'

const store = useThemeStore()
const theme = computed(() => {
  return store.currentTheme === 'light' ? 'vs' : 'vs-dark'
})
```

## Setting Theme

```javascript
import { useThemeStore } from '@/stores/theme'

const themeStore = useThemeStore()
themeStore.setTheme('dark')  // or 'light' or 'system'
```

## Applying Theme to DOM

The `themeApply` helper applies the theme by toggling CSS classes on the `:root` element:

```javascript
import { themeApply } from '@/helpers'

themeApply('dark')   // Applies azion-dark class
themeApply('light')  // Applies azion-light class
themeApply('system') // Resolves to dark/light based on OS preference
```

### CSS Classes

- `azion-light` - Light theme styles
- `azion-dark` - Dark theme styles

## System Theme Detection

To detect or react to OS theme preference:

```javascript
import { DARK_SCHEME_QUERY, getSystemTheme } from '@/stores/theme'

// Get current system theme
const systemTheme = getSystemTheme() // Returns 'dark' or 'light'

// Listen for OS theme changes
window.matchMedia(DARK_SCHEME_QUERY).addEventListener('change', (event) => {
  const newTheme = event.matches ? 'dark' : 'light'
  console.log('OS theme changed to:', newTheme)
})
```

## Common Patterns

### Adding theme support to a new component

1. Import the theme store:
   ```javascript
   import { useThemeStore } from '@/stores/theme'
   ```

2. Get the current theme:
   ```javascript
   const store = useThemeStore()
   const theme = computed(() => store.currentTheme)
   ```

3. Use in template or logic:
   ```html
   <div :class="{ 'dark-mode': theme === 'dark' }">...</div>
   ```

### Theme-aware styling

```javascript
const badgeClass = computed(() => {
  return currentTheme.value === 'light'
    ? 'bg-gray-200 text-gray-800'
    : 'bg-gray-700 text-gray-200'
})
```

## Testing

Test file: `src/tests/helpers/themeApply.test.js`

When testing theme-related code, mock `window.matchMedia`:

```javascript
vi.spyOn(window, 'matchMedia').mockReturnValue({
  matches: true // true = dark mode, false = light mode
})
```

## Important Notes

1. **Never store theme in account data** - Theme is UI preference, not user data
2. **Always use `useThemeStore`** - Don't access localStorage directly for theme
3. **Use `storeToRefs` for reactivity** - Ensures components update when theme changes
4. **Handle 'system' theme** - Always resolve to actual theme when applying to DOM
5. **Test with both themes** - Ensure components look correct in both light and dark modes

## Migration Guide (from account store)

If you find code using the old pattern:

```javascript
// OLD - Don't use
import { useAccountStore } from '@/stores/account'
const accountStore = useAccountStore()
const theme = accountStore.currentTheme
accountStore.setTheme('dark')
```

Replace with:

```javascript
// NEW - Use this
import { useThemeStore } from '@/stores/theme'
const themeStore = useThemeStore()
const theme = themeStore.currentTheme
themeStore.setTheme('dark')
```
