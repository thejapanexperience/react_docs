# Prettier

- install prettier globally
- add .prettierrc in root with `{}` for default settings
```
{
  "singleQuote": true
}
```
- set `editor.formatOnSave` to true
- set `prettier.requireConfig` to true

eslint - add prettier and prettier/react
```
"extends": [
    "@bedegaming/eslint-config-bede",
    "plugin:react/recommended",
    "prettier",
    "prettier/react"
  ],
  ```
