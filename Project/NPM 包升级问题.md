### 1. eslint-config-prettier `v7.5.0` > `v8.0.0`

**改动**：所有的 `prettier/*` 配置，全部都移到一个里面去。

改动之前

```json
{
  "extends": [
    "prettier",
    "prettier/@typescript-eslint",
    "prettier/babel",
    "prettier/flowtype",
    "prettier/react",
    "prettier/standard",
    "prettier/unicorn",
    "prettier/vue"
  ]
}
```

改动之后

```json
{
  "extends": [
    "prettier"
  ]
}
```

### 2. babel-eslint `v10.1.0`

该包已经被废弃，需要改动为 `@babel/eslint-parser`