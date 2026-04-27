# ai_presentation_v5 — 작업 가이드

이 레포는 발표 데크입니다. 다음 기능들은 **절대 제거하지 마세요**. 리팩터·재작성·디자인 정돈 시에도 동일 키바인딩과 동일 UX로 보존되어야 합니다.

## 보존해야 하는 기능

### 1. 전체화면 모드 (F)
- 키 `F`/`f` → `document.documentElement.requestFullscreen()` ↔ `document.exitFullscreen()` 토글
- 하단 오버레이의 **`FS` 버튼** 클릭으로도 토글
- WebKit 폴백(`webkitRequestFullscreen` / `webkitExitFullscreen`)도 함께 둘 것

### 2. 레이저 포인터 모드 (L)
- 키 `L`/`l` → 켜기·끄기 토글, `Esc` → 끄기
- 하단 오버레이의 **`Laser` 버튼** 클릭으로도 토글
- 켜진 동안:
  - 시스템 커서를 숨김 (`:host([data-laser]) { cursor: none; }`)
  - 빨간 점(`.laser-dot`)이 `mousemove`의 `clientX/clientY`를 따라 이동
  - 점은 `position: fixed`, `pointer-events: none`, `z-index: 2147483647` — 캔버스 `transform: scale()`과 무관하게 뷰포트 좌표계에서 동작
- 인쇄(`@media print`)에서는 `display: none !important`로 숨김

### 구현 위치
- `deck-stage.js`
  - 상태: 생성자의 `_laserOn`, `_laserDot`
  - 메서드: `_toggleFullscreen()`, `_toggleLaser(force?)`
  - 키 핸들러: `_onKey` 안 `f`/`F`, `l`/`L`, `Escape` 분기
  - 마우스 추적: `_onMouseMove(e)` 안에서 `_laserOn`일 때 dot의 `transform`을 갱신
  - 스타일: 셰도우 DOM 스타일시트의 `.laser-dot`, `:host([data-laser])`
  - DOM: `_render()`에서 `laser-dot`을 `_root`에 append
  - 오버레이 버튼: `.btn.fs`, `.btn.laser`

### 회귀 방지 체크리스트
새 변경 전에 다음을 모두 통과해야 합니다.
- [ ] `F` 키로 전체화면 진입·해제가 동작
- [ ] `L` 키로 빨간 점이 켜지고 커서가 사라짐
- [ ] `Esc` 또는 다시 `L`로 레이저 모드 해제
- [ ] 하단 오버레이의 `FS`·`Laser` 버튼이 동일 동작
- [ ] 빨간 점이 `transform: scale()`로 축소된 상태에서도 마우스 정확한 위치에 표시
- [ ] 인쇄 미리보기(Save as PDF)에서 점·오버레이 모두 숨김

## 동기화 주의
부모 레포 `Mahler90/ai_presentation`과 수동 동기 관계입니다 (자동 sync 없음).
- 부모 레포에서 가져올 때 본 파일(`CLAUDE.md`)과 위 두 기능을 덮어쓰지 않도록 확인.
- 가져온 뒤 회귀 체크리스트 재실행.
