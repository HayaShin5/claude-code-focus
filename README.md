# Claude Code Focus

Claude Codeの状態に応じてVSCodeのターミナルパネルを自動制御する拡張機能。

自律作業中はターミナルパネルを閉じて画面を広く使い、ユーザーの確認が必要になったら自動でターミナルを開いてフォーカスします。

## 動作

| 状態 | ターミナルパネル | ステータスバー |
|---|---|---|
| `working` | 閉じる | 🤖 Claude: 作業中 |
| `waiting` | 開く＋フォーカス | ⚠️ Claude: 要確認 |
| `idle` | 閉じる | ✅ Claude: 完了 |
| 起動時 | 変更しない | 💤 Claude: 待機中 |

## 仕組み

```
Claude Code (hooks)                 VSCode拡張機能
──────────────────                  ──────────────────────
Notification発火
  → ~/.claude/vscode-status に      ← fs.watch で監視
    "waiting" を書き込む
                                        → ターミナルを開く＋フォーカス

PostToolUse発火
  → "working" を書き込む            ←
                                        → ターミナルを閉じる

Stop発火
  → "idle" を書き込む               ←
                                        → ターミナルを閉じる
```

## セットアップ

### 1. 拡張機能のインストール

```bash
# リポジトリをクローンしてビルド
npm install
npm run compile

# VSIXパッケージを作成してインストール
npx vsce package
code --install-extension claude-code-focus-0.0.1.vsix
```

### 2. Claude Code hooks の設定

`~/.claude/settings.json` に以下を追加:

```json
{
  "hooks": {
    "Notification": [
      {
        "matcher": "",
        "hooks": [
          { "type": "command", "command": "echo 'waiting' > ~/.claude/vscode-status" }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "",
        "hooks": [
          { "type": "command", "command": "echo 'working' > ~/.claude/vscode-status" }
        ]
      }
    ],
    "Stop": [
      {
        "matcher": "",
        "hooks": [
          { "type": "command", "command": "echo 'idle' > ~/.claude/vscode-status" }
        ]
      }
    ]
  }
}
```

## 開発

```bash
npm install
npm run watch    # TypeScriptをウォッチモードでコンパイル
```

`F5` で拡張機能のデバッグ実行。手動テスト:

```bash
echo 'waiting' > ~/.claude/vscode-status
echo 'working' > ~/.claude/vscode-status
echo 'idle' > ~/.claude/vscode-status
```
