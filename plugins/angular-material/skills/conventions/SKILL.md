---
name: conventions
description: The user's conventions for Angular Material (@angular/material) apps, layered on angular:conventions. Use before building, styling, theming, porting or restyling any UI in a project that uses Angular Material. Covers delivering a custom look as a Material theme (mat.theme, mat.theme-overrides, mat.<component>-overrides), using Material components wherever Material has the thing, styling with --mat-sys-* tokens, and when the app's own classes and tokens are allowed.
---

# Angular Material conventions

Layered on `angular:conventions` and Material's own documentation
(<https://material.angular.dev/guide/theming>).

## 0. Applies when

`@angular/material` is in a `package.json`.

## Overrides

| Topic | Tempting default | Override | Why |
| --- | --- | --- | --- |
| A custom look | the app's own CSS classes and tokens over plain elements | a **Material theme**: `mat.theme()` (colours, type, density) plus `mat.theme-overrides()` and `mat.<component>-overrides()` on `:root` and on each themed scope | the app *is* Material; the look reaches every Material component |
| Components | hand-built buttons, menus, tabs | the Material component wherever Material has one (buttons, icon buttons, toolbar, tabs, tree, menus, selects, sliders, toggles, chips, dialogs, sidenav) | behaviour, accessibility and theming for free |
| Colours and sizes in component styles | the app's own variables | Material's `--mat-sys-*` tokens | one token set, themed in one place |
| App classes and tokens | anywhere | only where Material has no component (a panel, a bar's look, a badge) | they extend Material, never replace it |

## Rules

- Themes nest: setting a theme attribute on any element re-themes Material inside it (name the
  attribute plainly, e.g. `theme`, per `angular:conventions`).
- If the theme is generated from a source (tokens file, design system), change the source and
  regenerate; never edit the generated SCSS.
- Before building a component, check whether Material has it, and whether the design system says
  which Material component to use; follow that.

**Mistakes to avoid:**

- Converting components from `--mat-sys-*` to the app's own tokens, or Material components to
  custom classes, while restyling. The user was furious when this happened.
- Loading a theme mapping as plain CSS instead of `mat.theme` / overrides mixins.
- Re-implementing a Material component because its default look differs from the design: theme it.

## Check it

Look at the change in the running app, in each theme the app offers.
