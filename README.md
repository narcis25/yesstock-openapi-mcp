# 예스스탁 모의 OpenAPI MCP 커넥터

Claude · Cursor · Windsurf · VS Code · Codex 등에서 예스스탁 모의 OpenAPI 를 쓰기 위한
MCP 커넥터입니다. 모델이 API 를 **찾고, 읽고, 실행할** 수 있게 합니다.

> 이 저장소는 **배포용**입니다. 소스는 두지 않고, 릴리스 자산만 올립니다.

필요한 것은 개발자 포털에서 발급받은 **access_key / secret_key** 둘입니다.
**자격증명은 이 PC 를 벗어나지 않습니다** — 커넥터가 게이트웨이에서 직접 토큰을 받아
쓰고, 모델도 사용자도 토큰을 보지 않습니다.

---

## 설치 — Claude Desktop

**가장 쉬운 경로입니다. 따로 설치할 것이 없습니다** — Node 는 Claude 가 함께 제공합니다.

1. [최신 릴리스](../../releases/latest)에서 **`yesstock-openapi-mcp.mcpb`** 를 받습니다.
2. 파일을 더블클릭합니다. Claude 가 설치 창을 엽니다.
3. 발급받은 값을 입력합니다.

| 항목              | 설명                                         |
| ----------------- | -------------------------------------------- |
| 게이트웨이 URL    | 기본값 `https://test-openapi.yesstock.com`   |
| Access Key        | 발급받은 `access_key`                        |
| Secret Key        | 발급받은 `secret_key`                        |
| 종목마스터 구분값 | 비워두면 주식 전체. 순수 주식만 원하면 `8,9` |

---

## 설치 — 그 밖의 클라이언트

