# Handoff: 워크보드 (Workboard) — 노드 그래프 캔버스 UI 리디자인

## Overview
개인용 "워크보드" — 프로젝트/작업/아이디어를 노드로 배치하고 선으로 연결하는 무한 캔버스 보드입니다. 기존 앱(백엔드 연동 실사용 중)의 **UI/UX를 리디자인**한 프로토타입으로, 다음을 다룹니다: 상태·진행률이 보이는 노드 카드, 곡선 연결선, 그룹 프레임, 상태 필터, 커맨드 팔레트, 미니맵, 인라인 편집, 다중 선택, 되돌리기, 노드별 체크리스트.

## About the Design Files
이 번들의 `워크보드.dc.html`은 **HTML로 만든 디자인 레퍼런스**입니다 — 의도한 모습과 동작을 보여주는 프로토타입이지, 그대로 복사해 쓰는 프로덕션 코드가 아닙니다. (내부적으로 "Design Component"라는 스트리밍 렌더 포맷을 쓰므로 브라우저에서 바로 열어 보긴 어렵고, 스크린샷/설명과 함께 참조하세요.)

할 일은 이 디자인을 **대상 코드베이스의 기존 환경(React/Vue 등)과 패턴·라이브러리로 재현**하는 것입니다. 이미 워크보드 앱이 있으므로, 기존 캔버스/상태 관리 구조 위에 이 문서의 컴포넌트·인터랙션·토큰을 얹으면 됩니다. 노드 그래프 렌더링은 직접 구현해도 되고, **React Flow / Svelte Flow / vue-flow** 같은 라이브러리 위에 이 비주얼 토큰을 입혀도 됩니다(추천 — 팬/줌/엣지/미니맵/드래그연결이 기본 제공).

## Fidelity
**High-fidelity (hifi)** — 최종 색상·타이포·간격·인터랙션이 확정된 목업입니다. 색/폰트/반경/그림자 값을 그대로 재현하세요. 단, 노드 좌표와 체크리스트 항목 등 **콘텐츠는 더미 데이터**이며, 실제 데이터로 대체됩니다.

---

## Screens / Views
단일 풀스크린 뷰입니다: **무한 캔버스 + 떠 있는 오버레이 UI**.

### 레이어 구조 (z-index 순)
1. **캔버스 뷰포트** (`position:absolute; inset:0`) — 팬/줌 대상. 빈 곳 드래그 = 팬, Shift+드래그 = 박스 선택, 더블클릭 = 노드 생성, 휠 = 줌.
   - 내부에 **월드 레이어** (`transform: translate(panX,panY) scale(scale); transform-origin:0 0`): 점 그리드 배경, 프레임, SVG 엣지, 노드가 모두 이 좌표계에 배치됨.
2. **오버레이 (z:20~40)** — 상단 툴바, 좌측 상태 필터/범례, 좌하단 줌 컨트롤, 우하단 미니맵, 하단 컨텍스트 바, 우측 상세 드로어, 커맨드 팔레트.

### 좌표/변환 모델
- 각 노드는 월드 좌표 `{x, y, w, h}`(px)를 가짐.
- 화면 좌표 = `world * scale + pan`. 역변환(클릭→월드): `worldX = (clientX - rectLeft - panX) / scale`.
- 줌은 커서 기준: `newPan = cursor - (cursor - pan) * (newScale/oldScale)`, scale clamp `[0.35, 2.2]`.

---

## Components

