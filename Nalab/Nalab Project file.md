# Na-Lab Project



## 게시판 파일관리



* 목차
  1. [파일업로드](#up)
  2. [파일수정 및 삭제](#mod)
  3. [파일다운로드](#down)

-----------------------



<div id="up">●  파일업로드</div>

Na-Lab 페이지에서 이용할 게시판에 파일을 업로드하는 기능을 추가하였습니다.



먼저 기본설정입니다.

```xml
<!-- MultipartHttpServletRequset -->
<dependency>
	<groupId>commons-io</groupId>
	<artifactId>commons-io</artifactId>
	<version>2.4.0</version>
</dependency>
<dependency>
	<groupId>commons-fileupload</groupId>
	<artifactId>commons-fileupload</artifactId>
	<version>1.3.1</version>
</dependency>
```

pom.xml에 파일 업로드 및 다운로드를 위한 dependency를 추가해주었습니다.

```xml
<!-- MultipartResolver 설정 -->
<bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
	<property name="maxUploadSize" value="100000000" />
	<property name="maxInMemorySize" value="100000000" />
</bean>
```

그리고 서버에 업로드할 수 있는 파일의 최대크기를 설정해주었습니다.





기본적인 설정이 완료되었으면 파일을 업로드하기 위한 코드를 작성합니다.

작성한 코드는 다음과 같습니다.

```java
public List<Map<String,Object>> parseInsertFileInfo(Map<String,Object> map, HttpServletRequest request, String filePath) throws Exception{
		MultipartHttpServletRequest multipartHttpServletRequest = (MultipartHttpServletRequest)request;
    	Iterator<String> iterator = multipartHttpServletRequest.getFileNames();
    	
    	MultipartFile multipartFile = null;
    	String originalFileName = null;
    	String originalFileExtension = null;
    	String storedFileName = null;
    	
    	List<Map<String,Object>> list = new ArrayList<Map<String,Object>>();
        Map<String, Object> listMap = null; 
        String file_id  = null;
        int order = 0; // 순서
        
        //파일 id있는지 검사
        if(map.containsKey("file_id") != false){
        	file_id = (String)map.get("file_id");
        } else {
        	file_id = UUID.randomUUID().toString();
        }
        
        if(map.containsKey("order") != false){
        	order = (int)map.get("order");
        }
        
        File file = new File(filePath);
        if(file.exists() == false){
        	file.mkdirs();
        }
        
        while(iterator.hasNext()){
        	multipartFile = multipartHttpServletRequest.getFile(iterator.next());
        	if(multipartFile.isEmpty() == false){
        		originalFileName = multipartFile.getOriginalFilename();
        		originalFileExtension = originalFileName.substring(originalFileName.lastIndexOf("."));
        		storedFileName = CommonUtils.getRandomString() + originalFileExtension;
        		
        		file = new File(filePath + storedFileName);
        		multipartFile.transferTo(file);
        		listMap = new HashMap<String,Object>();
        		listMap.put("file_id", file_id);
        		listMap.put("ori_file_name", originalFileName);
        		listMap.put("real_file_name", storedFileName);
        		listMap.put("file_size", multipartFile.getSize());
        		listMap.put("order", order++);
        		list.add(listMap);
        	}
        }
		return list;
}
```

업로드될 파일은 5가지의 정보를 가지게됩니다.

`file_id` : 기본키이며 각 파일의 고유 id

`originalFileName` : 유저가 지정한 파일의 이름

`storedFileName` : 서버에 저장될 파일의 이름

`file_size` : 파일 크기 정보. 설정해둔 값보다 큰 사이즈일 경우 javascript를 통해 메시지 출력

`order` : 파일의 순서



만약 2가지 이상의 파일을 한꺼번에 업로드하게되면 while문에 의해 순서대로 업로드됩니다.



<div id="mod">● 파일수정 및 삭제</div>

![image](https://user-images.githubusercontent.com/78251137/106371569-4a2cb980-63a9-11eb-90be-3681908d8dba.png)



게시글 수정페이지로 넘어가면 해당 게시글의 첨부파일이 모두 나타나게되고 몇 가지 기능을 이용할 수 있습니다.

첨부파일 옆의 `삭제`버튼을 누르면 현재 화면에서 파일이 제거되고 이후 `수정하기`버튼을 누르면 실제 파일은 서버에 남겨두고 데이터베이스상에서 삭제되었다는 표시를 해주게됩니다.

새로운파일을 추가하려면 `파일추가`버튼을 클릭하여 새롭게 파일을 추가시킬 수 있습니다.

실제 코드는 아래와 같습니다.

```jsp
<c:if test="${delete_division ne 'report'}">
	<form action="${pageContext.request.contextPath}/UpdateLabBoard.do" method="post" name="fr" onsubmit="return checkform();" style="float:left;" enctype="multipart/form-data">
		<div class="card-body">
			<input type="hidden" name="board_num" value="${board_num}">
			<input type="hidden" name="file_id" value="${Detail.file_id}">
            
			<div class="mb-3">
				<label for="title" style="padding-left: 10px; padding-top: 5px; font-weight:bold;">제목</label>
				<input type="text" class="form-control" name="title" id="title" value="${title}">
			</div>

			<div class="mb-3">
				<label for="content" style="padding-left: 10px; font-weight:bold;">내용</label>
				<textarea class="form-control" rows="5" name="content" >${content}</textarea>
			</div>

			<div class="mb-3" id="fileDiv">
				<label style="padding-left: 10px; font-weight:bold;">첨부파일</label>
				<a href="#this" id="addFile" class="btn btn-warning btn-sm">파일추가</a>
				<br/>
				<c:if test="${fn:length(file_info) eq 0}">
					<a href="#">첨부파일 없음.</a>
				</c:if>
				<c:forEach items="${file_info}" var="file" varStatus="var">
					<p>
						<input type="hidden" name="IDX_${var.index }" value="${file.IDX}">
						<input type="hidden" name="file_id_${var.index }" value="${file.file_id}">
						<a href="#this" name="name_${var.index}">${file.ori_file_name}</a>
						<input type="file" id="file_${var.index}" name="file_${var.index}" style="display:none" onchange="checkFile(this)">
						<a href="#this" class="btn btn-danger btn-sm" id="delete_${var.index}" name="delete_${var.index}">삭제</a>
					</p>
				</c:forEach>
			</div>

			<input type="submit" class="btn btn-info btn-sm ml-2" value="수정하기">&nbsp;
			<button type="button" class="btn btn-success btn-sm" onclick="window.history.back();">돌아가기</button>
		</div>
	</form>
</c:if>
```

p태그로 감싸진부분은 javascript를 통해 파일추가버튼을 클릭했을 때 추가적으로 생성되는 부분입니다. 



<div id="down">● 파일다운로드</div>

```jsp
<tr>
	<th scope="row" width="70px">첨부파일</th>
	<td>
		<c:if test="${fn:length(file_info) eq 0}">
			<a href="#">첨부파일 없음.</a>
		</c:if>
		<c:forEach items="${file_info}" var="file">
			<a href="#" onclick="$(this).children().submit();">
				<form action="${pageContext.request.contextPath}/Filedown.do" method="post" style="display:none;">
					<input type="hidden" value="${file.file_id}" name="file_id">
					<input type="hidden" value="${file.real_file_name}" name="real_file_name">
				</form>
				${file.ori_file_name}<br/>
			</a>
		</c:forEach>
	</td>
</tr>
```

파일 다운로드는 `file_id`와 `real_file_name`을 이용하여 진행합니다. 두 값을 히든으로 넘겨주고 두가지 값이 일치하는 데이터를 데이터베이스에서 찾아 그 정보를 서버로부터 다운받는 방식입니다.



```java
@RequestMapping(value="/Filedown.do")
public void downloadFile(CommandMap commandMap, HttpServletResponse response) throws Exception{
	Map<String,Object> map = labService.Filedown(commandMap.getMap());

	try {
		byte fileByte[] = FileUtils.readFileToByteArray(new File("files/file" + map.get("real_file_name")));
		response.setContentType("application/octet-stream");
		response.setContentLength(fileByte.length);
		response.setHeader("Content-Disposition", "attachment; fileName=\"" + URLEncoder.encode((String) map.get("ori_file_name"), "UTF-8") + "\";");
		response.setHeader("Content-Transfer-Encoding", "binary");
		response.getOutputStream().write(fileByte);
		response.getOutputStream().flush();
		response.getOutputStream().close();
	} catch(Exception e) {
		log.error("ERROR:::::"+e);
	}
}
```

labService.Filedown를 통해서 첨부파일의 정보를 가져오고 UTF-8 인코딩처리까지 해주면 파일 다운로드 기능을 사용할 수 있습니다.

