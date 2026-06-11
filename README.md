# ERP Asset Management Module

This project is a full-stack asset management module built with a Go backend and a Vue.js frontend.

## Overall Project Idea

The application manages company assets such as IT equipment, furniture, vehicles, and other fixed assets.
It allows users to:
- register new assets,
- view a searchable asset list,
- filter by asset status and category,
- inspect detailed asset information,
- and mark assets as disposed.

The backend handles asset storage, retrieval, update, and disposal using a database.
The frontend provides a responsive user interface for listing assets, showing details, and creating new assets.

## Your Completed Part

You have completed the two main frontend views:
- `frontend/src/views/AssetList.vue`
- `frontend/src/views/AssetDetail.vue`

These files are responsible for the asset registry page and the individual asset detail page.

---

# `AssetList.vue`

## Purpose

This view shows the asset registry with:
- summary statistics,
- status and category filters,
- a table of assets,
- and a modal form to create a new asset.

## Structure

### Template

- `page-container` wraps the full page and applies an entrance animation.
- `page-header` contains the title and a button to open the add-asset modal.
- `stats-grid` displays computed statistics about assets.
- `controls-bar` contains two filters:
  - `Status Filter` and
  - `Category Filter`.
- The `<table>` shows asset rows loaded from the store.
- Clicking a row navigates to the asset detail page using `$router.push('/assets/' + a.id)`.
- A modal form appears when `showForm` is true.
- The form fields bind to `newAsset` using `v-model`.
- The submit button calls `submitAsset()` to create a new asset.

### Script

- `ref`, `computed`, and `onMounted` come from Vue.
- `useAssetStore()` connects to the global asset store.
- `filterStatus` and `filterCat` are reactive variables for filters.
- `load()` fetches assets from the store with the selected filters.
- `onMounted(load)` loads the asset list when the component mounts.
- `showForm` controls modal visibility.
- `stats` is a computed array with four values:
  - total assets,
  - active deployment,
  - under maintenance,
  - total valuation.
- `badgeClass(status)` returns a CSS class for asset status badges.
- `newAsset` stores the new asset form values.
- `submitError` stores any error text during form submission.
- `submitAsset()` validates required fields, converts the input into the correct payload, calls `store.createAsset(payload)`, closes the modal, resets the form, and reloads assets.

### Style

- Defines grid layout for stats and forms.
- Styles cards, pills, badges, modal overlay, and spinner.
- Uses `scoped` styles so they apply only to this component.

---

# `AssetDetail.vue`

## Purpose

This view shows details for a single selected asset and lets the user dispose of it.

## Structure

### Template

- A back button uses `$router.back()` to return to the previous page.
- The detail card shows when `asset` is loaded.
- It displays:
  - asset name,
  - asset code,
  - status badge,
  - category,
  - location,
  - serial number,
  - purchase order ID,
  - purchase value,
  - current value.
- The `Modify Asset` button is present visually but does not have behavior defined.
- The `Dispose Asset` button calls `disposeAsset()`.
- When `asset` is not loaded yet, a loading spinner is shown.

### Script

- `useRoute()` reads the current route parameters, such as the asset ID.
- `useRouter()` is used for navigation.
- `api` is the Axios instance used for backend calls.
- `asset` is a reactive variable holding the loaded asset.
- `onMounted()` fetches the asset from `/assets/:id` when the component loads.
- If fetching fails, it shows an alert and redirects to `/assets`.
- `disposeAsset()` uses `confirm()` to ask the user before calling `api.delete(`/assets/${asset.value.id}`)` and then navigates back to the list.
- `badgeClass(status)` returns a CSS class for the status label.

### Style

- Uses a two-column grid for layout.
- Styles labels and values for clarity.
- Includes a spinner animation for the loading state.

---

# How to Present This in a Viva

1. Explain that Vue single-file components contain three parts:
   - `template` for markup,
   - `script setup` for logic,
   - `style scoped` for CSS.
2. Describe `AssetList.vue` as the registry page where users can filter assets, view stats, and add new assets.
3. Describe `AssetDetail.vue` as the detail page for one asset, with a disposal action.
4. Mention that both views use Vue reactivity with `ref`, `computed`, and lifecycle hooks.
5. Note that routing is handled with `$router` and `useRoute()`.
6. Highlight that the app uses `axios` for backend communication and a centralized asset store to fetch and create asset data.

---

# Useful Notes

- The backend is in Go and provides REST API endpoints.
- The frontend is Vue 3 and uses `v-model`, `v-for`, and event handlers like `@click`.
- A modal dialog is implemented with `v-if` and CSS overlay.
- Empty states and loading spinners improve user experience.

If you want, I can also add a short “commands to run” section for starting the app during your demo.