# Na-Lab Project



## 과제게시판



* 목차
  1. [교수](#pro)
     1. [과제게시판 관리](#pro_as1)
     2. [과제 제출현황](#pro_as2)
  2. [학생](#stu)

-----------------------

</br>

<div id="pro">●  교수</div></br>

<div id="pro_as1">1. 과제게시판 관리</div></br>

교수계정으로 과제를 생성하고 세부적인 설정이 가능합니다.

 <img src="https://user-images.githubusercontent.com/78251137/106372255-4735c700-63b1-11eb-8e3b-cfcdc5a29439.png" alt="image" style="zoom:80%;" />



과제를 생성할 때 `제출 기간`, `마감이후 제출허용`, `팀별 제출` 등 여러 옵션을 이용할 수 있습니다.</br>

```java
@Override
    public void insertBoard(Map<String, Object> map, HttpServletRequest req) throws Exception {

        String lab_code = (String)map.get("lab_code");
        String board_code = UUID.randomUUID().toString();

        map.put("lab_code",lab_code );
        map.put("board_code",board_code );

        List<Map<String,Object>> list = fileUtils.parseInsertFileInfo(map, req);
        if(list.isEmpty() == false){
            for(int i=0, size=list.size(); i<size; i++){
                labDAO.insertFile(list.get(i));
            }
            String file_id = list.get(0).get("file_id").toString();

            map.put("file_id",file_id);
        } else {
            map.put("file_id",null);
        }

        if(map.containsKey("end_day")){
            String end = (String)map.get("end_day");
            String start = (String)map.get("start_day");

            String start_hour = (String)map.get("start_hour");
            String end_hour = (String)map.get("end_hour");

            String start_minute = (String)map.get("start_minute");
            String end_minute = (String)map.get("end_minute");

            if( start_hour.equals("00")) {
                map.put("start",start+" 00:00:00");
            } else {
                map.put("start", start + " " + start_hour + ":" +  start_minute + ":00");
            }

            if( end_hour.equals("00") ) {
                map.put("end",end+" 23:59:59");
            } else {
                map.put("end", end + " " + end_hour + ":" + end_minute + ":00");
            }
            labDAO.insert_assignment(map);
        }

        labDAO.insertBoard(map);
    }
```

`fileUtils.parseInsertFileInfo` : fileutil 파일에서 parseInsertFileInfo부분을 이용하여 파일을 업로드시켜줍니다. 업로드하기전 파일의 고유 id를 지정해주고 `map`에 넣어 전달해줍니다. 

`start`, `end`  : 제출 기간을 설정할 때 시간과 분을 지정하지않고 날짜만 선택했을 때 기본적으로 설정할 값을 지정해주는 코드입니다.  

<br><br>

`마감이후 제출 허용`은 체크할경우 마감날짜 이후에도 계속하여 과제를 제출가능한 상태입니다.  단, 마감이후에 제출한 학생은 제출현황에서 별도의 표시로 체크됩니다.

![image](https://user-images.githubusercontent.com/78251137/106372409-a6480b80-63b2-11eb-8fdc-1550f0cdb773.png)



![image](https://user-images.githubusercontent.com/78251137/106372417-bc55cc00-63b2-11eb-85d9-1b7d23f79239.png)



```jsp
<div class="card shadow mb-4">
	<c:if test = "${submit_info.file_id == null && assignment.today > assignment.end}">
		<button class="btn btn-danger btn-lg btn-block" type="button">
                                                마감되었습니다.
		</button>
	</c:if>
	<c:if test = "${submit_info.file_id != null && assignment.today > assignment.end}">
		<button class="btn btn-danger btn-lg btn-block" type="button">
                                                마감되었습니다.(마감 이후 수정 불가)
		</button>
	</c:if>
	<c:if test = "${submit_info.file_id != null && assignment.today < assignment.end && submit_info.score != null}">
		<button class="btn btn-danger btn-lg btn-block" type="button">
                                                마감되었습니다. <small>(교수가 해당 과제에 대한 평가를 완료했습니다.)</small>
		</button>
	</c:if>
	<c:if test="${assignment.today < assignment.start}">
		<button class="btn btn-warning btn-lg btn-block" type="button">
                                                제출 기한이 아닙니다. <small>(${assignment.start} 이후 부터 제출 가능)</small>
		</button>
	</c:if>
</div>
```

위 코드는 마감이후 제출 여부에 따라서 수정이 가능한지 불가능한지에 대한 정보를 표시해줍니다.

`assignment.end` : 마감날짜를 의미하며 오늘날짜와 비교하여 마감됐는지, 또는 아직 제출기간인지 표시해주는 역할을 수행합니다.

</br>

</br>

`팀별 제출`은 소속된 Lab에 여러 팀이 존재하고 그 팀안에 개인이 존재하는데 과제를 개인단위가 아닌 팀단위로 제출받을 수 있는 기능입니다. 팀원중 한명이 제출하게되면 다른팀원들은 자동으로 제출처리됩니다.

![image](https://user-images.githubusercontent.com/78251137/106372430-e60ef300-63b2-11eb-89d1-9985ad211973.png)

```java
@RequestMapping(value="/LabBoardDetail.do")
    public ModelAndView labboarddetail(CommandMap commandMap, HttpServletRequest request) throws Exception { //board_num
        ModelAndView mv = new ModelAndView("/lab/LabBoardDetail");
        HttpSession session = request.getSession();
        Map<String,Object> map2 = (Map<String,Object>)session.getAttribute("user_info");

        if(commandMap.getMap().containsKey("room")) { //메인페이지에서 과제 열람 시, session을 Lab모임방이나 캡스톤모임방으로 변경
            map2.put("room_type", ((String)commandMap.getMap().get("room")).charAt(1));
            labService.setLab_code(map2, request);
            map2 = (Map<String,Object>)session.getAttribute("user_info");
        }

        Map<String,Object> map = labService.selectBoardDetail(commandMap.getMap());

        mv.addObject("Detail" , map.get("map"));
        mv.addObject("comment", labService.commentList((Map)map.get("map")));
        mv.addObject("file_info", map.get("file_info"));
        mv.addObject("assignment", map.get("assignment"));

        mv.addObject("money_total",labService.total_money(map2));
        mv.addObject("steams", labService.steams(map2));
        mv.addObject("selectlabboard", labService.selectlabboard(map2));
        mv.addObject("selectteamboard", labService.selectteamboard(map2));
        mv.addObject("selectcommonteamboard", labService.selectcommonteamboard(map2));

        map2.put("board_num", request.getParameter("board_num"));
        map2.put("flag", ((HashMap)map.get("assignment")).get("team_flag"));
        mv.addObject("as_status", labService.assignment_status(map2));
        mv.addObject("as_team_status", labService.assignment_team_status(map2));
        if(map2.get("auth")!=null && map2.get("auth").equals(1)) {
            if(((Map)map.get("assignment")).get("team_flag")!=null&&((Map)map.get("assignment")).get("team_flag").equals("on")) {
                mv.addObject("team_submit_info", labService.assignment_team_status_ck(map2));
            } else {
                mv.addObject("submit_info", labService.assignment_status_ck(map2));
            }
     }

     return mv;
}
```

`team_flag` : 팀별 제출 여부를 판단하는 변수

`labService.assignment_status` : 개인별 과제 제출내역을 담고있는 정보

`labService.assignment_team_status` : 팀별 과제 제출내역을 담고있는 정보

과제현황을 표시하기 전 개인별 제출인지 팀별 제출인지 `team_flag`값으로 판단한뒤 if문을 통하여 해당하는 정보만 `map`에 담아서 view로 넘겨준다.

</br>

</br>

<div id="pro_as2">2. 과제 제출현황</div></br>

소속된 Lab의 모든 과제를 한 페이지에서 볼 수 있도록 현황페이지를 제작했습니다. 해당 페이지는 교수만 확인이 가능합니다. 각 과제의 번호를 클릭하면 해당하는 과제의 게시글로 이동할 수 있고 과제가 늘어나면 스크롤로 표시되어 페이지의 가시성을 향상시켰습니다.

![image](https://user-images.githubusercontent.com/78251137/106372495-a0065f00-63b3-11eb-9761-81f15f0c2c2f.png)

</br>

현황페이지의 특정 문자는 다음과 같은 의미를 가집니다.

`X`  : 과제를 제출하지 않음

`△` : 과제를 제출했으나 마감일 이후에 제출

`○` : 마감일 이전에 정상적으로 과제 제출

`·` : 교수가 점수를 부여하지 않은 상태. 점수를 부여하면 숫자로 바뀌게됩니다.

</br>

</br>

</br>

```sql
<select id="member_list" parameterType="hashmap" resultType="hashmap">
        <![CDATA[
            SELECT
                u.id, u.name, GROUP_CONCAT(IF(score is null, '.', score) ORDER BY write_date) score, GROUP_CONCAT(IF(write_date > end, 1, 0) ORDER BY board_code) check_late, GROUP_CONCAT(board_code ORDER BY board_code) board_code, count(board_code) cnt, ifNull(SUM(score), 0) total
            FROM
                team_member tm
            JOIN
                user_info u
            ON
                tm.student_id = u.id
            LEFT JOIN
                (
                SELECT
                    a.board_code, a.id, a.write_date, a.end, e.score
                FROM
                    assignment_info a
                JOIN
                    board_list bl
                ON
                    a.board_code = bl.board_code
                LEFT JOIN
                    evaluation e
                ON
                    (a.id = e.id
                    AND a.board_code = e.board_code)
                WHERE
                    bl.d_code = 0
                ) a
            ON
                tm.student_id = a.id
            WHERE
                tm.lab_code = #{lab_code}
            group by u.id
        ]]>
</select>
```

`board_code` : 각 게시글에 부여된 고유 코드로써 해당값을 이용하여 조인

`total` : 교수가 부여한 점수의 합계를 표시한 값

여기선 받아와야하는 정보가 많습니다. 모든 현황을 표시해주어야하기 때문에 각 과제의 점수인 `score`, 과제를 순서대로 정렬해주기 위해 오름차순으로 정렬한 `borad_code`, 각 학생에게 부여된 점수의 합산값인 `total` 등 여러 정보를 받아와서 화면에 뿌려주어야합니다.

</br>

</br>

점수는 과제 상세페이지에서 부여할 수 있습니다. 아래의 제출현황을 보면 `상세보기`버튼이 보이게되는데 이를 클릭하면 점수를 부여할 수 있는 모달창을 출력해줍니다.

![image](https://user-images.githubusercontent.com/78251137/106372613-b2cd6380-63b4-11eb-92c5-1f3dc2a71215.png)



 ![image](https://user-images.githubusercontent.com/78251137/106372636-edcf9700-63b4-11eb-9eb5-0099da52622e.png)



숫자를 입력하고 등록버튼을 누르게되면 정상적으로 점수가 부여되며 각각의 점수는 자동으로 합산하여 현황페이지에서 보여줍니다.

</br>

</br>

```java
@Override
    public void assignment_score(Map<String, Object> map) throws Exception {
        String team_flag = (String)map.get("team_flag");

        if(team_flag.equalsIgnoreCase("on")) {
            List<Map<String,Object>> member = labDAO.assignment_teamList(map);
            map.remove("id");
            for(int i=0; i<member.size(); i++) {
                member.get(i).putAll(map);
                labDAO.assignment_score(member.get(i));
            }
        }else{
            labDAO.assignment_score(map);
        }
}
```

점수를 입력하기전 `team_flag` 값을 먼저 확인합니다. `team_flag`가 `on`이라면 개인이 아닌 팀에 점수를 부여해주어야하므로 `map.remove("id")` 를 입력하여 `map`에서 id값을 지워줍니다. 이후 for문을 이용하여 맴버의 숫자만큼 반복해 팀 구성원 모두에게 같은 점수를 부여해줍니다.

반대로 `team_flag`가 `on`이 아니라면 팀 점수가 아닌 개인의 점수이므로 별다른 과정없이 `map`에 있는 값 그대로 쿼리를 실행시켜주면 됩니다.