### 1) 노드 카드 (일반)
- 컨테이너: `background: linear-gradient(180deg,#171b21,#12161b)`, `border:1px solid rgba(255,255,255,.08)` (선택 시 `rgba(accent,.6)`), `border-radius:13px`, `padding:13px 14px 13px 17px`, `overflow:hidden`.
- **왼쪽 액센트 바**: `position:absolute; left:0; top:0; bottom:0; width:3px; background:<accent>`.
- **그림자**: 기본 `0 6px 20px -12px rgba(0,0,0,.6)`; 선택 시 `0 0 0 1.5px rgba(accent,.7), 0 12px 34px -10px rgba(0,0,0,.7)`.
- **제목**: 700/14px, `color:#f2f5f8`, `letter-spacing:-.01em`.
- **상태 배지** (있을 때): 10.5px/500, `padding:2px 8px`, `border-radius:20px`, `background:rgba(accent,.14)`, `color:<accent>`.
- **서브텍스트**: 12px, `color:#8a94a2`, `line-height:1.45`, `margin-top:4px`.
- **메타 행** (`margin-top:10px`): 액센트 점(5px 원) + 상대시간(11px, `#6b7482`) + **파일 칩**(IBM Plex Mono 10.5px, `padding:2px 7px`, `border-radius:6px`, `background:rgba(255,255,255,.05)`, `border:1px solid rgba(255,255,255,.07)`, `color:#9aa4b2`).
- **진행률 바** (체크리스트 있을 때, `margin-top:11px`): 트랙 `height:5px; border-radius:3px; background:rgba(255,255,255,.08)`; 채움 `width:<done/total %>; background:<accent>; transition:width .25s`; 우측 라벨 `n/m` (Mono 10.5px, `#8a94a2`).
- **즐겨찾기 별** (starred): 우상단 `★` `#fbbf24` 12px.
- **연결 핸들**: 우측 중앙 파란 점, `right:-5px; top:50%`, 11px 원, `background:#3b82f6`, `border:2px solid #0b0d10`, `box-shadow:0 0 8px rgba(59,130,246,.7)`, `cursor:crosshair`. 여기서 드래그 시작 → 다른 노드에 놓으면 엣지 생성.
- 노드는 `cursor:grab`, 드래그로 이동. 필터로 흐려질 때 `opacity:.2; transition:opacity .18s`.

### 2) 허브 노드 (카테고리 라벨용)
- `background:#15191f`, `border:1px solid`(동일 규칙), `border-radius:11px`, `padding:11px 15px`, 700/14px.
- 앞에 7px 사각 점(액센트, `box-shadow:0 0 10px <accent>`). 서브/메타 없음.

### 3) 그룹 프레임 (자동 래핑 컨테이너)
- 멤버 노드들의 바운딩 박스 + 패딩(좌우 30, 상 38, 하 26)으로 **자동 계산**되어 노드 이동 시 따라 감싸짐.
- 펼침: `border:1px dashed rgba(color,.28)`, `background:rgba(color,.045)`, `border-radius:20px`, `pointer-events:none`. 좌상단에 라벨 칩(클릭 시 접기): `background:#12161b`, `border:1px solid rgba(color,.28)`, `border-radius:20px`, 11px/600, 색점 + 라벨 + `▼`.
- 접힘: 콤팩트 pill `background:#15191f`, `border-radius:12px`, `▶` + 색점 + 라벨 + `· N개`. 멤버 노드와 관련 엣지는 숨김.

### 4) SVG 엣지 (연결선)
- 소스 우측 중앙 → 타깃 좌측 중앙. 곡선: `M sx sy C sx+dx sy, tx-dx ty, tx ty` (`dx=max(40, |tx-sx|*0.5)`). 직선 옵션도 있음(tweak `edgeStyle`).
- 기본 stroke `rgba(255,255,255,.14)`, width 1.4.
- **활성**(양 끝 중 선택 노드 포함): stroke = 선택 노드 accent, width 2, `stroke-dasharray:6 8` + `animation: dashflow 1s linear infinite` (`@keyframes dashflow { to { stroke-dashoffset:-28 } }`) — 흐르는 효과.
- **선택된 엣지**: stroke `#f87171`, width 2.5, 중점에 삭제 마커(반경 10 원 `#f87171` + 흰 ×). 클릭으로 선택하려면 각 엣지에 투명 히트 패스(stroke-width 16, `pointer-events:stroke`)를 겹쳐 둠. SVG 컨테이너는 `pointer-events:none`, 개별 패스만 활성화.

### 5) 상단 툴바 (z:20, 글래스)
- `background:rgba(18,22,28,.78)`, `backdrop-filter:blur(14px)`, `border:1px solid rgba(255,255,255,.08)`, `border-radius:13px`, `box-shadow:0 8px 30px -12px rgba(0,0,0,.6)`.
- 좌측 그룹: 로고(16px 사각, `linear-gradient(135deg,#a78bfa,#60a5fa)`) + "워크보드" + `+ 노드`(그라디언트 `linear-gradient(135deg,#7c6bf0,#5b8def)`, 흰 글자) + `스캔`(돋보기 아이콘) + `새로고침` + `되돌리기`(⟲ 아이콘, `disabled` 시 `#4a525e`).
- 가운데: 단축키 힌트 pill.
- 우측 그룹: 동기화 상태(초록 점 + 시각) + `내보내기` + `가져오기`.
- 보조 버튼 공통: `padding:6px 10px`, `border:1px solid rgba(255,255,255,.09)`, `border-radius:9px`, `background:rgba(255,255,255,.03)`, `color:#c3cbd6`, 12.5px/500.