**[Node 20.11 이상](https://nodejs.org)이 필요합니다.** 설정 파일에 아래 블록을 넣고
클라이언트를 **완전히 재시작**하세요. macOS 는 창을 닫는 것이 아니라 ⌘Q 로 끝내야
설정이 다시 읽힙니다.

```json
{
  "mcpServers": {
    "yesstock": {
      "command": "npx",
      "args": ["-y", "yesstock-openapi-mcp@latest"],
      "env": {
        "WEBOPENAPI_GATEWAY_URL": "https://test-openapi.yesstock.com",
        "WEBOPENAPI_ACCESS_KEY": "발급받은_access_key",
        "WEBOPENAPI_SECRET_KEY": "발급받은_secret_key"
      }
    }
  }
}
```

넣을 파일은 클라이언트마다 다릅니다.

| 클라이언트         | 설정 파일                                                                                                                       |
| ------------------ | ------------------------------------------------------------------------------------------------------------------------------- |
| **Claude Desktop** | macOS `~/Library/Application Support/Claude/claude_desktop_config.json`<br>Windows `%APPDATA%\Claude\claude_desktop_config.json` |
| **Cursor**         | 전역 `~/.cursor/mcp.json` · 프로젝트 `.cursor/mcp.json`                                                                          |
| **Windsurf**       | `~/.codeium/windsurf/mcp_config.json`                                                                                           |
| **Gemini CLI**     | `~/.gemini/settings.json`                                                                                                       |

**VS Code** 는 최상위 키가 `mcpServers` 가 아니라 `servers` 이고 `type` 이 필요합니다.
`.vscode/mcp.json` 에 넣습니다.

```json
{
  "servers": {
    "yesstock": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "yesstock-openapi-mcp@latest"],
      "env": {"...": "위와 동일"}
    }
  }
}
```

**Codex** 는 JSON 이 아니라 TOML 입니다. `~/.codex/config.toml` 에 넣습니다.

```toml
[mcp_servers.yesstock]
command = "npx"
args = ["-y", "yesstock-openapi-mcp@latest"]
env = { WEBOPENAPI_GATEWAY_URL = "https://test-openapi.yesstock.com", WEBOPENAPI_ACCESS_KEY = "...", WEBOPENAPI_SECRET_KEY = "..." }
```

### 명령 한 줄로

파일을 열지 않아도 되는 클라이언트도 있습니다.

```bash
claude mcp add yesstock \
  --env WEBOPENAPI_GATEWAY_URL=https://test-openapi.yesstock.com \
  --env WEBOPENAPI_ACCESS_KEY=... \
  --env WEBOPENAPI_SECRET_KEY=... \
  -- npx -y yesstock-openapi-mcp@latest
```

### 그 밖의 설정값

`env` 에 함께 넣을 수 있습니다. 전부 선택입니다.

| 환경변수                      | 용도                                                           |
| ----------------------------- | -------------------------------------------------------------- |
| `WEBOPENAPI_SYMBOL_TYPES`     | 종목마스터 구분값. 비우면 주식 전체, 순수 주식만 쓰려면 `8,9`   |
| `WEBOPENAPI_EXCHANGE_CODE`    | 거래소코드. 기본 `XKRX`                                        |
| `WEBOPENAPI_NO_UPDATE_CHECKS` | `1` 이면 기동할 때 새 버전을 확인하지 않습니다 (`.mcpb` 에만 해당) |

---

## 쓰는 법

모델에게 하고 싶은 일을 말하면 됩니다. 커넥터가 툴 4개를 제공합니다.

| 툴               | 하는 일                         |
| ---------------- | ------------------------------- |
| `search_apis`    | 쓸 수 있는 API 검색             |
| `describe_api`   | 파라미터·주의사항·예제·코드샘플 |
| `call_api`       | API 실행                        |
| `search_symbols` | 종목명·종목코드로 종목 찾기     |

검색과 조회는 **API 목록을 커넥터가 들고 있어** 게이트웨이가 멈춰도 동작합니다.
종목마스터는 첫 조회 때 메모리에 올라가고, 그 뒤로는 네트워크를 쓰지 않습니다.

**위험한 API 는 실행 전에 확인을 받습니다.** 주문 접수처럼 되돌릴 수 없는 호출은
모델이 사용자에게 먼저 묻고, 확인해야 실행됩니다.

---

## 새 버전 받기

**설치 경로에 따라 방식이 다릅니다.**

**`.mcpb` 로 설치했다면** 커넥터가 스스로 합니다. 기동할 때 이 저장소의 최신 릴리스를
확인하고, 새 버전이 있으면 받아서 적용합니다. 하루에 한 번만 확인하고, 실패하면 그냥
현재 버전으로 뜹니다. 받은 파일은 **SHA-256 으로 검증한 뒤에만** 적용하고, 새 버전이
기동에 실패하면 직전 버전으로 자동으로 되돌립니다.

끄려면 커넥터 설정에서 *자동 업데이트 끄기* 를 켜거나 `WEBOPENAPI_NO_UPDATE_CHECKS=1`
을 줍니다. 폐쇄망이거나 버전을 고정해 배포할 때 필요합니다.

**npm 으로 설치했다면** `@latest` 가 그 일을 합니다. 설정에서 빼면 캐시에 남은 옛
버전으로 계속 돌 수 있으니 붙여두세요.

반대로 기동할 때 네트워크를 쓰면 안 되는 환경이라면, 전역 설치 후 명령을 고정합니다.

```bash
npm i -g yesstock-openapi-mcp
# "command": "yesstock-openapi-mcp", "args": []
```

---

## 안 될 때

MCP 커넥터는 실패해도 화면에 아무것도 보이지 않습니다. 아래부터 확인하세요.

**도구 4개가 아예 안 보입니다**

- 클라이언트를 **완전히** 종료했다 켰는지 확인하세요 (macOS 는 ⌘Q).
- `.mcpb` 로 설치했다면 커넥터가 기동 기록을 남깁니다.
  macOS `~/Library/Application Support/yesstock-openapi-mcp/launch.log`
  Windows `%LOCALAPPDATA%\yesstock-openapi-mcp\launch.log`
- npm 으로 설치했다면 터미널에서 직접 돌려보세요. 같은 내용이 그대로 보입니다.
  ```bash
  npx -y yesstock-openapi-mcp@latest
  ```

**`spawn npx ENOENT`**

클라이언트가 GUI 로 떠서 셸의 PATH 를 물려받지 못한 경우입니다 (nvm 이나 Apple
Silicon Homebrew 에서 가끔 납니다). `which npx` 결과를 `command` 에 절대경로로 적으세요.

**도구는 보이는데 호출하면 401**

키 오타입니다. `env` 의 `WEBOPENAPI_ACCESS_KEY` / `WEBOPENAPI_SECRET_KEY` 두 줄을
확인하세요.

---

## 릴리스 자산

| 파일                        | 받는 쪽                                   |
| --------------------------- | ----------------------------------------- |
| `yesstock-openapi-mcp.mcpb` | 사람. Claude Desktop 설치에 이 파일만 받으면 됩니다 |
| `payload.mjs`               | 커넥터가 자동 업데이트에 씁니다           |
| `update.json`               | 커넥터가 버전과 체크섬을 확인합니다       |

뒤의 둘은 직접 받을 일이 없습니다. npm 경로는 릴리스 자산을 쓰지 않습니다.

---

## 문제가 있으면

[Issues](../../issues) 에 남겨주세요. 다음이 있으면 도움이 됩니다.

- 커넥터 버전과 **설치 방법** (`.mcpb` 인지 npm 인지)
- 무엇을 시켰고 무엇이 돌아왔는지
- 운영체제
