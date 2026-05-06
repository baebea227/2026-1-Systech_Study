# HTML 생성 규칙

모든 HTML 페이지는 `background.html`의 디자인 시스템을 따른다.
스타일 상세는 `background.html`을 직접 참조한다.

---

## 페이지 골격

```
header
└── .container > h1 + p + .badge

div.container
├── 목록으로 링크 (← 목록으로)
├── nav.toc
├── section#s1 ~ section#sN
└── div.nav-between
footer
```

- 각 `<section>`에 반드시 `id="sN"` 부여 (TOC 앵커용)
- 첫 페이지(이전 없음)면 `.nav-between`에서 왼쪽 버튼 생략, `justify-content: flex-end`

---

## CSS 변수

```css
:root {
    --navy:    #003366;
    --blue:    #004a99;
    --accent:  #c9a227;
    --light:   #f7f9fc;
    --border:  #e2e8f0;
    --code-bg: #1e293b;
    --code-fg: #e2e8f0;
    --success: #16a34a;
    --warn:    #d97706;
    --info:    #2563eb;
}
```

---

## 컴포넌트 목록

| 클래스 / 태그 | 용도 | 비고 |
|---|---|---|
| `header` | 그라디언트 헤더 | h1 + p + `.badge` |
| `nav.toc` | 목차 | `<ol>` 사용 |
| `section` | 본문 카드 | h2 > h3 > h4 계층 |
| `pre > code` | 코드 블록 | 어두운 배경 |
| `code` (인라인) | 인라인 코드 | 밝은 배경 |
| `table` | 일반 표 | |
| `.cmp-table` | 비교 표 | `.row-label` / `.good` / `.bad` / `.mid` |
| `.callout-info` | 정보 콜아웃 | 파란 왼쪽 테두리 |
| `.callout-warn` | 주의 콜아웃 | 주황 왼쪽 테두리 |
| `.callout-success` | 핵심 콜아웃 | 초록 왼쪽 테두리 |
| `.figure` | 다이어그램 영역 | `.figure-caption` 함께 사용 |
| `.cmd-grid` | 2열 카드 그리드 | |
| `.cmd-card.reset/open/close/finish` | 명령 카드 4종 | 각각 빨강/파랑/주황/초록 |
| `.nav-between` | 페이지 이동 버튼 | `.nav-btn` |
| `.bullet-list` | 커스텀 불릿 목록 | 하위 목록은 `.sub` |
| `footer` | 하단 | 중앙 정렬, 작은 글씨 |

---

## 체크리스트

- [ ] `:root` CSS 변수 포함
- [ ] Google Fonts `<link>` 포함 (Noto Sans KR + JetBrains Mono)
- [ ] `header > .container > h1 + p + .badge` 구조
- [ ] `nav.toc` → `section#sN` 순서
- [ ] `div.nav-between` 으로 마무리
- [ ] `footer` 포함
- [ ] `@media (max-width: 640px)` 반응형 블록 포함