### 6) 상태 필터 / 범례 (좌상단, z:20)
- 노드 status 값들을 자동 집계해 목록화 (색점 + 라벨 + 개수 Mono). 클릭 시 해당 상태만 남기고 나머지 노드 `opacity:.2`. 선택 항목 `background:rgba(color,.15)`, `color:<color>`. "전체 보기"로 해제.

### 7) 줌 컨트롤 (좌하단) / 미니맵 (우하단)
- 줌: `+` / 배율% (Mono) / `−` / 맞춤(fit) 버튼 세로 스택, 글래스.
- 미니맵: 212×142, 모든 노드를 축소 배치(색상 유지) + 현재 뷰포트 사각(보라 테두리). 클릭/드래그로 팬, 더블클릭으로 해당 위치 1.3배 확대.

### 8) 컨텍스트 바 (하단 중앙, 선택 1개일 때, z:25)
- 글래스, `border-radius:14px`, `animation: fadein .18s`. 구성: 색점 + 제목 + **색상 스와치 12개**(클릭 시 accent 변경) + 즐겨찾기(★ 토글) + `상세`(드로어 토글) + `삭제`(빨강).
- 색상 팔레트(12): `#8a94a2 #60a5fa #818cf8 #a78bfa #22d3ee #34d399 #a3e635 #fbbf24 #fb923c #f87171 #f472b6 #e879f9`.

### 9) 다중 선택 바 (선택 2개 이상)
- "N개 선택" + 색상 스와치(전체 적용) + "모두 삭제".

### 10) 상세 드로어 (우측, z:24)
- `background:rgba(15,18,23,.92)`, `backdrop-filter:blur(18px)`, `border-radius:16px`, `top:70; right:16; bottom:74; width:290`.
- **편집 폼**: 제목 input, 상태 input, 설명·메모 textarea (모두 `background:#0e1116`, `border:1px solid rgba(255,255,255,.12)`, `border-radius:8px`) — 입력 즉시 노드에 반영.
- 읽기 정보: 최근 활동, 파일 칩, 연결 개수.
- **체크리스트 섹션**: 헤더(라벨 + `n/m`) + 진행률 바 + 항목 리스트(체크박스 16px, 완료 시 accent 채움 + 흰 체크 아이콘 + 취소선 `#6b7482`; × 삭제) + 하단 "항목 추가" input + `추가` 버튼(Enter 지원).

### 11) 커맨드 팔레트 (⌘K, z:40)
- 중앙 상단 모달(520px), 딤 배경 `rgba(6,8,11,.55)` + blur. 검색 input + `ESC` 칩 + 결과 리스트(색점 + 제목 + 상태). 항목 클릭 → 해당 노드로 팬 이동 + 선택.

---

## Interactions & Behavior
- **팬**: 빈 캔버스 드래그. **줌**: 휠(커서 기준). **줌 버튼/맞춤**: 중앙 기준 / 전체 fit.
- **노드 이동**: 노드 드래그. 다중 선택 상태면 선택 전체가 함께 이동(드래그 시작 시 각 노드 시작 좌표 스냅샷).
- **노드 생성**: `+ 노드`(뷰 중앙) 또는 빈 곳 더블클릭(클릭 지점).
- **인라인 편집**: 노드 더블클릭 → 제목 input(Enter 저장, Esc 취소, blur 저장).
- **연결 생성**: 노드 우측 파란 점에서 드래그 → 임시 점선(파랑) 따라오다가 다른 노드 위에서 놓으면 엣지 추가(중복/자기연결 방지).
- **연결 삭제**: 엣지 클릭 → 빨강 선택 + 중점 × → × 클릭 또는 Delete.
- **다중 선택**: Shift+드래그 박스. 박스와 겹치는 노드 선택. Delete로 일괄 삭제, 스와치로 일괄 색 변경.
- **상태 필터**: 범례 클릭 → 비매칭 노드 디밍.
- **프레임 접기/펼치기**: 라벨/칩 클릭.
- **되돌리기**: ⌘Z(또는 툴바 버튼). 이동·생성·삭제·색변경·즐겨찾기·연결·편집·체크 모두 히스토리 대상(스택 상한 60).
- **키보드**: ⌘K 팔레트, Esc(팔레트/드로어/선택 해제), Delete/Backspace(선택 노드·엣지 삭제; input 포커스 시 무시).
- **애니메이션**: 활성 엣지 dashflow(1s linear infinite), 오버레이 등장 `fadein .18s`, 진행률/디밍 transition.

