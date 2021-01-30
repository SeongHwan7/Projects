# Schoolware Project



## 졸업사정회 유지보수



* 목차
  1. [500 server error](#500-server-error)
  2. [성능향상](#Improving)

-----------------------



첫번째로 유지보수를 담당하게된 졸업사정회 기능입니다. 이미 개발된 기능이었고 잘 사용하고 있었으나 다른 기능을 개발하는 과정에서 졸업사정회에 오류가 발생했다고하여 면밀히 살펴보고 수정한 내용입니다.



#### ● <div id="500-server-error">500 server error</div>

1. 원인분석

   졸업사정회의 기능은 아래 화면처럼 자신의 학과에 해당하는 과목이 나타나고 빨간글씨로된 과목을 클릭할 경우 과목에 줄이 그어지며 학점이 합산되어 졸업가능 여부를 체크할 수 있습니다.

   ![졸업사정회1](https://user-images.githubusercontent.com/78251137/106366849-ef339c00-6381-11eb-92e3-fd5b671ed0d4.png)

   

   하지만 직접 테스트를 거친결과 빨간글씨를 클릭했을때 줄이 정상적으로 그어지나 학점이 합산되지않았고 해당 과목을 취소하려고하면 500 error가 발생하는것을 확인했습니다.

    <img src="https://user-images.githubusercontent.com/78251137/106366108-09b74680-637d-11eb-86ee-e27bc1906c7b.png" alt="500"  />

   

   코드가 수정된적은 없었기때문에 sql에 문제가 있을것이라 생각하여 찾아보았고 문제를 발견했습니다.

   아래는 문제가 있었던 sql의 일부분입니다.

   ```mysql
   (
   	SELECT 
   		sum(grades) as sum
   	FROM
   	    graduated_student_subject
   	WHERE
   	    classification = '전공필수'
   	AND
   		id = #{s_id}
   )
   , major_select2 =
   (
   	SELECT 
   		sum(grades) as sum
   	FROM
   	    graduated_student_subject
   	WHERE
   	    classification = '전공선택'
   	AND
   	    id = #{s_id}
   )
   ```

   

   과목을 클릭해도 점수가 합산이 되지 않기때문에 sum(grades)에 문제가 있다고 판단했고 해당 부분을 살펴본 결과는 아래와 같습니다.

    ![그림3](https://user-images.githubusercontent.com/78251137/106368137-d24f9680-638a-11eb-9554-71c1072fc9a6.png)

    ![그림4](https://user-images.githubusercontent.com/78251137/106368164-fad79080-638a-11eb-9061-8ea6d0108b16.png)

   

   NotNULL로 설정되어있으나 실제로는 `null`값이 삽입되고있었고 이로인해 전체적인 시스템에 문제가 발생했습니다.

   

2. 문제해결

   문제가 파악되었으므로 이를 해결하기위하여 `null`값을 `0`으로 치환하도록 sql을 수정해주었습니다. 

   ```mysql
   (
   	SELECT 
   		ifnull(sum(grades),0) as sum
   	FROM
   	    graduated_student_subject
   	WHERE
   	    classification = '전공필수'
   	AND
   		id = #{s_id}
   )
   , major_select2 =
   (
   	SELECT 
   		ifnull(sum(grades),0) as sum
   	FROM
   	    graduated_student_subject
   	WHERE
   	    classification = '전공선택'
   	AND
   	    id = #{s_id}
   )
   ```

   

#### ● <div id="Improving">성능향상</div>

아래 사진은 Schoolware에 존재하는 졸업사정회 학생조회 기능입니다.

![그림5_LI](https://user-images.githubusercontent.com/78251137/106368507-4db24780-638d-11eb-92f5-55cd49571660.jpg)

해당 페이지에 접근할 때 페이지 로딩시간이 매우 길어 해당 기능의 로딩속도를 개선하고자 데이터베이스에서 테스팅을 진행하였습니다.



테스팅 결과는 다음과 같습니다.

 <img src="https://user-images.githubusercontent.com/78251137/106368579-f791d400-638d-11eb-9009-5269166e885f.png" alt="image" style="zoom:80%;" />

졸업사정회 학생조회 페이지에 필요한 데이터베이스를 불러오는데 걸리는시간이 약5초가 소요되었고 이로인하여 페이지를 로딩하는데 걸리는 시간이 길어지는것을 파악하였습니다.



이 부분을 수정하기위하여 mysql에서 `explain`명령어를 이용해 테이블을 살펴보았습니다.

![그림6](https://user-images.githubusercontent.com/78251137/106368700-075de800-638f-11eb-9721-14fa5b2190c7.png)

gus 테이블의 타입이 `ALL`로 설정되어있어 데이터 풀스캔이 발생했고 이로인해 많은양의 데이터를 읽어오느라 약 5초의 지연시간이 발생했습니다.



해당 부분은 다음과 같이 수정하였습니다.

![그림7](https://user-images.githubusercontent.com/78251137/106368701-07f67e80-638f-11eb-8712-aa685ea0d02f.png)

type을 `ALL`에서 `eq_ref`로 수정하여 PRIMARY_key를 비교하여 데이터를 불러올 수 있도록 수정해주었습니다.



수정된 결과는 다음과 같습니다.

 <img src="https://user-images.githubusercontent.com/78251137/106368579-f791d400-638d-11eb-9009-5269166e885f.png" alt="image" style="zoom:80%;" />

기존에 약 5초이상 걸리던 Fetching Time이

 <img src="https://user-images.githubusercontent.com/78251137/106368822-07aab300-6390-11eb-9bf0-1b8b5277103c.png" alt="image" style="zoom:50%;" />

약 0.05초로 줄어들어 성능이 향상되었습니다.

