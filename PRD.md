# PRD — 뱅크법률집사 (역할 선택 포털)

## 1. Product Overview

| 항목 | 내용 |
|------|------|
| **서비스명** | 뱅크법률집사 (BankCasa) |
| **한 줄 설명** | 신협·단위농협·새마을금고 대상 AI 금융 업무 자동화 플랫폼의 진입점. 로그인 후 행원/조합원/이사장 역할을 선택하면 각 역할 전용 앱(별도 URL)으로 이동한다. |
| **대상 사용자** | 신협·단위농협·새마을금고 행원, 조합원, 이사장 |

## 2. Tech Stack

| 구분 | 기술 |
|------|------|
| 프레임워크 | React 19 + TypeScript |
| 빌드 도구 | Vite 8 |
| 스타일링 | Tailwind CSS |
| 라우팅 | HashRouter (react-router-dom) |
| 상태 관리 | Context API (AppProvider) |
| 저장소 | localStorage |
| 배포 | GitHub Pages (gh-pages 브랜치) |

## 3. Architecture

```
SPA (Single Page Application)
├── HashRouter
├── AppProvider (Context API)
│   ├── user 상태 (기관, 이름, 역할)
│   ├── case 상태 (현재 상담 케이스)
│   └── form 상태 (서류 작성 폼 데이터)
├── Pages/
│   ├── LoginPage
│   ├── RolePage
│   ├── Dashboard (역할별 분기)
│   │   ├── TellerDashboard
│   │   ├── MemberDashboard
│   │   └── ChairmanDashboard
│   ├── ConsultPage (+ 하위 단계)
│   ├── DocumentsCatalogPage
│   ├── CalculatorPage
│   ├── RegulationSearchPage
│   └── ...기타 페이지
└── Components/
    ├── AppLayout (사이드바 + 메인)
    ├── ChatInterface
    └── FormComponents
```

## 4. Pages & Routes

| 경로 | 페이지 | 기능 설명 |
|------|--------|-----------|
| `/login` | LoginPage | 로그인 — 기관유형 선택 + 기관명 + 담당자 입력. 좌측: WebGL 3D 구체 애니메이션 + 카피("금융기관의 업무 효율을 AI로 혁신합니다", 변호사 광고규정 준수). 우측: 로그인 폼. |
| `/role` | RolePage | 역할 선택 — 행원/조합원/이사장 3카드. 각 카드 클릭 시 `window.open`으로 해당 역할 전용 URL 새 탭 오픈. URL 파라미터: `#/?role=xxx&inst=xxx&name=xxx` |
| `/` | Dashboard | 대시보드 — role에 따라 분기: TellerDashboard / MemberDashboard / ChairmanDashboard |
| `/consult` | ConsultPage | AI 법률 상담 — 채팅 모드/버튼 모드 토글 지원 |
| `/consult/documents` | DocumentsPage | 서류 안내 — 해당 케이스에 필요한 서류 목록 |
| `/consult/form` | FormPage | 정보 입력 — 서류 생성을 위한 상세 정보 폼 |
| `/consult/generating` | GeneratingPage | 서류 생성 중 — 2.8초 애니메이션 |
| `/consult/preview` | PreviewPage | 서류 미리보기 — 생성된 서류 확인 |
| `/consult/complete` | CompletePage | 서류 생성 완료 |
| `/documents-catalog` | DocumentsCatalogPage | 서류 자동화 카탈로그 — 8종 카드 그리드 |
| `/calculator` | CalculatorPage | 채권계산기 — 원금+이자+지연손해금+인지대·송달료 |
| `/regulation-search` | RegulationSearchPage | AI 법령·지침 검색 |
| `/finance` | FinancePage | 금융 안내 |
| `/community` | CommunityPage | 커뮤니티 (조합원 전용) |
| `/member-portal` | MemberPortalPage | 조합원 법률 현황 조회 |
| `/report` | ReportPage | 이사장 법적 조치 리포트 |
| `/board` | BoardPage | 이사장 조합원 게시판 |
| `/notice` | NoticePage | 이사장 공지 발송 |
| `/history` | HistoryPage | 상담 내역 |
| `/mypage` | MyPage | 마이페이지 |

