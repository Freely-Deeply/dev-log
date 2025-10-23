reservation.md)

# 🏢 회의실 예약 페이지 구조 설계 (SetMeetRoom.jsx)

> React 프로젝트 중 회의실 예약 기능을 설계하며  
> 컴포넌트 구조화, 상태 관리, 검증 로직 분리를 중심으로 기록했다.

---

## 💡 들어가며

이번에는 실제 사용자 시나리오 기반 기능인 **회의실 예약 페이지**를 구현했다.  
이 페이지는 단순 조회가 아니라,  
**캘린더 기반 일정 표시 + 예약 등록/수정/삭제 + 권한 체크**까지 포함된  
실제 서비스 수준의 기능이었다.

---

## 🧭 기능 개요

| 구분          | 내용                                               |
| ------------- | -------------------------------------------------- |
| **기능명**    | 회의실 예약 (SetMeetRoom.jsx)                      |
| **기능 목적** | 회의실별 예약 일정 조회, 등록, 수정, 삭제          |
| **사용 기술** | React (Vite), FullCalendar, axios, Spring REST API |
| **구조**      | 캘린더 컴포넌트 + 예약 모달 + 검증 훅 분리         |

---

## ⚙️ 화면 구조도

```mermaid
graph TD
  A[SetMeetRoom.jsx] --> B[Calendar.jsx]
  A --> C[ReservationModal.jsx]
  A --> D[useReservationFormValidation.js]
  B --> E[getMeetRoomInfo() in meetroomApi.js]
  C --> F[saveReservation() in meetroomApi.js]
  F --> G[(axiosInstance.js)]
  G --> H[(Spring API /meetroom)]


상위 컴포넌트(SetMeetRoom.jsx)에서 선택된 날짜와 회의실 정보를
하위 컴포넌트(Calendar, Modal)에 props로 전달하고,
모든 API 통신은 axiosInstance.js를 통해 백엔드(Spring)과 연결된다.

📁 핵심 폴더 구조
📦 src
 ┣ 📂components/meetroom
 ┃ ┣ 📜 Calendar.jsx
 ┃ ┣ 📜 ReservationModal.jsx
 ┃ ┗ 📜 useReservationFormValidation.js
 ┣ 📂assets/services
 ┃ ┗ 📜 meetroomApi.js
 ┗ 📜 SetMeetRoom.jsx

🧠 핵심 로직 요약
① SetMeetRoom.jsx

선택된 날짜/회의실 정보를 하위 컴포넌트로 전달

const [selectedRoom, setSelectedRoom] = useState(null);
const [selectedDate, setSelectedDate] = useState(new Date());

<Calendar onSelectDate={setSelectedDate} onSelectRoom={setSelectedRoom} />
<ReservationModal selectedRoom={selectedRoom} selectedDate={selectedDate} />

② useEffect로 초기 데이터 로드
useEffect(() => {
  fetchMeetroomInfo();
}, []);

const fetchMeetroomInfo = async () => {
  const res = await getMeetroomInfo();
  setMeetroomList(res.data.list);
};

③ 예약 등록/수정 모달
const handleSubmit = async () => {
  await saveReservation(formData);
  alert("예약이 완료되었습니다.");
  refreshCalendar();
  onClose();
};

④ useReservationFormValidation.js

입력값 검증 로직을 훅으로 분리

export const useReservationFormValidation = () => {
  const validate = (data) => {
    if (!data.room_id) return "회의실을 선택하세요.";
    if (!data.start_time || !data.end_time) return "시간을 입력하세요.";
    if (data.start_time >= data.end_time) return "시간 범위가 잘못되었습니다.";
    return null;
  };
  return { validate };
};

🧩 개발 중 배운 점

✅ 상태(state)는 페이지의 단일 진실(Single Source of Truth)
→ 상위 컴포넌트에서 관리하고 하위로 전달해야 데이터 일관성 유지됨.

✅ UI 중심의 캘린더 / 로직 중심의 모달 분리
→ 역할이 명확해지면 유지보수가 쉬워진다.

✅ 폼 검증 로직 훅 분리의 장점
→ 재사용 가능하고 테스트가 쉬워진다.

🎯 다음 개선 목표

 예약 중복 체크 기능 추가

 관리자 / 일반 사용자 권한 구분

 캘린더 주/월 단위 전환 UI 개선

 예약 수정 이력 관리

✨ 마무리

이번 작업은 단순한 “예약 기능 구현”이 아니라,
데이터 흐름이 살아 있는 사용자 시나리오 설계였다.

상태, 이벤트, 검증 로직을 각각의 역할에 따라 분리하면서
React 구조 설계의 기본기를 더 탄탄히 다질 수 있었다.
```
