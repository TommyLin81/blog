---
title: "MCP (Model Context Protocol) Guide"
date: 2025-07-07T17:38:10+08:00
description: "Anthropic 定義的開放協議，讓 AI Agent 的 tool calling 可以透過標準化的 MCP 介面使用第三方工具，方便使用者自行擴充 AI Agent 能調用的工具。"
tags: [mcp, guide]
categories: [AI]
---

## TL;DR

Anthropic 定義的開放協議，讓 AI Agent 的 tool calling 可以透過標準化的 MCP 介面使用第三方工具，方便使用者自行擴充 AI Agent 能調用的工具。

## Background

因為 LLM 是語言對話的模型，AI Agent 需要開發許多外部工具來協助建立 LLM 所需的文本上下文（Context），例如使用 web search 工具將網站整理成內容大綱的描述、使用 calculator 工具將文本不擅長的複雜數學計算做預先處理。

過去這些工具都是由 AI Agent 的開發商進行開發和整合，現在則可以透過介接 MCP 介面，來讓使用者自行擴充 AI Agent 可以使用的工具。

## MCP Architecture

### Architecture & Components

{{< figure src="/images/mcp-architecture.png" alt="mcp architecture" height="500" >}}

- **MCP Host**：使用工具完成任務的 LLM 應用程式（Claude Desktop、Cursor）
- **MCP Client**：管理與 Server 通訊的客戶端
- **MCP Server**：提供具體工具和資源的服務端
- **通訊協議**：基於 JSON-RPC 2.0 標準

## Transport

MCP Client 和 MCP Server 之間有兩種通訊方式：

- Standard I/O
- HTTP/SSE & Streamable HTTP

### Standard I/O

透過 command-line interface，並以 standard input 和 standard output 串流實現。

相關的特性：

- Process-based: 服務透過 client 端的子進程來執行
- Simple to implement: 只需要從 stdin 和 stdout 讀取和寫入資料
- Suitable for local tools: server 和 client 都可以執行在同一台機器上

使用情境:

- CLI-based Services
- file operations
- local database

### HTTP/SSE & Streamable HTTP

相關特性如下：

- Network-based: 可以透過 URL 進行存取
- Scalable: 可以遠端訪問，並且提供給多個 client 使用
- Stateful: 可以維持持久連線

使用情境:

- Web-Based services
- cloud APIs
- distributed system

## What Make an MCP Successful

透過觀察市面上實用的 MCP Server，整理出一些符合市場需求的 MCP 應該具備的幾個特點：

### 1. 資料轉換能力

將非文字、複雜的團隊資產轉換成 LLM 可以清楚閱讀的格式

- **[Jira & Confluence MCP](https://community.atlassian.com/forums/Atlassian-Platform-articles/Using-the-Atlassian-Remote-MCP-Server-beta/ba-p/3005104)**：將任務票、文件轉換為結構化的 Markdown 格式
- **[Figma-Context-MCP](https://github.com/GLips/Figma-Context-MCP)**：提取設計稿的結構和註解資訊
- **[AWS Labs MCP](https://github.com/awslabs/mcp)**：將雲端架構與狀態轉換為可閱讀的格式

### 2. 即時資訊獲取

提供各領域最新資訊，解決 LLM 訓練資料過時問題

- **[Context7](https://github.com/upstash/context7)**：獲取最新的技術文件與範例
- **[AWS Documentation MCP](https://github.com/awslabs/mcp/tree/HEAD/src/aws-documentation-mcp-server)**：獲取最新的 AWS 文件

### 3. 互動執行環境

提供處理與執行平台，讓 LLM 可以驗證解決方案並自我除錯

- **[Desktop Commander](https://github.com/wonderwhy-er/DesktopCommanderMCP)**：檔案系統操作和程式執行環境

## References

- [Introduction - Model Context Protocol](https://modelcontextprotocol.io/introduction)
