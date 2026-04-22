# 한글 Type3 PDF → Markdown 변환 파이프라인

> **상황**: 한글 공공기관 PDF는 본문이 **Type3 임베디드 폰트 (ToUnicode 맵 없음)** 로 되어 있어 `pymupdf`, `pdfplumber`, `PyPDF2` 등 텍스트 추출 도구가 모두 한글을 CID 코드로만 반환 → 깨짐.
> **결론**: 표준 PDF 텍스트 추출 경로 포기. **렌더링 → OCR** 경로가 유일한 해.

## 적용 사례
- `F:\PROPOSAL\202604_산림청_국립산립과학원\제안요청서(...).pdf` (133p)
- 결과: `00_제안요청서/` 아래 6개 장 md + `index.md` (Agent RAG용 네비)

## 진단 체크리스트 (PDF 받은 직후)
```python
import fitz
doc = fitz.open(PDF_PATH)
# 1) Font types on sample pages
for pg in [0, len(doc)//2, len(doc)-1]:
    print(pg, {f[2] for f in doc[pg].get_fonts()})
# Type3 가 대부분이면 OCR 경로 확정
# 2) TOC (outlines)
print('toc entries:', len(doc.get_toc()))
# 공공기관 한글 PDF는 대개 toc=0 — 수동 파싱 필요
```

## 파이프라인 (Windows + Python 3.12 + uv)

### 환경
- **EasyOCR** 채택 (PaddleOCR 3.x는 oneDNN/PIR 런타임 버그로 CPU에서 실패)
- CPU 모드로 충분 (GPU 전환 시 torch cuda 빌드 재설치 필요 — 마진 크지 않음)
- `paddlepaddle` + `modelscope` 조합은 torch 설치 시 `torch.multiprocessing` import 꼬임 → 피할 것

### 설치 주의
```bash
uv venv .venv --python 3.12
uv pip install easyocr pymupdf
# 주의: easyocr는 torch CPU 버전을 자동 설치. GPU 원하면 torch+cuda 별도 재설치.
```

### 단계별 스크립트
1. `01_render_pdf.py` — `fitz`로 300 DPI PNG (133p, ~30초)
2. `02_run_ocr.py` — EasyOCR(`ko,en`), 페이지별 JSON(text+bbox+conf)
3. `03_reflow.py` — bbox 기반 행 재정렬 (y-center 그룹 → 좌→우 정렬)
4. `04_build_md.py` — TOC 수동 매핑 기반으로 6개 장 md + index.md

### 133p 처리 시간 (CPU)
- 렌더링: ~30초
- OCR: **~56분** (페이지당 20~30초)
- 리플로우 + md 빌드: <5초

## 주요 함정

### 1. Windows 콘솔 cp949 vs EasyOCR 진행바
```python
# EasyOCR 기본 진행바가 `█` (U+2588) 출력 → cp949 콘솔에서 UnicodeEncodeError로 크래시
reader = easyocr.Reader(['ko','en'], gpu=..., verbose=False)  # ← 필수
# 환경변수로 PYTHONIOENCODING=utf-8 도 함께 설정
```

### 2. OpenCV + 한글 경로
```python
# cv2.imread('한글_경로.png')  → None 반환 (Windows cp949 제약)
# 대신:
arr = np.fromfile(str(path), dtype=np.uint8)
img = cv2.imdecode(arr, cv2.IMREAD_COLOR)
reader.readtext(img, ...)  # 이미지 객체 직접 전달
```

### 3. 동일 페이지의 여러 절(節) 시작 처리
- TOC상 `I.1`, `I.2`, `I.3`, `I.4` 가 모두 doc-p1에 있는 경우 있음
- 단순 `str.replace` 루프로 앵커 삽입 시 **역순**으로 쌓임
- 해결: 페이지별 섹션 리스트를 미리 그룹핑 → 한 번의 replace로 전체 블록 삽입

### 4. TOC의 OCR 오탈자
- `협상` → `현상` , `적격자` → `적겨자` , `5m` → `5r` 등
- TOC 항목은 index.md의 네비게이션이므로 **수동 교정** 필요
- 본문은 `원문 그대로` 원칙을 지켜 수정하지 않음

## 산출물 구조
```
00_제안요청서/
├── index.md                   # Agent용 네비 (장/절 링크 + 키워드 역인덱스 + 페이지 역인덱스)
├── 01_사업개요.md             # I장
├── 02_현황및문제점.md          # II장
├── 03_사업추진방안.md          # III장
├── 04_제안요청내용.md          # IV장 (130KB, 62p)
├── 05_제안관련사항.md          # V장
├── 06_붙임자료및별지서식.md    # VI장 (76KB, 46p)
├── _raw/
│   ├── pages/p001.png ~ p133.png   # 렌더 PNG (재활용 가능)
│   ├── ocr/p001.json ~ p133.json   # OCR 원시 결과 (bbox+conf)
│   └── text/p001.txt ~ p133.txt    # 리플로우된 페이지 텍스트
└── _scripts/
    ├── 01_render_pdf.py
    ├── 02_run_ocr.py
    ├── 03_reflow.py
    └── 04_build_md.py
```

## 각 md 파일 내 앵커 규칙
- `<!-- pdf-page: NNN / doc-page: N -->` — 페이지 구분 주석
- `<a id="pdf-pNNN">` + `<a id="doc-pN">` — 양쪽 페이지 체계 앵커
- `<a id="{roman}-{num}-{slug}">` + `## {roman}.{num} {title}` — 절 헤딩 앵커
- Index.md에서 `#pdf-p020` 같은 하위 앵커로 특정 페이지에 바로 링크 가능

## 검증 스크립트 (필수)
```python
# 1. 장별 페이지 마커 수 == 할당된 페이지 수
# 2. 장별 섹션 헤딩 수 == TOC 섹션 수
# 3. 3개 페이지 랜덤 스팟체크 (텍스트 잘 들어갔는지)
```

## RAG 활용 시 권장 설정
- **청크 경계**: `<!-- pdf-page: ... -->` 주석 기준으로 자르면 페이지별 청킹
- **메타데이터**: 파일명(장) + 페이지 주석 → chunk metadata로 전달
- **제목 보강**: 각 청크 앞에 소속 장/절 제목을 prefix로 붙이면 검색 정확도 상승

## 품질 한계
- OCR 정확도 ~85~95% (표, 도식 부분은 특히 낮음)
- 긴 도식 페이지(시스템 구성도 등)는 재구성이 거의 불가 — 원본 PDF 참조 유도 필요
- 숫자/영문 혼합(금액, 코드, FP-MM 등)은 상대적으로 정확