## State Management
핵심 상태(현재 프로토타입 기준):
- `nodes: [{ id, title, accent(hex), status?, subtitle?, time?, file?, x, y, w, h, starred?, hub?, checklist?: [{t, done}] }]`
- `edges: [[fromId, toId], ...]`
- 뷰: `panX, panY, scale`, 측정된 `containerW/H`
- 선택: `selectedId`(주), `selectedIds[]`(다중), `selectedEdge`("a__b")
- 편집: `editingId, editTitle`, `newCheck`(체크 추가 입력)
- UI: `paletteOpen, detailOpen, query, filterStatus, collapsed{frameId:bool}, box(선택 사각), link(연결 드래그 중)`
- `history[]`(undo 스냅샷: nodes + edges 딥카피)

전이 트리거: 위 Interactions 참조. 데이터 fetch/저장은 프로토타입에 없음 — **실제 앱의 노드/엣지 저장소(현 백엔드)에 연결**하고, 각 mutation을 저장 API로 위임하세요. `history`는 클라이언트 undo용으로 유지.

> **진행률 근거**: 순수하게 `체크된 항목 수 / 전체 항목 수`. 자동 판정 로직 없음(수동 체크). 필요 시 커밋/파일 스캔·status·하위 노드 롤업 등으로 자동화 가능(미구현).

## Design Tokens
- **배경**: 앱 `#0b0d10` (+ 상단 방사형 하이라이트 `radial-gradient(120% 90% at 50% -10%, #12161c, #0b0d10 60%)`); 카드 그라디언트 `#171b21→#12161b`; 허브/입력 `#15191f` / `#0e1116`; 글래스 패널 `rgba(18,22,28,.78~.9)` + `blur(14~18px)`.
- **텍스트**: 주 `#f2f5f8`, 본문 `#e7ebf0`, 보조 `#c3cbd6` / `#9aa4b2` / `#8a94a2`, 흐림 `#6b7482` / `#5f6875`.
- **테두리**: `rgba(255,255,255,.06~.12)`.
- **액센트/상태색**: violet `#a78bfa`, indigo `#818cf8`, blue `#60a5fa`, cyan `#22d3ee`, emerald `#34d399`, lime `#a3e635`, amber `#fbbf24`, orange `#fb923c`, red `#f87171`, pink `#f472b6`, fuchsia `#e879f9`, slate `#8a94a2`. 연결 핸들 `#3b82f6`. 별 `#fbbf24`.
- **반경**: 카드 13, 허브 11, 프레임 20, 배지/칩 6~20, 버튼 8~9, 패널 12~16.
- **그림자**: 카드 `0 6px 20px -12px rgba(0,0,0,.6)` / 선택 링 포함 `0 0 0 1.5px rgba(accent,.7), 0 12px 34px -10px rgba(0,0,0,.7)`; 패널 `0 8px 30px -12px rgba(0,0,0,.6)`; 드로어 `0 20px 60px -18px rgba(0,0,0,.8)`; 팔레트 `0 30px 80px -20px rgba(0,0,0,.85)`.
- **타이포**: UI = **IBM Plex Sans KR** (400/500/600/700), 코드/파일/수치 = **IBM Plex Mono** (400/500). 크기: 제목 14, 배지/메타 10.5~12, 본문 12~13.5. `letter-spacing:-.01~-.02em`(제목), `.08~.1em`+uppercase(섹션 라벨).
- **그리드 배경**: `radial-gradient(circle, rgba(255,255,255,.055) 1px, transparent 1px)`, `background-size:26px 26px` (월드 레이어에 적용 → 팬/줌 따라감).

## Assets
- 이미지 없음. 아이콘은 인라인 SVG(돋보기/맞춤/되돌리기/체크/×)와 유니코드(`★ ▼ ▶ + − ×`).
- 폰트: Google Fonts — IBM Plex Sans KR, IBM Plex Mono. (사내 코드베이스에 이미 폰트 시스템이 있으면 그걸 우선.)

## Files
- `워크보드.dc.html` — 전체 프로토타입(마크업 + 로직). 색/간격/좌표/핸들러의 **정본 소스**. Design Component 포맷이라 브라우저 직접 실행은 어렵고, 위 명세와 함께 읽으세요. 필요하면 대화에서 스크린샷을 요청하세요.
