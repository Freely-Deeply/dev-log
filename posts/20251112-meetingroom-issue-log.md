# 📘 회의실 예약 페이지 UI/UX 개선 — 툴바 재배치 · 시간 확장 · 페이지네이션 리디자인

---

## 🎯 작업 목표

FullCalendar 기반 회의실 예약 페이지에서  
날짜 중심 툴바 재배치, 사용 가능 시간 확장(07–23시), 공통 페이지네이션 디자인 통일로  
UI 일관성과 사용 편의성을 높인다.

---

## ✅ 오늘 완료한 작업

### 1️⃣ 날짜 양옆으로 Prev / Next 버튼 배치

> 날짜(Title)를 중심으로 Prev·Next·Today를 배치하고,  
> 뷰 전환 버튼(Day/Week/Month)은 오른쪽 끝으로 유지.

```js
// Before
headerToolbar={{
  left: "prev,next today",
  center: "title",
  right: "dayGridMonth,timeGridWeek,timeGridDay",
}}

// After
headerToolbar={{
  left: "prev",
  center: "title",
  right: "next today dayGridMonth,timeGridWeek,timeGridDay",
}}
```
