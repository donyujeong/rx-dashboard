# 처방 데이터 분석 대시보드 (Rx Dashboard)

캘리포니아 지역 Medicare Part D 처방자 데이터를 약물별·전문과별·지역별로 탐색하는 웹 대시보드입니다. 별도 백엔드 서버 없이 정적 HTML + Supabase(Postgres) 조합으로 동작합니다.

**Live Demo:** https://donyujeong.github.io/rx-dashboard/dashboard.html

## 개요

- **대상 사용자**: 제약사 내부 임상·약무 분석 담당자, 전문과·지역 담당자
- **데이터**: CMS(미국 정부) Medicare Part D Prescriber 공개 데이터 중 캘리포니아 주, 약물 3종(Entresto, Cosentyx, Lucentis) — NPI 4,036명, 처방자-약물 조합 4,097건
- **핵심 기능**: 약물 / 전문과 / 도시 필터, KPI 4종(총 처방건수·총 약제비·총 환자수·처방자 수), 4개 탭(약물별 / 전문과별 / 지역별 / 처방자 상세)

## 기술 스택

| 구분 | 사용 기술 |
| --- | --- |
| 프론트엔드 | Vanilla HTML/CSS/JS 단일 파일, Chart.js 4.4.0 |
| 디자인 시스템 | Wanted Design System(Montage) 토큰 subset, Pretendard 폰트 |
| 데이터베이스 | Supabase (Postgres + REST API) |
| 배포 | GitHub Pages |

## 폴더 구조

```
├── dashboard.html   # 최종 대시보드 (Supabase 연동)
├── index.html       # 루트 접속 시 dashboard.html로 리다이렉트
├── part_d.csv       # Supabase 테이블 시딩용 원본 데이터
└── 처방데이터분석대시보드_PRD.docx  # 최초 요구사항 문서
```

## 데이터베이스 스키마

Supabase 테이블 `part_d`:

| 컬럼 | 타입 | 설명 |
| --- | --- | --- |
| drug | text | 약물명 (Entresto / Cosentyx / Lucentis) |
| specialty | text | 처방자 전문과 |
| city | text | 처방자 소재 도시 |
| tot_clms | numeric | 총 처방건수 |
| tot_drug_cst | numeric | 총 약제비(USD) |
| tot_benes | numeric (nullable) | 총 환자수 — 10명 미만은 개인정보 보호를 위해 null 처리 |
| prscrbr_npi | bigint | 처방자 NPI |

RLS(Row Level Security)는 활성화되어 있으며, 익명 읽기(SELECT) 전용 정책만 허용합니다.

```sql
alter table part_d enable row level security;
create policy "Allow public read access" on part_d for select using (true);
```

## 로컬에서 직접 연결하고 싶다면

`dashboard.html` 상단 스크립트의 아래 세 값을 본인 Supabase 프로젝트 값으로 바꾸면 됩니다.

```js
const SUPABASE_URL = 'https://<본인-프로젝트>.supabase.co';
const SUPABASE_ANON_KEY = '<본인 anon/publishable key>';
const SUPABASE_TABLE = 'part_d';
```

anon key는 클라이언트에 노출돼도 되는 공개 키이며, 실제 데이터 보호는 위 RLS 정책으로 이루어집니다. `service_role` 키는 절대 이 파일에 넣지 않습니다.

## 데이터 한계

- 연도(시계열) 축이 없어 트렌드가 아닌 동일 시점 내 비교로만 해석해야 합니다.
- 캘리포니아 단일 주 데이터로, 전국 단위 일반화는 어렵습니다.
- 10명 미만 표본은 억제(비공개 표시)되어 일부 세그먼트의 환자수는 부분적으로만 집계됩니다.
