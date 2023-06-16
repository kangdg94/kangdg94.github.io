# [JAVA] Service Project (without Springboot)

## 1. 제작배경

* 첫 토이프로젝트로 자료가 비교적 많고 결과물이 눈에 좀 더 와닿는 웹서비스를 첫 프로젝트로 만들었다.평소 실제 웹사이트를 이용시 사용되는 기능들이 어떤 원리로 사용되는지 궁금하였고 직접 코딩을 하며 구현 해보았다.

#### 1.1 개발 기간

* 2021年 1月10日 ~ 2021年 04月10日

#### 1.2 개발 환경

* Java 1.8.0 / Tomcat 8.5 / BootStrap / JQuery / Mysql

---

## 2. 기능 소개

#### 2.1 회원가입

![join](/assets/img/join.png)



#### 2.2.1 이메일 인증 (SHA-256)

SHA 해쉬값 생성 

#### 해킹방지를 위해 salting값을 바이트 변환 후 해쉬값에 추가해줌

```java
public class SHA256 {
  public static String getSHA256(String input) {
    StringBuffer result = new StringBuffer();
    try { 
      MessageDigest digest = MessageDigest.getInstance("SHA-256");
      byte[] salt = "salting".getBytes();
      digest.reset();
      digest.update(salt);
      byte[] chars = digest.digest(input.getBytes("UTF-8"));
        for ( int i = 0; i < chars.length; i++) {
          String hex =  Integer.toHexString(0xff & chars[i] );
          if(hex.length() == 1 ) result.append(0);
          result.append(hex);
        }
    
    }catch(Exception e) {
      e.printStackTrace();
    }return result.toString();
```



#### Java 라이브러리를 통한 이메일 연동

javax.mail.Authenticator 라이브러리를 통한 Gmail과 연동

```java
package util;

import javax.mail.Authenticator;
import javax.mail.PasswordAuthentication;

public class gmail extends Authenticator {
	protected PasswordAuthentication getPasswordAuthentication() {
		return new PasswordAuthentication("ID", "PW");	
	}
}
```

#### 2.2 게시판 작성

#### 게시판 목록 jsp 화면

#### 게시글 데이터를 10개 단위로 잘라서 list로 출력

![list](/assets/img/list.png)

#### 	2.2.1 게시판 게시

```javascript
for(int i =0 ;i <list.size(); i++)
{
	<tr>
		<td><%= list.get(i).getBbsID() %></td>
		<td><a href = "view.jsp?bbsID=<%=list.get(i).getBbsID() %>"><%= list.get(i).getBbsTitle().replaceAll(" ","&nbsp;").replaceAll("<","&lt;").replaceAll(">", "&gt;").replaceAll("\n","<br>") %></td></a></td>
		<td><%= list.get(i).getUserID() %></td>
		<td><%= list.get(i).getBbsDate().substring(0, 11) + list.get(i).getBbsDate().substring(11,13) + "시" + list.get(i).getBbsDate().substring(14,16) + "분" %></td>
		<td><%= list.get(i).getBbsHit() %></td>
	</tr>
}
```



### 2.2.2 게시판 수정 및 삭제

권한 체크(글쓴이 혹은 관리자) 후 수정, 삭제

![del](/assets/img/del.png)

```java
if(!userID.equals(bbs.getUserID()) && !adminCheck.equals(true)) {
			PrintWriter script = response.getWriter();
			script.println("<script>");
			script.println("alert('권한이 없습니다.')");
			script.println("location.href = 'bbs.jsp'");
			script.println("</script>");	
		}
```



#### 2.2.3 게시판 파일 업로드& 다운로드

![upload](/assets/img/upload.png)

```java
public class downloadAction extends HttpServlet {
	private static final long serialVersionUID = 1L;
 	protected void doGet(HttpServletRequest request, HttpServletResponse response) 
			throws ServletException, IOException {
		String fileName = request.getParameter("file");
		String directory = this.getServletContext().getRealPath("/upload/");
		File file = new File(directory + "/" + fileName);
		
		String mimeType = getServletContext().getMimeType(file.toString());
		if (mimeType == null) {
			response.setContentType("application/octet-stream");
		}
		String downloadName = null;
		if (request.getHeader("user-agent").indexOf("MSIE") == -1 ) {
			downloadName = new String(fileName.getBytes("UTF-8"), "8859_1");
		}else {
			downloadName = new String(fileName.getBytes("EUC-KR"), "8859_1");
		}
		
		response.setHeader("Content-Disposition", "attachment;filename=\""
				+ downloadName + "\";");
		
		FileInputStream fileInputStream = new FileInputStream(file);
		ServletOutputStream servletOutputStream = response.getOutputStream();
		
		byte b[] = new byte[1024];
		int data = 0;
		
		while ((data = (fileInputStream.read(b,0,b.length))) != -1 ) {
			servletOutputStream.write(b, 0, data);
		}
		servletOutputStream.flush();
		servletOutputStream.close();
		fileInputStream.close();
```

![download](/assets/img/download.png)

파일 다운로드 가능

#### 2.3 미니 검색엔진

![engine1](/assets/img/engine1.png)

내용만 혹은 전체의 콤보박스를 통해 내용 필터만 검색 가능

![engine2](/assets/img/engine2.png)



#### 2.4 Ajax를 통한 비동기식 게시판 현황 최신화

![ajax](/assets/img/ajax.png)



```javascript
var request = new XMLHttpRequest();
			request.open("post", "/KangTest/BbsSearchServlet");
			request.onreadystatechange = function()
```

```java
public class BbsSearchServlet extends HttpServlet {
	private static final long serialVersionUID = 1L;
       
	protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
		request.setCharacterEncoding("UTF-8");
		response.setContentType("text/html; charset=UTF-8");

		String result = getJason();
		PrintWriter out = response.getWriter();
		out.print(result);
		
	}
	public String getJason(){
		StringBuffer result = new StringBuffer("");
		result.append("{\"result\":[");
		BbsDAO bbsDAO = new BbsDAO();
		ArrayList<Bbs> bbslist = null;
		bbslist = bbsDAO.search();
		for( int i=0; i < bbslist.size(); i++)
		{
			result.append("[{\"value\":\"" + bbslist.get(i).getUserID()+ "\"},");
			result.append("{\"value\":\"" + bbslist.get(i).getBbsTitle()+ "\"},");
			result.append("{\"value\":\"" + bbslist.get(i).getBbsDate()+ "\"}] ");
			if(i != bbslist.size() - 1)
			{
				result.append(",");
			}
		}
		result.append("]}");
		return result.toString();
	}

}

```

#### 2.5 마이페이지

##### 2.5.1 개인정보 수정

![edit](/assets/img/edit.png)

##### 2.5.2 게시판 게시 내역 조회

![myhistory](/assets/img/myhistory.png)



#### 3. 마무리

* 평소 실제 웹사이트를 이용하면서 사용되는 기능들이 어떤 원리로 사용되는지 알게 되었고 공부 하면 할수록 더 많은 것들이 있고 내가 알고 있던것들은 빙산의 일각이라는 느낌을 받았다. 앞으로 더 열심히 공부하고 새로운 it기술이나 cs지식을 관심있게 주시하고 한번씩 사용 해보도록 해야겠다.   ps.프레임워크 없이 개발하니 ~~개고생~~이다.
