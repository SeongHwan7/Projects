# Na-Lab Project



## 일정관리 달력



* 목차
  1. [할 일 추가](#todo)
  2. [편의성 개선](#up)
     1. [기능보완](#up)
     2. [과제게시판연동](#assignment)

-----------------------

</br>

교수님께서 네이버캘린더를 사용하면서 편리했던점을 말씀해주시고 관련기능의 추가를 요청하셔서 해당 기능들을 추가하고 편의성을 개선한 내용입니다. 

</br>

<div id="todo">●할 일 추가</div>

일정과는 다르게 단 하루만 할 일이 있을 때 사용할 수 있는 기능입니다. 기존의 일정은 하루만 지정해도 막대로 나타나고 할 일과 일정을 확실하게 구분짓는 명확한 경계가 없었습니다. 그래서 일정은 막대로 표시하고 할 일은 점으로 표시되어 구분하기 편리하게 만들었습니다.

![image](https://user-images.githubusercontent.com/78251137/106393138-bacee700-6438-11eb-96e8-ed45457cd162.png)



일정은 무조건 막대로, 할 일은 점으로 표시하기위해 sql을 수정해주었습니다.

```sql
<select id="Cal_ajax_lab" resultType="hashmap">
        <![CDATA[
            SELECT
            contents as title, INSERT(start,11,1,'T') as start, INSERT(end,11,1,'T') as end,'#641e64' as color, '#ffffff' as textColor, 'block' as display, '3' as id
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
```

`display`값을 `block`으로 주어서 일정은 항상 막대로 표시되도록 설정했습니다.



<div id="up">●편의성 개선</div>

할 일이나 일정을 통합적으로 관리하는것이 아닌 특정 할 일이나 일정을 클릭했을 때 해당 내용을 수정하는 기능이나 비어있는 날짜를 클릭했을 때 해당 날짜에 일정을 등록하는 기능을 구현하였습니다.

```javascript
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
        document.getElementById('contents_modal').value = eventObj.title;
        document.getElementById('start_modal').value = eventObj.startStr.substr(0, 10);
        document.getElementById('start_time').value = eventObj.startStr.substr(11, 5);
        document.getElementById('end_modal').value = eventObj.endStr.substr(0, 10);
        document.getElementById('end_time').value = eventObj.endStr.substr(11, 5);

    }
},
dateClick: function(info) {
	var u_info = '${user_info.auth}';
	if(u_info==3 | u_info==4) {
		$("#dateclick_add").modal();
		document.getElementById('Sdt').value = info.dateStr;
		document.getElementById('Sdt2').value = info.dateStr;
	}
}
```

</br>

`eventClick` : 할 일, 일정을 클릭하면 모달창을 출력하고 해당 모달창에서 수정이 가능합니다.

 ![image](https://user-images.githubusercontent.com/78251137/106394025-ab05d180-643d-11eb-881a-fd95a29b2b06.png)

</br>

`dateClick` : 비어있는 날짜를 클릭하면 모달창을 출력하고 할 일 또는 일정을 선택하여 추가할 수 있습니다. 해당 기능은 학생은 비활성화되어있으며 교수나 관리자만 접근이 가능한 기능입니다.

 ![image](https://user-images.githubusercontent.com/78251137/106394076-fcae5c00-643d-11eb-8101-7647abf91714.png)

</br>

