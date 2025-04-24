# 🧠 sam-serverless-mcp-server
A super simple Model Context Protocol (MCP) server deployed on AWS Lambda and exposed via Amazon API Gateway, deployed with Serverless Application Model (SAM).
This skeleton is based on the awesome work of [Frédéric Barthelet](https://github.com/fredericbarthelet): which has developed a middy middleware for Model Context Protocol (MCP) server integration with AWS Lambda functions in [this repo](https://github.com/fredericbarthelet/middy-mcp)

## Long story
📖 [Read the article of this series here on dev.to](https://dev.to/aws-builders/deploy-a-minimal-mcp-server-on-aws-lambda-with-serverless-framework-3e42)

## 🛠 Features
- 🪄 Minimal MCP server setup using @modelcontextprotocol/sdk
- 🚀 Deployed as a single AWS Lambda function
- 🌐 HTTP POST endpoint exposed via API Gateway at /mcp
- 🔄 Supports local development via SAM
- 🧪 Includes a simple example tool (add) with JSON-RPC interaction

## 📦 Project Structure
```
sam-serverless-mcp-server/
├── __tests__/              # Jest tests
├── src/                    # Source code
│   └── index.js                # MCP server handler
├── .gitignore              # Git ignore file
├── buildspec.yml           # Buildspec file for AWS CodeBuild and CodePipeline (CI/CD)
├── jest.config.mjs         # Jest config file
├── package.json            # Project dependencies
├── package-lock.json       # Project lock file
├── README.md               # This documentation file
├── samconfig.toml          # Serverless Application Model config
└── template.yml            # Serverless Application Model template
```

## 🛠 Prerequisites
- Node.js v22+ 
- Docker
- [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)
- [SAM CLI](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/install-sam-cli.html)

## 🚀 Getting Started
1. Install dependencies:
```bash
npm install
```

2. Run Locally with SAM
```bash
sam local start-api
```

Local endpoint will be available at:
POST `http://localhost:3000/mcp`

## Switch to Api Gateway V2 (HTTP API)
If you want to use API Gateway V2, you can change the `template.yml` file to use `HttpApi` instead of `Api` in the `Events` section. This will allow you to use HTTP APIs instead of REST APIs.
This will allow you to use HTTP APIs instead of REST APIs.

```yaml
Events:
  McpHttpApi:
    Type: HttpApi  # 👈 This switches to HTTP API (v2)
    Properties:
      Path: /mcp
      Method: POST
```

## 🧪 Test with curl requests
### List tools
```bash
curl --location 'http://localhost:3000/mcp' \
--header 'content-type: application/json' \
--header 'accept: application/json' \
--header 'jsonrpc: 2.0' \
--data '{
  "jsonrpc": "2.0",
  "method": "tools/list",
  "id": 1
}'
```

### ➕ Use the add Tool
```bash
curl --location 'http://localhost:3000/mcp' \
--header 'content-type: application/json' \
--header 'accept: application/json' \
--header 'jsonrpc: 2.0' \
--data '{
  "jsonrpc": "2.0",
  "id": 2,
  "method": "tools/call",
  "params": {
    "name": "add",
    "arguments": {
      "a": 5,
      "b": 3
    }
  }
}'
```

## 🧪 Test with jest
There are some basic tests included in the `__tests__` folder. You can run them with:

```bash
npm run test
```

## 🧬 Code Breakdown
This code is based on the awesome work of [Frédéric Barthelet](https://github.com/fredericbarthelet): which has developed a middy middleware for Model Context Protocol (MCP) server integration with AWS Lambda functions in [this repo](https://github.com/fredericbarthelet/middy-mcp)

### src/index.js
```javascript
import middy from "@middy/core";
import httpErrorHandler from "@middy/http-error-handler";
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { z } from "zod";
import mcpMiddleware from "middy-mcp";

const server = new McpServer({
  name: "Lambda hosted MCP Server",
  version: "1.0.0",
});

server.tool("add", { a: z.number(), b: z.number() }, async ({ a, b }) => ({
  content: [{ type: "text", text: String(a + b) }],
}));

export const handler = middy()
  .use(mcpMiddleware({ server }))
  .use(httpErrorHandler());
```

## 📡 Deploy to AWS
Just run:

```bash
sam build
sam deploy --guided
```
After deployment, the MCP server will be live at the URL output by the command.

## 📘 License
MIT — feel free to fork, tweak, and deploy your own version!

