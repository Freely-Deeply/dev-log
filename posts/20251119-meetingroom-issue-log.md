# 📘 2025-11-19 — 결재/공통코드 리스트 검색 필터 개선 (statusFilter 기본값 Y)

## 🎯 작업 목표

- `statusFilter` 검색 필터 추가
- 초기 진입 시 `"Y"`(사용중)만 보이도록 기본값 설정
- 검색 버튼 클릭 시에만 API 호출
- 백엔드 필터 + 프론트 상태 관리를 조합해 안정적인 검색 흐름 구성

---

## ✅ 오늘 한 작업

### 1) 상태 선언 – statusFilter
```jsx
const [statusFilter, setStatusFilter] = useState("Y");
```

- 컴포넌트 진입 시 기본값 `"Y"` → 사용중 리스트만 먼저 노출
- `statusFilter`: 현재 필터 값
- `setStatusFilter`: 필터 값을 변경하는 함수
- `useState(초기값)`으로 React 상태 생성

---

### 2) 검색 버튼 핸들러 – onSearchClick
```jsx
const onSearchClick = () => {
  handleSearch();
};
```

- 검색 버튼을 눌렀을 때만 실제 검색 수행
- 셀렉트 변경(onChange)와 검색 실행을 분리 → 불필요한 API 호출 방지

---

### 3) 검색 함수 – handleSearch
```jsx
const handleSearch = useCallback(async () => {
  setLoading(true);
  try {
    const searchParams = {};
    if (statusFilter) {
      searchParams.use_yn = statusFilter;
    }
    const res = await getApprovalList(searchParams);
    setRows(fixedItems);
  } finally {
    setLoading(false);
  }
}, [statusFilter]);
```

**핵심 해석:**

- `statusFilter`가 있으면 `searchParams.use_yn`에 담아서 API 호출
- 응답 결과를 가공해 `setRows(fixedItems)`로 테이블 데이터 업데이트
- `setLoading(true) ~ finally { setLoading(false) }`로 로딩 상태를 안전하게 관리
- 의존성 배열 `[statusFilter]` → 필터 값이 바뀌면 최신 값 기준으로 함수가 갱신됨

---

### 4) 초기 실행 – useEffect
```jsx
useEffect(() => {
  handleSearch();
}, [refreshTrigger]);
```

- `refreshTrigger` 값이 바뀔 때마다 `handleSearch` 실행
- 초기 렌더링 + 강제 새로고침 트리거를 한 곳에서 관리

---

### 5) UI – 사용 여부 셀렉트 박스
```jsx
<select
  value={statusFilter}
  onChange={(e) => setStatusFilter(e.target.value)}
>
  <option value="">전체</option>
  <option value="Y">사용중</option>
  <option value="N">미사용</option>
</select>
```

- `value={statusFilter}` → controlled component 패턴
- 화면에 보이는 값과 내부 상태가 항상 일치
- `onChange`에서 `e.target.value`를 읽어 상태 업데이트
- `value=""`는 "전체 조회"용 필터 초기화에 사용

---

## 🧠 오늘 느낀 점 / 인사이트

- 필터 하나를 추가해도 State → UI → API 파라미터 흐름을 한 번에 설계해야 나중에 안 꼬인다.
- 검색 버튼과 `onChange`를 분리해두면 "값만 바꾸고, 언제 검색할지는 사용자가 선택"하는 구조가 된다.
- `finally` 블록에서 로딩 상태를 확실히 내려줘야 에러 상황에서도 스피너가 멈추지 않는 문제를 예방할 수 있다.
- 셀렉트 박스를 상태로 제어하면, 디버깅할 때 "지금 실제 필터 값이 무엇인지"를 바로 추적할 수 있다.

---

## 📝 한 줄 요약

**"검색 필터는 단순한 UI 옵션이 아니라, 상태와 API 흐름을 동시에 제어하는 핵심 데이터다."**
