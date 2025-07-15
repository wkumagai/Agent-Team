# Agent-Team

PRESIDENT agent <-> BOSS agent <-> Worker agent 1,2,3  の階層型指示システム

### 👥 エージェント構成

```
📊 PRESIDENT セッション (1ペイン)
└── PRESIDENT: プロジェクト統括責任者

📊 multiagent セッション (4ペイン)  
├── boss1: チームリーダー
├── worker1: 実行担当者A
├── worker2: 実行担当者B
└── worker3: 実行担当者C
```

## 🚀 クイックスタート

### 0. リポジトリのクローン

```bash
git clone https://github.com/wkumagai/Agent-Team.git
```

### 1. Window 1: President Agent

⚠️ **注意**: 既存の `multiagent` と `president` セッションがある場合は自動的に削除されます。また，Claude Codeが起動したら、各エージェントはBashツールを使用してコマンドを実行します。直接コマンドを入力するのではなく、Claudeに指示を与えてBashツールでコマンドを実行させてください。

Terminalウィンドウを開く．
```bash
cd Agent-Team
```
```bash
./setup.sh
```
```bash
tmux attach-session -t president
```
```bash
tmux send-keys -t president 'claude --dangerously-skip-permissions' C-m
```
認証プロンプトに従って許可を与えてください。

### 2. Window 2: Team Agents

別のTerminalウィンドウを開く．

```bash
cd Agent-Team
```

```bash
tmux attach-session -t multiagent
```

```bash
for i in {0..3}; do tmux send-keys -t multiagent:0.$i 'claude --dangerously-skip-permissions' C-m; done
```


### 4. デモ実行

PRESIDENTセッションで直接入力：
```
あなたはpresidentです。指示書に従って
```

## 📜 指示書について

各エージェントの役割別指示書：
- **PRESIDENT**: `instructions/president.md`
- **boss1**: `instructions/boss.md` 
- **worker1,2,3**: `instructions/worker.md`

**Claude Code参照**: `CLAUDE.md` でシステム構造を確認

**要点:**
- **PRESIDENT**: 「あなたはpresidentです。指示書に従って」→ Bashツールでboss1に指示送信
- **boss1**: PRESIDENT指示受信 → Bashツールでworkers全員に指示 → 完了報告
- **workers**: Hello World実行 → 完了ファイル作成 → 最後の人が報告

**⚠️ 重要な注意事項:**
- 各エージェントはメッセージ送信時に必ずBashツールを使用してコマンドを実行する必要があります
- 指示書に記載されているコマンドは、Claudeのインターフェースで直接実行するのではなく、Bashツールを通じて実行してください

## 🎬 期待される動作フロー

```
1. PRESIDENT → boss1: "あなたはboss1です。Hello World プロジェクト開始指示"
2. boss1 → workers: "あなたはworker[1-3]です。Hello World 作業開始"  
3. workers → ./tmp/ファイル作成 → 最後のworker → boss1: "全員作業完了しました"
4. boss1 → PRESIDENT: "全員完了しました"
```

## 🔧 手動操作

### agent-send.shを使った送信

```bash
# 基本送信
./agent-send.sh [エージェント名] [メッセージ]

# 例
./agent-send.sh boss1 "緊急タスクです"
./agent-send.sh worker1 "作業完了しました"
./agent-send.sh president "最終報告です"

# エージェント一覧確認
./agent-send.sh --list
```

**注意**: Claude Code内でメッセージを送信する場合は、Bashツールを使用して上記のコマンドを実行してください。

## 🧪 確認・デバッグ

### ログ確認

```bash
# 送信ログ確認
cat logs/send_log.txt

# 特定エージェントのログ
grep "boss1" logs/send_log.txt

# 完了ファイル確認
ls -la ./tmp/worker*_done.txt
```

### セッション状態確認

```bash
# セッション一覧
tmux list-sessions

# ペイン一覧
tmux list-panes -t multiagent
tmux list-panes -t president
```

## 🔄 環境リセット

```bash
# セッション削除
tmux kill-session -t multiagent
tmux kill-session -t president

# 完了ファイル削除
rm -f ./tmp/worker*_done.txt

# 再構築（自動クリア付き）
./setup.sh
```

## 🐛 トラブルシューティング

### PRESIDENTからBOSSにメッセージが送信されない場合

1. **問題**: PRESIDENTが指示書に従ってもメッセージが送信されない
   - **原因**: Claude CodeがBashツールを使用せずに直接コマンドを実行しようとしている
   - **解決策**: 指示書が更新されていることを確認し、Claude Codeに「Bashツールを使用して」コマンドを実行するよう明示的に指示する

2. **確認方法**:
   ```bash
   # 送信ログを確認
   cat logs/send_log.txt | grep president
   ```
   PRESIDENTからの送信記録がない場合は、Bashツールが使用されていない

3. **手動テスト**:
   ```bash
   # コマンドラインから直接テスト
   ./agent-send.sh boss1 "テストメッセージ"
   ```

---

## 📄 ライセンス

このプロジェクトは[MIT License](LICENSE)の下で公開されています。

