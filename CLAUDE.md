# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

선비칼국수 예비점주 적성·인성 검사 웹앱. 단일 HTML 파일(`index.html`)로 구성된 정적 웹앱으로 GitHub Pages를 통해 배포됩니다.

**배포 URL:** `https://sunbikalguksu.github.io/sunbi-test/`

## Architecture

빌드 도구 없이 단일 `index.html` 파일에 HTML/CSS/JS가 모두 포함된 구조입니다.

### 외부 서비스
- **Firebase Realtime Database** (`https://sunbi-test-default-rtdb.firebaseio.com/`)
  - `/records` — 응시자 검사 결과 저장
  - `/config/gk` — Gemini API 키 (Base64 인코딩 저장)
- **Google Gemini API** (`gemini-2.5-flash`, thinkingBudget:0) — AI 성향 분석
  - 키는 Firebase에서 런타임에 fetch 후 `atob()`으로 디코딩하여 사용

### 주요 데이터 흐름
1. 응시자 정보 입력 → 20문항 검사 → `showResult()` 호출
2. `calcScores()` → 4개 섹션 점수 계산 → 종합 판정
3. `loadDB()` / `saveDB()` — Firebase REST API로 localStorage 대신 클라우드 저장
4. `runAI()` — Firebase에서 키 fetch → Gemini API 호출 → AI 성향 분석 표시

### 관리자 기능
- 비밀번호: `ADMIN_PW` 상수 (코드 내 정의)
- 관리자 패널: 응시자 현황 조회, 개별 삭제, 전체 초기화

## Git Automation

`post-commit` hook이 설정되어 있어 커밋 시 자동으로:
1. `git push origin main` 실행
2. `C:/Users/강한빛/Desktop/index.html` 복사 (바탕화면 자동 업데이트)

## API 키 관리 원칙

저장소가 **Public**이므로 API 키를 코드에 직접 작성하면 GitHub/Google 스캐너에 즉시 감지되어 차단됩니다. 키는 반드시 Firebase에 인코딩하여 저장하고 런타임에 fetch해야 합니다.

```javascript
// 키 저장 (Firebase에 Base64 인코딩)
PUT /config/gk.json  ← btoa("AIzaSy...")

// 키 사용 (런타임 디코딩)
const keyRes = await fetch(".../config/gk.json");
const apiKey = atob(await keyRes.json());
```
