# 전국 100M 격자 인구 3D 단계구분도 (2024)

전국 100m 격자 단위 인구 데이터를 3D 기둥(fill-extrusion) 방식으로 시각화하는 웹 뷰어입니다.  
**MapLibre GL JS** + **PMTiles** 기반으로 별도 서버 없이 정적 파일만으로 동작합니다.

🔗 **라이브 데모**: [thlee33.github.io/pmtiles_pop100](https://thlee33.github.io/pmtiles_pop100/)

---

## 미리보기

> 인구 1명 = 기둥 높이 1m. 최대값 슬라이더로 색상 및 높이 범위 실시간 조정.

---

## 데이터 출처

| 항목 | 내용 |
|---|---|
| 격자 폴리곤 | 통계지리정보서비스(SGIS) — 전국 100m 격자 경계 (2025년 기준, 29개 타일) |
| 인구 속성 | 통계지리정보서비스(SGIS) — 100m 격자 인구 통계 (2024년 기준) |
| 좌표계 원본 | EPSG:5179 (Korea 2000 / Unified CS) |
| 웹 서빙 좌표계 | EPSG:4326 (WGS84) |

- 속성: `grid_cd` (격자코드), `pop_total` (총인구), `pop_male` (남자), `pop_female` (여자)
- 인구 없는 격자는 `pop_total = -1` (NULL 처리), 뷰어에서 필터로 제외하여 투명하게 표시

---

## 파일 구성

```
.
├── index.html                        # 웹 뷰어 (PMTiles + GeoJSON 심리스 전환)
├── grid100m_pop1000.geojson          # 인구 ≥1,000명 격자 추출본 (소축척용, ~0.5MB)
├── build_grid100m_local.py           # 원본 데이터 → GPKG / GeoParquet / PMTiles 생성
├── build_grid100m_parquet_pmtiles.py # GPKG 완성 후 GeoParquet + PMTiles만 이어서 생성
└── extract_pop1000.py                # GeoParquet → 인구 ≥1,000명 GeoJSON 추출
```

> `grid100m_pop_2024.pmtiles` (~1.8GB) 는 Cloudflare R2에 호스팅되어 있으며 이 저장소에는 포함되지 않습니다.

---

## 시각화 방식

### 줌 레벨에 따른 심리스 데이터 전환

| 줌 레벨 | 데이터 소스 | 표시 격자 |
|---|---|---|
| < 10 (전국·광역) | `grid100m_pop1000.geojson` (로컬) | 인구 ≥ 1,000명 격자 (1,250개) |
| ≥ 10 (시·구·동 이하) | PMTiles (Cloudflare R2) | 전체 격자 |

패널 하단 배지(📦 GeoJSON / 🗂 PMTiles)로 현재 사용 중인 소스를 실시간 확인 가능합니다.

### 3D 기둥 (fill-extrusion)

- **기둥 높이**: 인구 1명 = 1m (최대값 슬라이더로 상한 조정)
- **색상 스케일**: RdYlBu 역전 (파랑 = 소, 빨강 = 다)
- **NULL 격자**: `filter`로 렌더링에서 완전 제외 (투명)

---

## 컨트롤

| 컨트롤 | 설명 |
|---|---|
| 불투명도 | 레이어 전체 투명도 조절 (기본 60%) |
| 최대값(명) | 색상 및 기둥 높이의 상한값 (기본 1,000명) |
| 기울기 | 지도 pitch 각도 (기본 55°) |
| 회전 | 지도 bearing 방향 (기본 -20°) |
| Ctrl+드래그 | 마우스로 기울기·회전 직접 조작 |

---

## 로컬 실행 방법

```bash
# 1. 이 저장소 클론
git clone https://github.com/thlee33/pmtiles_pop100.git
cd pmtiles_pop100

# 2. 로컬 HTTP 서버 실행 (Python 3)
python -m http.server 8888

# 3. 브라우저에서 열기
# http://localhost:8888/index.html
```

> PMTiles 데이터는 Cloudflare R2에서 자동으로 로드됩니다. 별도 설치 불필요.

---

## 데이터 재생성 방법

원본 SGIS 데이터에서 직접 생성하려면 아래 순서로 실행합니다.

### 사전 설치

```bash
pip install geopandas pyarrow pandas
# PMTiles 생성 시 추가 (Linux/WSL)
sudo apt-get install tippecanoe
```

### 실행 순서

```bash
# Step 1. 전국 격자 + 인구 통합 → GPKG / GeoParquet / PMTiles
python build_grid100m_local.py

# Step 2. (GPKG 완성 후 이어서) GeoParquet + PMTiles만 재생성
python build_grid100m_parquet_pmtiles.py

# Step 3. 인구 ≥1,000명 격자 GeoJSON 추출
python extract_pop1000.py
```

### PMTiles R2 업로드 (rclone)

```powershell
# rclone 설정 완료 후
rclone copy "grid100m_pop_2024.pmtiles" r2:my-map-data --progress --s3-no-check-bucket
```

---

## 기술 스택

| 라이브러리 | 버전 | 용도 |
|---|---|---|
| MapLibre GL JS | 4.7.1 | 웹 지도 GPU 렌더링 |
| pmtiles.js | 3.2.1 | PMTiles 프로토콜 처리 |
| CartoDB Positron | - | 배경지도 |
| Cloudflare R2 | - | PMTiles 정적 호스팅 |
| GeoPandas | - | 공간데이터 처리 |
| tippecanoe | - | FlatGeobuf → PMTiles 변환 |

---

## 라이선스

- **코드**: MIT License
- **데이터**: 통계청 공공데이터 (출처 표기 필요)
