# 🔎 AI 섹터 저평가 종목 탐색기

매일 07:00 KST에 **인프라/반도체 · 클라우드/데이터센터 · 모델/소프트웨어 · AI 애플리케이션** 4개 섹션의
시장 동향을 요약하고, **가치 대비 저평가된 AI 종목 Top 3**를 정량 분석으로 선별해 Jekyll 블로그에 자동 포스팅합니다.

> ⚠️ 본 리포트는 재무제표 기반의 정량적 분석 자료이며, 실제 투자 결과에 대한 책임은 투자자 본인에게 있습니다.

## 분석 로직

1. 4개 섹션 전 종목의 **PER · PBR · EV/EBITDA · ROIC · PEG**를 수집·계산.
2. 섹션 내 **동종업계 평균** 대비 멀티플(PER·PBR·EV/EBITDA)이 낮은 종목을 1차 후보군으로 선정.
3. **ROIC가 높고**(수익성 우수), **PEG가 1 미만**(성장 대비 저평가)인 종목에 가점.
4. `종합점수 = 멀티플 할인폭 + 0.35·ROIC가점 + 0.50·PEG가점` 상위 3종목을 **Top 3**로 선정.

데이터 소스: 국내 종목은 **DART 공시 최신 분기 보고서**, 해외 종목은 **Yahoo Finance**.

## 디렉터리 구조 (Hugo)

```
ai-stock-blog/
├─ scripts/analyze_stock.py      # 핵심 분석 로직 + 리포트 생성
├─ data/sample_data.json         # 오프라인/예시용 표본 데이터
├─ content/posts/                # 생성된 일일 리포트 (YYYY-MM-DD-daily-ai-top3.md)
├─ layouts/                      # 자체 Hugo 레이아웃(외부 테마 미사용)
├─ static/css/style.css          # 스타일
├─ hugo.toml                     # Hugo 설정
├─ .github/workflows/deploy.yml  # 매일 07:00 KST 생성 + Hugo 빌드 + Pages 배포
└─ requirements.txt
```

## 사용법

```bash
# 1) 의존성 설치
pip install -r requirements.txt

# 2) 표본 데이터로 즉시 리포트 생성 (API 키 불필요 — 예시/검증용)
python scripts/analyze_stock.py --mode sample --date 2026-06-02

# 3) 실데이터로 생성 (DART_API_KEY 환경변수 필요)
export DART_API_KEY="발급받은_키"
python scripts/analyze_stock.py --mode live

# 표준출력 미리보기
python scripts/analyze_stock.py --mode sample --stdout | less
```

## 자동화

- `.github/workflows/deploy.yml`이 매일 **22:00 UTC(= 07:00 KST)** 실행됩니다.
  리포트 생성 → `content/posts/`에 커밋 → Hugo 빌드 → GitHub Pages 배포.
- 저장소 **Settings → Secrets → Actions**에 `DART_API_KEY`를 등록하세요.
  (미등록 시 국내 종목은 Yahoo Finance로 자동 폴백합니다.)
- **Settings → Pages → Source**를 반드시 **GitHub Actions**로 설정하세요(브랜치 배포 아님).

## 로컬 미리보기

```bash
hugo server -D     # http://localhost:1313/ai-stock-blog/
hugo --minify      # public/ 로 정적 빌드
```

## 분석 유니버스 수정

종목 추가/삭제는 `scripts/analyze_stock.py`의 `UNIVERSE` 딕셔너리에서 합니다.
국내 종목은 `source: "dart"`와 OpenDART `corp_code`를, 해외 종목은 `source: "yahoo"`와 티커를 지정합니다.
