# moodboard-plugin

키워드 한 줄로 핀터레스트 레퍼런스를 자동 수집하고 무드보드를 만드는 Claude 플러그인.

## 기능
- 핀터레스트에서 키워드로 이미지 30장 자동 수집 (Playwright MCP)
- Vision AI로 톤·소재별 자동 분류
- 폴더별 정리된 무드보드 완성

## 사용법
```
/moodboard {키워드}
```

**예시:**
```
/moodboard 미니멀 베이지 카페
```

## 설치 방법
Claude Desktop → 개인 플러그인 [+] → 마켓 플레이스 추가 → `{owner}/team-market` 입력 → 동기화

또는 직접 플러그인 폴더를 등록:
Claude Desktop → 개인 플러그인 [+] → 플러그인 생성 → 이 폴더 선택

## 요구사항
- Claude Desktop
- Playwright MCP (`npx @playwright/mcp@latest`)
- Node.js 18 이상

## 파일 구조
```
moodboard-plugin/
├── .claude-plugin/
│   └── plugin.json       # 플러그인 정보
├── .mcp.json             # Playwright MCP 등록
├── commands/
│   └── moodboard.md      # /moodboard 커맨드
├── skills/
│   ├── moodboard/
│   │   └── SKILL.md      # 이미지 수집 스킬
│   └── sort-files/
│       └── SKILL.md      # 이미지 분류 스킬
└── README.md
```
