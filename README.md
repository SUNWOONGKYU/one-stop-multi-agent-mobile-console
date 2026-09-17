# One-Stop Multi-Agent Mobile Console (원스톱 멀티 에이전트 모바일 콘솔)

> 어떤 PC에서든 더블 클릭 한 번으로 스마트폰과 AI 워커 3종(Antigravity · Codex CLI · Claude Code CLI)을 연결하는 무설치 포터블 스타터 킷.
> 폰은 북마크 하나, PC는 켜 두기만. 지시는 폰에서, 실행과 파일은 전부 PC 안에서.

**소개 페이지:** https://one-stop-multi-agent-mobile-console.vercel.app/

## 한눈에

| | |
|---|---|
| 필요한 것 | Node.js가 깔린 Windows PC 1대, 스마트폰 브라우저 |
| 설치 | 없음 — 폴더 하나(cloudflared · ttyd 실행 파일 내장) |
| 네트워크 설정 | 없음 — 포트 개방·공유기·VPN 불필요 (Cloudflare 터널) |
| 폰 접속 | 통합 허브 주소 1개 북마크 → 켜진 PC마다 탭(상태등) → 탭 터치로 PC 간 즉시 전환. PC별 전용 주소도 있음 |
| 워커 | 🪐 Antigravity(메인, 대화창 실시간 미러) · ⚡ Codex CLI · 🤖 Claude Code CLI (서브, 무인 데몬) |
| 콘솔 탭 | Antigravity / Codex / Claude Code / 프로젝트 탭 + 업로드 · 긴급 정지 · 단순화/상세 모드 |
| 데이터 | 클라우드에는 "어느 PC가 켜져 있고 주소가 무엇인가"만. 대화·파일·결과물은 PC 밖으로 안 나감 |

## 도해

- [관계도](docs/관계도.svg) — 스마트폰 · 작업 PC · 클라우드/AI 워커 세 열의 구성 요소와 연결
- [작업 흐름도](docs/흐름도.svg) — 기동 4단계(PC 1회) → 접속 3단계(폰 1회) → 지시·실행·회신 5단계(반복)

## 작동 방식 (12단계 요약)

**A. 기동 — PC에서 한 번**
1. `1클릭_모바일콘솔_실행.bat` 더블 클릭 → `watch_mobile_tasks.js` 백그라운드, `tunnel.js` 실행
2. `server.js`가 7890 포트에서 콘솔 화면 + API 기동
3. 내장 cloudflared가 공개 https 주소 발급
4. 기기 장부에 PC 이름·프로젝트·주소 등록, 60초마다 생존 신고

**B. 접속 — 폰에서 한 번**
5. 고정 허브 주소(북마크) 열기
6. 통합 허브가 장부를 읽어 켜진 PC마다 탭을 띄움 — 탭 터치로 PC 선택·전환 (PC별 전용 주소로 바로 열 수도 있음)
7. 터널 → `server.js` → 4탭 콘솔 화면. PIN 1회 → 세션 토큰, 이후 자동

**C. 지시 · 실행 · 회신 — 반복**
8. 탭을 고르고 지시(텍스트, 사진·파일 업로드, 요약 칩)
9. `server.js`가 길을 나눔 — Antigravity는 파일 큐(`tasks_from_mobile.json`)에 적고 watcher가 깨움 / Codex·Claude는 브릿지가 CLI를 stdin으로 바로 실행
10. 워커가 PC 안에서 작업
11. 결과 저장(`codex_messages.json` · `claude_messages.json`) / Antigravity 대화 로그는 그대로 읽어 미러
12. 폰이 1.2~1.5초 폴링으로 결과 표시 → 8로 반복

## 스타터 킷 구성

```
📁 원스톱_멀티에이전트_모바일콘솔_스타터킷/
├── 1클릭_모바일콘솔_실행.bat   # 실행기 — 더블 클릭 한 번 (올인원 start_all.js 포함)
├── PC2_전용_원클릭_실행기.bat  # PC별 실행기 — 좀비 프로세스·포트 정리 후 기동, 같은 Wi-Fi 직통 주소 안내
├── 영구_자동승인_등록기.bat    # Antigravity CLI 무인 자동 승인 등록 (폰 지시 시 PC 앞 확인 팝업 제거)
├── server.js                  # 모바일 콘솔 풀스택 서버 (4탭 UI + API, 7890)
├── tunnel.js                  # 터널 개통 + 기기 장부 등록 + 60초 하트비트
├── live_sync_engine.js        # Antigravity 대화 로그 → 폰 미러 (읽기 전용)
├── codex_bridge.js            # Codex CLI 실행·결과 저장
├── claude_bridge.js           # Claude Code CLI 실행·결과 저장
├── watch_mobile_tasks.js      # 폰 지시 감지 → Antigravity 깨우기
├── sync_urls.js               # 접속 링크 파일 자동 생성
├── delegate.js · send_reply_to_mobile.js · cli_sync.js
├── cloudflared.exe · ttyd.exe # 내장 실행 파일 (무설치)
└── *.json                     # 파일 큐 (지시 · 응답 · 장부)
```

## 설계 원칙

- **PIN 한 번, 이후 토큰** — 토큰 없는 API 요청은 전부 거절
- **클라우드에는 주소만** — 허브·장부는 켜진 PC와 주소만 안다
- **WebSocket 없이 폴링 + JSON 파일 큐** — 부품이 적어 끊겨도 스스로 돌아온다
- **폴더 하나가 전부** — 복사해서 어느 PC, 어느 프로젝트에 붙여도 동작

## 이 저장소에 있는 것 / 없는 것

이 저장소는 **소개서와 도해**입니다. 스타터 킷의 실행 코드(허브 주소·PIN·장부 식별자 포함)는 운영자 개인 환경에 묶여 있어 공개 저장소에 넣지 않았습니다.

## 만든 사람

선웅규 — AI Master · 공인회계사(CPA) · 스타트업 액셀러레이터 · https://saa.house/about

© 2026 Sun Woongkyu
