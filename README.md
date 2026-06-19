# AI エージェントワークショップ — 環境準備ガイド（Claude Code 版）

Claude Code on Amazon Bedrock を使って AI エージェントを開発します。

> ℹ️ **Claude Code 本体のインストールと Amazon Bedrock への接続設定は、ワークショップ環境（VS Code Server）に事前セットアップ済みです。** このガイドでは、AWS 認証情報の設定・MCP サーバー・プロジェクトルールの 3 点だけを準備します。

## 前提条件

- AWS マネジメントコンソールにログインできること
- VS Code Server（ブラウザで開ける状態。Claude Code セットアップ済み）

---

## 準備 1 / 3：AWS 認証情報の設定

マネジメントコンソールの「**Get credentials**」で取得した認証情報を、VS Code Server に設定します。

### 手順

1. マネジメントコンソール（または Workshop Studio）の「**Get credentials**」を開く
2. 表示された認証情報をコピーする（`AWS_ACCESS_KEY_ID` / `AWS_SECRET_ACCESS_KEY` / `AWS_SESSION_TOKEN`）
3. VS Code Server のターミナル（上部メニュー「**ターミナル**」→「**新しいターミナル**」）を開く
4. 「Get credentials」に表示された export 文をそのまま貼り付けて実行

```bash
export AWS_ACCESS_KEY_ID="ASIA..."
export AWS_SECRET_ACCESS_KEY="..."
export AWS_SESSION_TOKEN="..."
export AWS_REGION="us-west-2"
```

> 💡 「Get credentials」の表示形式は環境によって異なります（`~/.aws/credentials` への貼り付け形式の場合もあります）。画面の案内に従ってください。一時認証情報のため、有効期限が切れたら再取得します。

5. 設定確認

```bash
aws sts get-caller-identity
```

`Account` と `Arn` が表示されれば OK。

---

## 準備 2 / 3：MCP サーバーの設定

Claude Code に **AWS MCP Server**（2026 年 GA）を接続し、開発効率を上げます。AWS の API 実行・ドキュメント検索・スクリプト実行をこの 1 つのサーバーでまかなえます。

### 手順

プロジェクトディレクトリ直下に `.mcp.json` を作成します。

```bash
curl -o .mcp.json https://raw.githubusercontent.com/nemf/ai-agent-workshop/main/.mcp.json
```

または、Claude Code のコマンドで直接追加することもできます（ユーザースコープで全プロジェクト利用可）。

```bash
claude mcp add-json aws-mcp --scope user \
  '{"command":"uvx","args":["mcp-proxy-for-aws@latest","https://aws-mcp.us-east-1.api.aws/mcp","--metadata","AWS_REGION=us-west-2"]}'
```

#### 設定内容（`.mcp.json`）

```json
{
  "mcpServers": {
    "aws-mcp": {
      "command": "uvx",
      "args": [
        "mcp-proxy-for-aws@latest",
        "https://aws-mcp.us-east-1.api.aws/mcp",
        "--metadata",
        "AWS_REGION=us-west-2"
      ]
    }
  }
}
```

| 項目 | 内容 |
|------|------|
| サーバー | AWS MCP Server（マネージドリモート） |
| 主なツール | `call_aws`（15,000 以上の AWS API を実行）、`search_documentation` / `read_documentation`（最新ドキュメント取得）、`run_script`（サンドボックスで Python 実行） |
| 認証 | 準備 1 で設定した AWS 認証情報（SigV4）を `mcp-proxy-for-aws` が橋渡し。ドキュメント取得は認証不要 |

> 💡 AWS MCP Server は AgentCore のドキュメント検索、CDK/CloudFormation 操作、AWS API 実行を 1 つに集約しています。AgentCore のデプロイや Lambda・DynamoDB のインフラ操作もこのサーバーから行えます。

### 反映

Claude Code を再起動し、`/mcp` と入力して `aws-mcp` が接続済みであることを確認します。

---

## 準備 3 / 3：CLAUDE.md の設定

プロジェクトルールを定義して、Claude Code にプロジェクトの文脈を理解させます。

### 手順

プロジェクトディレクトリ直下に `CLAUDE.md` を配置します。

```bash
curl -o CLAUDE.md https://raw.githubusercontent.com/nemf/ai-agent-workshop/main/CLAUDE.md
```

#### 主な設定内容

- **応答ルール** — 日本語で返答、コメントも日本語
- **使用ライブラリ** — strands-agents, strands-agents-tools, pytest, ruff
- **開発フロー** — コード → テスト → テスト実行 → README 更新
- **コーディング規約** — Python 3.12+、型ヒント必須、ruff format/check
- **モデル** — Claude Opus 4.6（`us.anthropic.claude-opus-4-6-v1`）

> 💡 `CLAUDE.md` はプロジェクトのルールブック。Claude Code がこのファイルを読んで、プロジェクトの文脈を理解した上でコードを書いてくれます。
> こちらの内容はサンプルですので、適宜編集してご利用ください。

---

## チェックリスト

準備が完了したら、以下を確認してください。

- [ ] `aws sts get-caller-identity` で認証情報が表示される
- [ ] `claude` を起動でき、Bedrock 経由で応答が返る
- [ ] `/mcp` で `aws-mcp` が接続済みと表示される
- [ ] `.mcp.json` がプロジェクト直下にある
- [ ] `CLAUDE.md` がプロジェクト直下にある

すべて ✓ なら準備完了です。ワークショップを始めましょう！
