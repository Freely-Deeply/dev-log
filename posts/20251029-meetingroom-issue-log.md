# 🧩 회의실 예약 페이지 — 개발 이슈 & 개선 기록

## 💡 들어가며

지난 포스트에서는 **회의실 예약 페이지의 전체 구조**를 설계했다.  
이번에는 실제 API 연동, 모달 동작, UI 개선 과정에서 발생한 **이슈와 해결 과정**을 정리했다.  
단순히 동작하는 화면이 아니라, **사용성과 일관성**을 높이는 실무 디테일을 다뤘다.

---

## ⚙️ API 연동 구조 검증

회의실 예약 기능은 `setMeetRoomApi.js`를 통해 AxiosInstance와 통신한다.  
초기엔 문서 상 구조대로 구현했으나 실제 서버는 일부 다르게 동작하여 **Postman으로 직접 검증**했다.

| 동작      | 문서 상 구조  | 실제 서버 구조   | 수정 내용                |
| --------- | ------------- | ---------------- | ------------------------ |
| 예약 등록 | PUT (query)   | PUT (body JSON)  | JSON 직렬화 후 body 전송 |
| 예약 수정 | POST (query)  | POST (body JSON) | axios.post로 수정        |
| 예약 삭제 | DELETE (body) | DELETE (params)  | seq를 params로 전달      |

> ✅ 서버 동작 방식과 일치하도록 수정 후, 모든 CRUD 요청에서 **200 OK** 정상 응답 확인.

---

## 🧠 주요 이슈 & 해결 과정

| 구분               | 문제                     | 원인                                 | 해결                                |
| ------------------ | ------------------------ | ------------------------------------ | ----------------------------------- |
| 수정 요청 403      | query param 호출         | Spring Controller가 JSON body만 허용 | body(JSON) 방식으로 변경            |
| 예약자 미반영      | reg_emp_id 수정 제한     | DB 정책상 수정 불가                  | select 비활성화 처리                |
| 수정 후 미갱신     | onSaved() 콜백 누락      | 상태 업데이트 누락                   | onSaved() → loadReservations() 통일 |
| select 화살표 표시 | disabled 상태에서도 표시 | 브라우저 기본 스타일                 | CSS `appearance:none` 적용          |
| 버튼 정렬 불균형   | 삭제 버튼만 세로 배치    | flex 미정의                          | 수정·삭제·닫기 버튼 한 줄 정렬      |

---

## 🎨 UI / UX 개선

- 수정 모드: 예약자 선택 불가 + select 화살표 제거
- 등록 모드: 예약자 선택 가능
- 모달 하단 버튼 3개 (수정 | 삭제 | 닫기) 균등 정렬
- 시간 검증: 시작시간 ≥ 종료시간 시 에러 표시

---

## 🧩 코드 구조 리팩토링

모달 내부의 등록/수정/삭제 로직을 하나의 `handleSubmit(mode)` 함수로 통합하여  
중복된 상태 및 API 호출 분기를 제거했다.

```js
const handleSubmit = async (mode) => {
  const apiFn = mode === "edit" ? updateReservation : saveReservation;
  await apiFn(formData);
  onSaved();
  onClose();
};
```
