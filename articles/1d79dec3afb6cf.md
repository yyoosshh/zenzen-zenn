---
title: "GitHub Actions Cache利用で速度改善"
emoji: "👋"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [GitHubActions, cache]
published: false
---

GitHub Actionsにおいて、cache利用をして速度改善してみた

### 利用したもの

https://github.com/actions/cache

### 完成品(workflow)

```yaml
name: Node.js CI

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v3

    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '14'

    - name: Cache node modules
      id: cache-node-modules
      uses: actions/cache@v3.3.3
      with:
        path: ~/.npm
        key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
        restore-keys: |
          ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
          ${{ runner.os }}-node-

    - name: Install dependencies
      if: steps.cache-node-modules.outputs.cache-hit != 'true'
      run: npm install

    - name: Run build
      run: npm run build

    - name: Run tests
      run: npm test

```

### 改善前

```yaml
name: Node.js CI

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v3

    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '14'

    - name: Install dependencies
      run: npm install

    - name: Run build
      run: npm run build

    - name: Run tests
      run: npm test

```

