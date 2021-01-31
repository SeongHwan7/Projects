# Na-Lab Project



## 일정관리 달력



* 목차
  1. [달력 ui 수정](#ui)
  2. [일정관리](#mana)
     1. [기능보완](#up)
     2. [과제게시판연동](#assignment)

-----------------------

</br>

<div id="ui">●  달력 ui 수정</div>

NaLab의 메인페이지를 조금씩 수정하는 과정에서 달력의 ui도 함께 변경하였습니다. 기존에 아래쪽에 위치했던 일정 테이블을 달력의 오른쪽에 배치하여 페이지가 한눈에 보이도록 가시성을 높혔습니다. 

![image](https://user-images.githubusercontent.com/78251137/106388840-a6ccba80-6423-11eb-9107-c3420e15af3f.png)

</br>

Lab에는 두가지의 일정이 존재합니다. 하나는 Lab 전체관리자가 작성하는 전체일정과 다른 하나는 각 Lab에 소속된 교수가 작성할 수 있는 Lab 일정으로 구분되어있습니다. 이 두가지의 일정을 구분하기위하여 sql문을 수정하였습니다.

```sql
<select id="Cal_ajax_lab" resultType="hashmap">
        <![CDATA[
            SELECT
                contents as title, start, end,'#ffff00' as color, '#ffffff' as textColor, '3' as id
            FROM
                calendar
            WHERE
                lab_code = #{lab_code}
            AND
                division = 0
            ORDER BY
                start
		]]>
    </select>

    <select id="Cal_ajax_lab_todo" resultType="hashmap">
        <![CDATA[
            SELECT
                contents as title, start, end,'#0000ff' as color, '#ffffff' as textColor, '3' as id
            FROM
                calendar
            WHERE
                lab_code = #{lab_code}
            AND
                division = 1
            ORDER BY
                start
		]]>
    </select>
```

</br>

sql문에 color라는 값을 임의로 추가해주었습니다. 이 color값은 javascript에서 정의해주는게 맞지만 Fullcalendar 라이브러리의 구버전에서는 두개의 JSON데이터를 가져와서 사용할 때 각각 색상을 설정해주는게 불가능하여 sql에 색상을 구분하는 color값을 임의로 추가해준뒤 JSON형태로 불러와 달력에 출력해주도록 설정해두었습니다. 수정된 sql의 대한 결과화면은 이러합니다.

![image](https://user-images.githubusercontent.com/78251137/106389692-04fb9c80-6428-11eb-8949-fffcdacaec2b.png)

</br>



<div id="mana">●  일정관리</div>

직접 일정관리기능을 이용하면서 느껴진 불편한 부분을 수정하고 이 기능은 꼭 있었으면 좋겠다고 생각했던 기능을 추가하였습니다.

</br>

<div id="up">1.기능보완</div>

먼저 불편했던 부분에 대한 보완내용입니다. 특정 날짜에 일정이 몰렸을때 달력의 ui가 뭉게지는현상이 발생했고 일정 내용이 길어지면 글씨가 잘려서 보이지않는 불편함이 있었습니다. 이를 특정 기능을 추가하여 보완하였습니다.

 ![image](https://user-images.githubusercontent.com/78251137/106389882-2315cc80-6429-11eb-9ec1-dfafc22e5378.png)

</br>

19일자를 보면 `+3 more` 라고 써진부분을 볼 수 있습니다. 이는 해당날짜에 3개의 일정이 더 있다는걸 의미하고 해당 글씨를 클릭시 어떤 일정인지 볼 수 있습니다. 코드는 아래와 같습니다.

```javascript
var calendar = new FullCalendar.Calendar(calendarEl, {
    headerToolbar: {
        left: 'prevYear,prev',
        center: 'title',
        right: 'next,nextYear'
    },
    dayHeaderFormat: function (date) {
        let weekList = ["", "", "", "", "일", "월", "화", "수", "목", "금", "토"];
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
    eventClick: function (info) {
        var eventObj = info.event;
        var u_info = '${user_info.auth}';
        if (eventObj.url) {
        } else {
            if(eventObj.id==4) {
                $("#detail").modal();
            }else {
                $("#detail_2").modal();
            }
        },
    events: events,
	displayEventTime: false
});
```

</br>

`views`에서 `timeGrid`의 `dayMaxEventRows`값을 지정해줌으로써 한 날짜에 최대 몇개까지의 일정이 표시될 수 있는지 제한해주는 역할을 합니다.  그리고 글씨가 길어서 잘리는부분은 `eventClick`에서 `eventObj`안에 포함된 값들을 모달창을 통하여 보여주는 방식을 사용하였습니다. 

 ![image](https://user-images.githubusercontent.com/78251137/106389885-2610bd00-6429-11eb-9f9e-9b18f99433d8.png)

</br>

<div id="assigment">2.과제게시판연동</div>

이번엔 새로운 기능 추가에 대한 내용입니다. Lab게시판에 제가 기존에 만들어두었던 과제 게시판이 존재하는데 여기에 과제를 등록할경우 자동으로 달력에 과제 일정이 추가되면 어떨까 라고 생각하여 개발하게된 기능입니다.

```sql
<select id="Cal_ajax_report" resultType="hashmap">
        <![CDATA[
            SELECT distinct
            a.title, a.start, a.end, '#ff0000' as color, '#ffffff' as textColor,b.board_num) as url
            FROM
            assignment_info a, board_list b
            WHERE
            a.lab_code = #{lab_code}
            AND
            del_flag = 0
            AND
            a.board_code = b.board_code
            ORDER BY
            start
        ]]>
</select>
```

과제게시판의 과제글목록을 조인하여 `start`(과제시작일), `end`(과제종료일), `title`(과제명)을 받아와 JSON데이터형태로 넘겨주고 달력에 출력해줍니다. 각각의 Lab교수님마다 등록하는 과제가 다르기때문에 이 부분은 로그인했을 때 자신의 `lab_code`와 과제게시판에 등록된 글들의 `lab_code`를 비교하여 자신이 소속된 랩의 과제만 보이도록 설정되어있습니다.

![image-20210201023435030](C:\Users\PC\AppData\Roaming\Typora\typora-user-images\image-20210201023435030.png)

이제 과제를 등록할 경우 달력에도 자동으로 동기화되는걸 확인할 수 있습니다. 

