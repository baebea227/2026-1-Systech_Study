# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 프로젝트 개요

2026-1 시스템최신기술 강의의 ZNS SSD 실습 가이드 사이트입니다. 빌드 시스템이나 패키지 매니저 없이 **순수 정적 HTML 파일**로 구성됩니다.

## 로컬 미리보기

```bash
# Python 내장 서버로 브라우저 확인
python3 -m http.server 8080
# → http://localhost:8080/index.html
```

## 파일 구조

| 파일 | 역할 |
|------|------|
| `index.html` | 메인 목차 (실습 흐름 전체 안내) |
| `background.html` | ZNS 개념 배경 지식 |
| `setup.html` | QEMU + null_blk 환경 구성 |
| `lab1.html` | 순차/랜덤 쓰기 성능 비교 |
| `lab2.html` | Zone 상태 전이 실습 |
| `lab3.html` | F2FS 레벨 WAF 측정 |
| `lab4.html` | Zoned UFS Case Study (FAST '26) |

## 공통 CSS 설계 패턴

각 HTML 파일은 CSS를 인라인으로 포함하며, 모든 파일이 동일한 CSS 변수와 컴포넌트 클래스를 **복사·공유**합니다 (공통 파일 없음).

**CSS 변수 (`--navy`, `--blue`, `--accent`, `--light`, `--border`, `--success`, `--warn`, `--danger`)** — 모든 색상은 이 변수를 통해 참조합니다.

**재사용 컴포넌트 클래스:**
- `.code-block` + `.copy-btn` — 쉘 명령어 코드 블록 (copy 버튼 포함)
- `.callout.{info|warn|ok|danger}` — 색상 구분 알림 박스
- `.output-block` — 예상 터미널 출력 표시
- `.step-flow` / `.step-entry` / `.step-dot` — 번호 달린 단계 흐름 레이아웃
- `.result-table` — 실습 결과 기록표

**코드 하이라이팅**: `<span class="{comment|kw|str|fn}">` 태그로 수동 마크업합니다.

**copy 버튼 동작**: `copyCode(btn)` JS 함수가 각 HTML 파일 하단 `<script>` 블록에 동일하게 삽입되어 있습니다.

## 실습 환경 (VM)

- QEMU VM (Linux Kernel 5.18.0) 에 SSH 접속: `ssh -p 2222 user@localhost`
- **`/dev/nullb0`**: Conventional SSD 역할 — `null_blk`, `memory_backed=1`, 2GB, non-zoned
- **`/dev/nullb1`**: ZNS SSD 역할 — `null_blk`, `memory_backed=1`, 2GB, Zone 64MB × 32

## HTML 수정 시 유의사항

- 한 파일의 CSS 변수나 공통 컴포넌트를 변경할 경우, **동일 내용이 모든 HTML에 중복**되어 있으므로 나머지 파일도 함께 수정해야 합니다.
- 코드 블록 내 `<`, `>`, `&`는 HTML 엔티티(`&lt;`, `&gt;`, `&amp;`)로 이스케이프되어 있습니다.
- 쉘 파이프 `|`는 코드 블록 내에서 그대로 쓰되, `2>&1`은 `2>&amp;1`로 표기합니다.
