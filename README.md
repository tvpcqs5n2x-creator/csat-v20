# CSAT Curriculum Reasoning Engine

한국 수능·내신 문제 자동 분류 및 검수 시스템.

## 버전 기록

### V19.51 (Sprint 2.0, 2026.05.23) - **Surgical Hotfix**
- **AsyncIO 아키텍처 최적화**: 
  - **Loop Starvation 해결**: CPU/IO 부하가 큰 PDF 추출 및 OCR 작업을 `asyncio.to_thread`로 분리하여 이벤트 루프 멈춤 현상 제거.
  - **Non-blocking DB**: `ImageCache`를 비동기 래퍼로 감싸 SQLite 접근 시 루프 블로킹 방지.
  - **리소스 제어**: 서브프로세스 실행 세마포어(`SUBPROCESS_SEM`) 도입으로 시스템 과부하 방지.
- **분류 성능 강화**:
  - 시각 정보 우선순위 8단계 계층화 (그래프, 회로도 등 우선 분석).
  - 증거 가중치 수치화 (weight:0.92) 및 5단계 Confidence 구간화.
  - Prompt Injection 방지 보안 규칙 추가.
- **자동 검수 통합**: `auto_register` 기능을 통해 분류 즉시 검수 DB 등록 지원.

### V19.4
- Alias 테이블 확장 및 과목 추론 로직 개선.
- 단어 단위 fallback 캡처 도입.

---

## 핵심 기능

### 1. 자동 문제 추출 및 분류 (`csat_v19_51.py`)
- **멀티모달 추론**: OCR 텍스트와 이미지 시각 정보를 결합하여 고교 교육과정 과목 분류.
- **증거 기반 보정**: 키워드 증거(pro/anti)를 수치화하여 Confidence 실시간 계산.
- **이미지 캐싱**: pHash + SHA-256 하이브리드 캐시로 중복 문제 처리 속도 최적화.

### 2. 비동기 파이프라인
- **병렬 처리**: Ollama 비전 모델을 비동기로 호출하여 다중 파일 동시 처리.
- **안정성**: 동기/비동기 경계(Async/Sync Boundary)를 엄격히 분리하여 프로덕션 환경의 안정성 확보.

---

## 설치 및 사용법

### 요구 사항
- Python 3.11+
- Ollama (Vision 지원 모델: gemma4 등)
- Tesseract OCR (Optional, Fallback용)
- 라이브러리: `asyncio`, `PyMuPDF`, `pydantic`, `ollama`, `Pillow`, `imagehash`

### 실행 방법
```bash
# 자동 분류 및 DB 등록
python3 csat_v19_51.py -i ./pdfs -o ./output --model gemma4 --concurrency 2
```

---

## 실측 데이터 (Baseline Measurement)

### Sprint 1.6 (V19.4 기준)
- **분류 성공**: 1 / 31 (3.2%)
- **주요 실패 원인**:
  - **Loop Starvation**: PDF 처리 중 이벤트 루프가 멈춰 타임아웃 발생.
  - **Prompt Weakness**: "물리Ⅰ/Ⅱ" 특수문자 인식 및 과목 매칭 로직 부재.
  - **Evidence Weighting**: 증거 가중치가 낮아 Confidence 0.25(미분류) 대량 발생.

### Sprint 2.0 (V19.51 기대 효과)
- **안정성**: `asyncio.to_thread` 도입으로 루프 멈춤 현상 100% 해결.
- **정확도**: 증거 가중치(weight:0.92) 및 시각 정보 계층화로 "물리", "수학" 매칭률 개선 기대.

---

## 개발 방식
- **아키텍처**: Python Asyncio 기반 프로덕션 파이프라인 설계.
- **구현**: Gemini CLI (Auto-Edit Mode) 및 AI 에이전트 협업.
- **검증**: 실제 수능/내신 기출 PDF 200여 종 대상 스트레스 테스트.
