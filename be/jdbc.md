# jdbc

## JDBC ( Java Database Connectivity)

* 자바를 이용한 데이터베이스 접속과 SQL 문장의 실행, 그리고 실행 결과로 얻어진 데이터의 핸들링을 제공하는 방법과 절차에 관한 규약
* 자바 프로그램 내 SQL 문을 실행하기 위한 자바 API
  * 자바는 표준 인터페이스인 JDBC API 를 제공

### JDBC 클래스 생성 관계

* DriverManager
  * Connect
    * Statement
      * ResultSet

### JDBC 수행 단계

1. import
2. 드라이버 로드
3. Connection 객체 얻기
4. Statement 생성
5. 질의 수행
6. ResultSet으로 결과 받기
7. Close

```java
public int addGuestBook(GuestBookVO vo){
		int result = 0;
		Connection conn = null;
		PreparedStatement ps = null;
		try{
			conn = DBUtil.getConnection();
			String sql = "insert into guestbook values("
					+ "guestbook_seq.nextval,?,?,?,sysdate)";
			ps = conn.prepareStatement(sql);
			ps.setString(1, vo.getId());
			ps.setString(2, vo.getTitle());
			ps.setString(3, vo.getConetnt());
			result = ps.executeUpdate();
		}catch(Exception e){
			e.printStackTrace();
		}finally {
			DBUtil.close(conn, ps);
		}
		
		return result;
	}
```
