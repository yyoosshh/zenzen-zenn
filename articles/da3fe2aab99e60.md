---
title: "【GitHubActions】"
emoji: "🐥"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: []
published: false
---

## 概要

GitHubActionsを使用して、実行するコンポーネントを動的に選択できるワークフローの作成方法を紹介します。`workflow_dispatch`イベントと`matrix`戦略を組み合わせることで、柔軟で効率的なCI/CDパイプラインを構築できます。

## ワークフロー例

```yaml
name: exec matrix component

on:
  workflow_dispatch:
    inputs:
      components:
        description: '実行するコンポーネントを選択（カンマ区切りで複数可）'
        required: true
        type: choice
        options:
          - action1
          - action2
          - action3
          - action1,action2
          - action1,action3
          - action2,action3
          - action1,action2,action3
      environment:
        description: '実行環境を選択'
        required: true
        type: choice
        options: [development, staging, production]
      debug_mode:
        description: 'デバッグモードを有効化'
        required: true
        type: boolean
        default: false
      version:
        description: 'バージョン番号（例: 1.2.3）'
        required: true
        type: string

jobs:
  run-component-actions:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        component: ${{ fromJson(format('[{0}]', github.event.inputs.components)) }}

    steps:
    - uses: actions/checkout@v4.1.7

    - name: ${{ matrix.component }} アクション実行
      uses: ./component/${{ matrix.component }}
      with:
        environment: ${{ github.event.inputs.environment }}
        debug_mode: ${{ github.event.inputs.debug_mode }}
        version: ${{ github.event.inputs.version }}
        component_param1: value1
        component_param2: value2
```

## ワークフローの詳細説明

1. **`workflow_dispatch`イベント**: GitHubのUI上でワークフローを手動実行できるようにします。

2. **動的な入力パラメータ**:
   - `components`: 実行するコンポーネントを選択（複数選択可能）
   - `environment`: 実行環境の選択（開発/ステージング/本番）
   - `debug_mode`: デバッグモードの有効/無効(ワークフローの詳細なログが見ることができます)
   - `version`: バージョン番号の指定

3. **`matrix`戦略**: 選択されたコンポーネントごとに並行ジョブを実行します。`fromJson`と`format`関数を使用して、選択されたコンポーネントをJSON配列に変換します。

4. **コンポーネント実行**: 各ジョブで`./component/`ディレクトリ内の指定されたコンポーネントアクションを実行します。

## パラメータの活用

- `environment`: デプロイ環境や環境固有の設定を選択
- `debug_mode`: 詳細なログ出力や特定のデバッグ機能を有効化
- `version`: 特定バージョンのコードデプロイやバージョン管理に使用
- `component_param1`, `component_param2`: コンポーネント固有のパラメータ設定

## ワークフローの利点

1. **柔軟性**: 必要なコンポーネントのみを選択して実行可能
2. **効率性**: 並行実行による時間短縮
3. **拡張性**: 新コンポーネントの追加が容易
4. **カスタマイズ性**: 実行時にパラメータを調整可能

## 使用シーン

このワークフローは、マイクロサービスアーキテクチャや複数の独立したコンポーネントを持つプロジェクトで特に有効です。

開発者は特定のコンポーネントのみをテストしたり、デプロイしたりすることができ、CI/CDプロセスの効率を大幅に向上させることができます。

## まとめ

GitHubActionsの`workflow_dispatch`と`matrix`戦略を組み合わせることで、柔軟で効率的なCI/CDパイプラインを構築できます。

このアプローチにより、開発チームは必要に応じて特定のコンポーネントを選択的に実行し、開発プロセスを最適化することができます。
