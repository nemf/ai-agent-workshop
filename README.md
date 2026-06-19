# AI エージェントワークショップ — 環境準備ガイド（Claude Code 版）

Claude Code on Amazon Bedrock を使って AI エージェントを開発します。

## 前提条件

- AWS アカウント（マネジメントコンソールにログインできること）
- VS Code Server（ブラウザで開ける状態）

---

## 準備 1 / 5：アクセスキーの発行（AWS CloudShell）

AWS CloudShell でスクリプトを実行し、IAM ユーザー・ポリシー・アクセスキーを一括作成します。

### 手順

1. マネジメントコンソールの検索バーで「**CloudShell**」を検索 → 開く
2. 以下のコマンドを順番に実行

```bash
git clone https://github.com/nemf/ai-agent-workshop.git
cd ai-agent-workshop
chmod +x script.sh
./script.sh
```

4. 出力された **AccessKeyId** と **SecretAccessKey** をメモ・保存

```json
{
  "AccessKey": {
    "UserName": "bedrock-agent-dev",
    "AccessKeyId": "AKIA...",
    "SecretAccessKey": "...",
    "Status": "Active"
  }
}
```

> ⚠️ **SecretAccessKey は再表示できません。** スクリプト実行直後に必ずメモしてください。紛失した場合はキーを削除して再発行が必要です。

### スクリプトが自動で行うこと

| ステップ | 内容 |
|---------|------|
| 1 | IAM ユーザー `bedrock-agent-dev` を作成 |
| 2 | カスタムポリシー `AIAgentWorkshopPolicy` を作成 |
| 3 | ポリシーをユーザーにアタッチ |
| 4 | アクセスキーを発行 |

---

## 準備 2 / 5：AWS CLI の設定（VS Code Server）

VS Code Server のターミナルでアクセスキーを登録します。

### 手順

1. VS Code Server をブラウザで開く
2. 上部メニュー「**ターミナル**」→「**新しいターミナル**」
3. 以下を実行

```bash
aws configure --profile bedrock-dev
```

4. 4 つの質問に回答

| 質問 | 入力する値 |
|------|-----------|
| AWS Access Key ID | CloudShell で取得した `AKIA...` |
| AWS Secret Access Key | CloudShell で取得したシークレットキー |
| Default region name | `us-west-2` |
| Default output format | `json` |

5. プロファイルを有効化

```bash
export AWS_PROFILE=bedrock-dev
```

6. （推奨）ターミナル起動時に自動で有効化

```bash
echo 'export AWS_PROFILE=bedrock-dev' >> ~/.bashrc
source ~/.bashrc
```

7. 設定確認

```bash
aws sts get-caller-identity
```

`"Arn"` に `bedrock-agent-dev` が含まれていれば OK。

---

## 準備 3 / 5：Claude Code のインストールと Bedrock 接続設定

Claude Code は Amazon Bedrock 経由で利用します。インストールと環境設定の手順は、以下の公式ワークショップに従ってください。

> 📘 **Claude Code on Amazon Bedrock**
> https://catalog.workshops.aws/claude-code-on-amazon-bedrock/ja-JP/lab-1/environment-configuration

ポイント:

- Bedrock を使うため、`CLAUDE_CODE_USE_BEDROCK=1` と `AWS_REGION=us-west-2` を設定します（詳細は上記ワークショップ参照）。
- **Amazon Q Developer のような SSO 認証（`*.awsapps.com` へのアクセス）は不要です。** 準備 2 で設定した IAM アクセスキー（`bedrock-dev` プロファイル）の認証情報をそのまま利用します。

設定が完了したら、ターミナルで `claude` を起動して動作を確認してください。

---

## 準備 4 / 5：MCP サーバーの設定

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
| 認証 | 準備 2 で設定した IAM 認証情報（SigV4）を `mcp-proxy-for-aws` が橋渡し。ドキュメント取得は認証不要 |

> 💡 AWS MCP Server は AgentCore のドキュメント検索、CDK/CloudFormation 操作、AWS API 実行を 1 つに集約しています。AgentCore のデプロイや Lambda・DynamoDB のインフラ操作もこのサーバーから行えます。

### 反映

Claude Code を再起動し、`/mcp` と入力して `aws-mcp` が接続済みであることを確認します。

---

## 準備 5 / 5：CLAUDE.md の設定

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

- [ ] `aws sts get-caller-identity` で `bedrock-agent-dev` が表示される
- [ ] `claude` を起動でき、Bedrock 経由で応答が返る
- [ ] `/mcp` で `aws-mcp` が接続済みと表示される
- [ ] `.mcp.json` がプロジェクト直下にある
- [ ] `CLAUDE.md` がプロジェクト直下にある

すべて ✓ なら準備完了です。ワークショップを始めましょう！
