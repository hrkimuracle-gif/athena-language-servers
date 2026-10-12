# Athena 언어 서버 마켓플레이스

Athena Code 의 LSP 심볼 도구(find_symbol, find_referencing_symbols, get_symbols_overview, get_call_hierarchy, get_hover_info, find_implementations)가 쓰는 언어 서버를 플러그인 형태로 배포하는 마켓플레이스 저장소입니다.

- 저장소: https://github.com/hrkimuracle-gif/athena-language-servers (임시. 이후 사내 git 으로 이전 예정)
- 마켓플레이스 이름: `athena-language-servers`
- 플러그인: `lsp-typescript` (설치 ID `lsp-typescript@athena-language-servers`)

## 사용 방법

1. Athena Code 플러그인 설정에서 마켓플레이스 소스를 추가합니다.
   - 주소: `https://github.com/hrkimuracle-gif/athena-language-servers.git`
   - 인증: 없음 (공개 저장소)
2. 마켓플레이스 목록에서 "TypeScript 언어 서버" 를 설치하고 활성화합니다.
   활성화 시 신뢰 확인 창에 "언어 서버" 항목이 표시됩니다. 네이티브 프로그램을 실행하는 권한이므로 내용을 확인한 뒤 승인합니다.
3. 설정 → 문서 및 인덱싱 → 언어 서버 (LSP) 섹션에 TypeScript 가 "활성화됨" 으로 보이면 준비된 것입니다.
   워크스페이스에 `.ts`/`.js` 파일이 있을 때만 목록에 나타납니다.

Athena Code 는 언어 서버 바이너리를 다음 순서로 찾습니다: 설치된 마켓플레이스 플러그인 → (설정에서 허용한 경우) 공개 저장소 자동 설치.
플러그인으로 설치하면 자동 설치 동의와 무관하게, 오프라인에서도 동작합니다.

## 실행 요구사항

- `node` 가 PATH 에 있어야 합니다. 서버 진입점(`lib/cli.mjs`)이 `#!/usr/bin/env node` 로 실행되고, Windows 에서는 `bin/typescript-language-server.cmd` 가 `node` 를 호출합니다.
- TypeScript 컴파일러는 플러그인에 동봉되어 있어 워크스페이스에 typescript 가 설치돼 있지 않아도 됩니다. 워크스페이스에 typescript 가 있으면 typescript-language-server 가 그쪽을 우선 사용합니다.

## 저장소 구성

```
marketplace.json                                      마켓플레이스 정의
plugins/lsp-typescript/
  .athena-plugin/plugin.json                          플러그인 정의 (이름, 버전, 설명, languageServers 경로)
  .language-servers.json                              서버 ID 와 플랫폼별 실행 명령 (Athena Code 가 로드 시 검증)
  bin/typescript-language-server.cmd                  Windows 실행 래퍼
  package.json, package-lock.json                     동봉 버전 고정 (typescript-language-server 4.3.4, typescript 5.7.3)
  node_modules/                                       동봉 서버와 컴파일러 (약 24 MB, 162 항목)
```

`node_modules/.bin` 은 넣지 않습니다. Athena Code 플러그인 설치기는 심볼릭 링크를 거부하므로 `.language-servers.json` 의 명령은 실제 파일(`./node_modules/typescript-language-server/lib/cli.mjs`)을 가리킵니다.
명령은 반드시 `./` 로 시작하는 패키지 내부 경로이거나 PATH 의 단순 명령이어야 하고, 패키지 밖(`../`)이나 절대 경로는 거부됩니다.

## 서버 갱신

1. `plugins/lsp-typescript/package.json` 의 버전을 올리고 `npm install --omit=dev` 를 다시 실행합니다.
2. `rm -rf node_modules/.bin` 으로 심볼릭 링크를 제거합니다.
3. `.athena-plugin/plugin.json` 의 `version` 을 올립니다.
4. 커밋하고 푸시하면 마켓플레이스에서 새 버전으로 보입니다.

## 플러그인 추가

언어마다 `plugins/<이름>/` 을 같은 구조로 만들고 `marketplace.json` 의 `plugins` 에 항목을 추가합니다.
`serverId` 는 Athena Code 의 어댑터가 쓰는 이름과 같아야 합니다: `typescript-language-server`, `pyright`, `jdtls`, `gopls`, `rust-analyzer`, `clangd`, `kotlin-language-server`, `csharp-ls`.
플랫폼 키는 `*`, `darwin`, `linux`, `win32`, 또는 `darwin-arm64` 처럼 `<플랫폼>-<아키텍처>` 입니다. 구체적인 키가 공통 키보다 우선합니다.

## 포함 서버

- typescript-language-server 4.3.4 (typescript-language-server/typescript-language-server, Apache-2.0)
- typescript 5.7.3 (microsoft/TypeScript, Apache-2.0)