## 5. Data Models

```typescript
// 사용자
interface User {
  institution: string;   // 기관명
  name: string;          // 담당자 이름
  role?: 'teller' | 'member' | 'chairman';
}

// 상담 내역
interface ConsultHistory {
  id: string;
  date: string;
  caseType: string;      // 케이스 유형 (예: 'auction', 'garnishment')
  caseName: string;      // 케이스 표시명
  status: string;        // 상태 (진행중, 완료 등)
  formData?: Record<string, any>;
  claimCalc?: ClaimCalculation;
}

// 케이스 정의
interface CaseData {
  id: string;
  name: string;
  description: string;
  documents: string[];           // 필요 서류 목록
  formFields: FormField[];       // 입력 필드 정의
  procedureSteps: string[];      // 절차 단계
  documentTemplate: string;      // 서류 템플릿 (문자열 치환용)
}

// 의사결정 트리
type DecisionTreeNode =
  | { result: string }
  | { question: string; options: Record<string, DecisionTreeNode> };
```

## 6. Key Features

### 6.1 AI 법률 상담
- 8개 케이스에 대한 의사결정 트리 기반 상담
- 자유 텍스트 입력 시 키워드 매칭으로 케이스 자동 분류
- 채팅 모드 / 버튼 모드 토글 지원

### 6.2 서류 자동화 (8종)
1. 임의경매 신청서
2. 가압류 신청서
3. 지급명령 신청서
4. 추심명령 신청서
5. 내용증명
6. 강제경매 신청서
7. 이의신청서
8. 출자금반환 청구서

### 6.3 채권계산기
- **원금** + **이자** (약정이율) + **지연손해금** (법정이율)
- **인지대** + **송달료** 자동 계산
- 법률 근거: 소송촉진법 §3① 연 12%, 민법 §379 연 5%

### 6.4 역할별 대시보드
- **행원**: 서류 작성, 채권계산, 법령 검색 중심
- **조합원**: 법률 현황 조회, 서류 안내, 커뮤니티
- **이사장**: 경영 대시보드, 연체/법적조치 현황, 게시판/공지

## 7. Design System

| 속성 | 값 |
|------|-----|
| 배경색 | `#ffffff` |
| 텍스트색 | `#000000` |
| 구분선 | `1px solid #e5e5e5` |
| 섹션 구분 | `"A /" "B /"` 슬래시 레이블 |
| 버튼 | `border-radius: 2px`, 검정/흰색만 사용 |
| 폰트 | Pretendard |
| 디자인 참조 | VW (v-w.co.kr) 스타일 — 미니멀, 흑백 기반 |

## 8. External Integrations

| 서비스 | URL | 연동 방식 |
|--------|-----|-----------|
| 집변 (법률 상담) | `homelawyer.kr` | `window.open()` 외부 링크 |
| 법령정보 | `law.go.kr` | 외부 링크 |

### Local Storage Keys
| 키 | 용도 |
|----|------|
| `blj_user` | 로그인 사용자 정보 |
| `blj_history` | 상담 내역 |
| `blj_current_case` | 현재 진행 중인 케이스 |
| `blj_current_form` | 현재 폼 데이터 |

## 9. Deployment

| 항목 | 값 |
|------|-----|
| 호스팅 | GitHub Pages |
| 브랜치 | `gh-pages` |
| URL | `https://{owner}.github.io/bankcasa-demo/` |
| 빌드 | `vite build` → `dist/` |

## 10. Known Limitations

- **모든 데이터는 목업** — 실제 API 연동 없음, localStorage 기반
- **AI 상담은 정적 의사결정 트리** — LLM 미연동, 하드코딩된 분기
- **채권계산은 클라이언트 사이드** — 서버 검증 없음
- **서류 템플릿은 하드코딩된 문자열 치환** — 동적 생성 아님
- **인증/인가 없음** — 로그인은 폼 입력만, 실제 인증 미구현
- **반응형 제한** — 역할별로 데스크탑/모바일 최적화 수준 상이
