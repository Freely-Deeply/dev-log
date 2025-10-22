비전공자 주니어의 첫 실무형 페이지 리팩토링 기록

💡 들어가며
비전공자로 개발을 시작한 지 몇 개월.
이번에는 “공통코드 관리 페이지”라는 실무 기능을 직접 설계하고 구현했다.

단순히 화면을 만드는 게 아니라,
데이터 흐름 + 컴포넌트 구조 + API 연동을 처음부터 끝까지 이해하면서 만든 첫 페이지다.

🧭 기능 개요
구분내용
기능명 공통코드 관리 (SetCode)
기능 목적 그룹코드(대분류)와 공통코드(상세코드)를 등록/수정/조회
기술 스택 React (Vite), axios, Spring API
구조 리스트 컴포넌트 + 모달 컴포넌트 분리
⚙️ 화면 구조도
graph TD
A[SetCode.jsx] --> B[SetCodeGroupList.jsx]
A --> C[SetCodeCommonList.jsx]
B --> D[SetCodeGroupModal.jsx]
C --> E[SetCodeCommonModal.jsx]
B --> F[setCodeApi.js]
C --> F
F --> G[(axiosInstance.js)]
G --> H[(Spring API /config)]

📁 핵심 폴더 구조
📦 src
┣ 📂components/setCode
┃ ┣ 📜 SetCodeGroupList.jsx
┃ ┣ 📜 SetCodeGroupModal.jsx
┃ ┣ 📜 SetCodeCommonList.jsx
┃ ┗ 📜 SetCodeCommonModal.jsx
┣ 📂assets/services
┃ ┗ 📜 setCodeApi.js
┣ 📂utils
┃ ┗ 📜 axiosInstance.js
┗ 📜 SetCode.jsx

🧠 핵심 로직 요약
① SetCode.jsx
상위 컨테이너: 그룹코드 선택 → 공통코드로 전달

const [selectedGroupCode, setSelectedGroupCode] = useState(null);

<SetCodeGroupList onGroupSelect={setSelectedGroupCode} />
<SetCodeCommonList groupCode={selectedGroupCode} />
② SetCodeGroupList.jsx
그룹코드 리스트 조회 및 등록 모달 관리

const fetchGroupCodeList = async () => {
const res = await getGroupCodeList();
setGroupCodeList(res.data.list);
if (list.length > 0) onGroupSelect(list[0].group_code);
};
③ SetCodeGroupModal.jsx
외부 클릭 시 자동 닫히는 모달 구현

const modalRef = useRef(null);
useEffect(() => {
const handleClickOutside = (e) => {
if (modalRef.current && !modalRef.current.contains(e.target)) onClose();
};
document.addEventListener('mousedown', handleClickOutside);
return () => document.removeEventListener('mousedown', handleClickOutside);
}, [onClose]);
④ setCodeApi.js
axiosInstance를 활용한 CRUD 분기

export const saveGroupCode = async (data, mode) => {
return mode === 'register'
? axiosInstance.put('/config/comm_group_code', data)
: axiosInstance.post('/config/comm_group_code', data);
};
⑤ axiosInstance.js
인터셉터를 통한 공통 설정 관리

axiosInstance.interceptors.request.use((config) => {
const token = localStorage.getItem('accessToken');
if (token) config.headers['Authorization'] = token;
return config;
});
🧩 개발 중 배운 점
✅ 컴포넌트는 UI 단위가 아니라 ‘데이터 단위’로 나눠야 한다.
→ state가 어디서 만들어지고 props로 어디까지 전달되는지가 핵심

✅ 모달은 단순한 팝업이 아니라, 상태를 제어하는 독립 단위다.
→ 닫힘/등록 후 리스트 새로고침까지 연결해야 완성

✅ axiosInstance는 ‘한 줄짜리 공통 설정’이지만 유지보수 효율은 크다.

🎯 다음 개선 목표
삭제 기능 추가
Validation (빈값 방지 / 중복 확인)
선택된 그룹 강조 UI 추가
공통 모달 컴포넌트로 리팩토링
✨ 마무리
오늘은 단순히 화면을 만든 게 아니라
“데이터의 흐름과 상태의 생명주기”를 이해했다.
앞으로 모든 화면 개발의 기본이 될 감각을 얻은 하루였다.
