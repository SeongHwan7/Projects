# Na-Lab Project



## 일정관리 달력



* 목차
  1. [Calendar 초안](#cal)
  2. [일정관리](#mana)

-----------------------

</br>

<div id="cal">●  Calendar 초안</div></br>

NaLab 메인페이지 아래쪽에 Lab의 공통일정이나 각각의 Lab일정을 보기쉽게 정리해보고자 달력을 이용하여 일정관리기능을 개발하였습니다.



달력은 Fullcalendar 라이브러리를 사용했고 달력에 데이터베이스를 연동시켜 데이터를 수정할 수 있도록 구현했습니다.  아래 그림은 가장처음 Fullcalendar를 적용시키고 데이터베이스와 연동하여 달력에 일정을 추가해주도록 설정한 화면입니다. 

 ![image](https://user-images.githubusercontent.com/78251137/106386801-163dac80-641a-11eb-904c-9fd344741658.png)

</br>



<div id="mana">●  일정관리</div></br>

Calendar의 오른쪽위 `일정관리` 버튼을 클릭하면 아래의 모달로 이동하게됩니다.

![image](https://user-images.githubusercontent.com/78251137/106386858-5e5ccf00-641a-11eb-8013-16ea8b4ad71e.png)

</br>

나타나는 모달은 시작날짜와 끝나는날짜를 지정하고 일정에 대한 내용을 입력하여 등록버튼을 누르게되면 데이터베이스로 데이터가 전송되어 달력에 출력되는 기능을 가지고있습니다.

```jsp
<p>
    <i class="icon-append fa fa-calendar" style="padding-left: 10px;"></i>
    <input type="text" name="start" id="start_${var.index}" placeholder="시작일(필수)" style="display: inline;" value="${cal.start}" class="required" >

    <select name="start_hour" id="start_hour_${var.index}" class="form-control" style="width:80px;display:inline;">
        <option value="${cal.start_hour}" selected>${cal.start_hour}시</option>
        <c:forEach begin="0" end="23" var="hour">
            <option value="${ hour < 10 ? '0' : '' }${hour}">${hour}시</option>
        </c:forEach>
    </select>

    <select name="start_minute" id="start_minute_${var.index}" class="form-control" style="width:80px;display:inline;">
        <option value="${cal.start_minute}" selected>${cal.start_minute}분</option>
        <c:forEach begin="0" end="59" var="minute">
            <option value="${ minute < 10 ? '0' : '' }${minute}">${minute}분</option>
        </c:forEach>
    </select>

    <label for="content" style="padding-left: 10px; font-weight:bold;">~</label>

    <i class="icon-append fa fa-calendar" style="padding-left: 10px;"></i>
    <input type="text" name="end" id="end_${var.index}" placeholder="종료일(필수)" style="display: inline;" value="${cal.end}" class="required_1" >

    <select name="end_hour" id="end_hour_${var.index}" class="form-control" style="width:80px;display:inline;">
        <option value="${cal.end_hour}" selected>${cal.end_hour}시</option>
        <c:forEach begin="0" end="23" var="hour">
            <option value="${ hour < 10 ? '0' : '' }${hour}">${hour}시</option>
        </c:forEach>
    </select>

    <select name="end_minute" id="end_minute_${var.index}" class="form-control" style="width:80px;display:inline;">
        <option value="${cal.end_minute}" selected>${cal.end_minute}분</option>
        <c:forEach begin="0" end="59" var="minute">
            <option value="${ minute < 10 ? '0' : '' }${minute}">${minute}분</option>
        </c:forEach>
    </select>

    <input type="text" class="form-control required_2" name="contents" id="con" value="${cal.contents}" style="width:400px;display:inline;" placeholder="일정 내용을 입력하세요" >
    <a href="#this" class="btn" id="delete_${var.index}" name="delete_${var.index}">삭제</a>
</p>
```

</br>

여기서 p태그로 감싸준 이유는 javascript를 통하여 일정추가버튼을 클릭했을 때 p태그로 감싸진 코드영역이 새롭게 추가될 수 있도록 구현하기 위하여 사용하였습니다. 동일한 id가 생성될경우 에러가 발생하기때문에 `index`값을 이용하여 id뒤에 숫자로 구분할 수 있도록 설정해두었습니다.

</br>

달력에 표시해줄 데이터베이스는 다음 코드를 통해 받아오고 출력해주게됩니다.

```javascript
$.ajax({
    type: 'POST',
    async:false,
    url: '/nalab/calendar_ajax.do',
    dataType: 'json',
    success: function (data) {
        console.log("success : " + JSON.stringify(data.cal_ajax));
        events = data.cal_ajax;
    },
    error: function (request, status, error) {
        console.log("error");
    }
});

var calendar = new FullCalendar.Calendar(calendarEl, {
    headerToolbar: {
        left: 'prevYear,prev',
        center: 'title',
        right: 'next,nextYear'
    },
    dayHeaderFormat: function (date) {
        let weekList = ["일", "월", "화", "수", "목", "금", "토"];
        return weekList[date.date.day];
    },

    titleFormat: function (date) {
        return date.date.year + "년 " + (date.date.month + 1) + "월";
    },
    dayMaxEventRows: true,
    views: {
        timeGrid: {
            dayMaxEventRows: 3
        }
    },
    events: {
        events
    },       
    displayEventTime: false
});
```

</br>

먼저 ajax를 이용하여 url에 Calendar 테이블의 데이터를 select해오는 쿼리를 요청합니다. 받아온 데이터는 JSON형태로 `data.cal_ajax`에 담겨져있으며 이를 `events`라는 변수에 넣어줍니다.

`events` 변수는 `FullCalendar.Calendar`함수내부의 events 항목에 넣음으로써 달력에 데이터를 출력해줄 수 있습니다.

