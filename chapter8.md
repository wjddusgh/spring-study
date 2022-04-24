# 1.JDBC 프로그래밍의 단점을 보완하는 스프링

JDBC 프로그래밍은 같은 코드의 반복이 매우 잦다
스프링은 **템플릿 메소드 패턴, 전략 패턴** 을 엮은 **JdbcTemplate** 클래스를 통해 재사용성을 높였다
또한 람다, 애노테이션 활용을 통해 사람은 **핵심 코드만 집중**해서 작성하면 된다

# 2. 프로젝트 준비

- 자바 프로그램에서 DBMS 커넥션 생성시간은 매우 느리다
- 동시에 여러 사용자가 커넥션 생성을 시작하면 부하가 크다
- **커넥션 풀** : 미리 연결이 되어있는 커넥션들을 모아놓은것
  - 미리 커넥션을 해놓기에 생성시간 아낌
  - 접속자가 몰려도 커넥션 생성 부하가 적어 동시접속자 처리에 유리
  - 커넥션을 일정 갯수로 유지가 가능해 서버 부하 예측 가능

## Tomcat JDBC의 주요 프로퍼티
- **setInitialSize(int)** : 커넥션 풀 초기화시 생성할 커넥션 갯수 (default: 10)
- **setMaxActive(int)** : 커넥션 풀에서 가져올 수 있는 최대 커넥션 갯수 (default: 100)
- **setTestWhileIdle(boolean)** : 커넥션이 풀에 유휴상태로 있는동안 검사 유무 (default: false)
- **setMinEvictableIdleTimeMillis(int)** : 커넥션 풀에 유휴 상태로 유지할 최소시간, mili초 단위 (default: 60000)
- **setTimeBetweenEvictionRunsMillis(int)** : 커넥션 풀의 유휴 커넥션 검사 주기, mili초 단위 (default: 5000)

# 4. JdbcTemplate을 이용한 쿼리 실행

JdbcTemplate 사용시 **Connection, Statement, ResultSet** 직접 사용안해도 됨

## JdbcTemplate을 이용한 조회 쿼리 실행
Select 쿼리 실행을 위한 **query()** 메소드 제공
- RowMapper 인터페이스 활용하여 쿼리 결과를 자바 객체에 매핑
- RowMapper 는 익명클래스, 람다, 직접 클래스 정의 중 골라서 진행

```java
public Member selectByEmail(String email) {
		List<Member> results = jdbcTemplate.query(
				"select * from MEMBER where EMAIL = ?",
				new RowMapper<Member>() {   // 익명클래스
					@Override
					public Member mapRow(ResultSet rs, int rowNum) throws SQLException {
						Member member = new Member(
								rs.getString("EMAIL"),
								rs.getString("PASSWORD"),
								rs.getString("NAME"),
								rs.getTimestamp("REGDATE").toLocalDateTime());
						member.setId(rs.getLong("ID"));
						return member;
					}
				}, email);    // query()의 세번째 인자부터 순서대로 쿼리 문자열의 ?(인덱스 파라미터)에 들어감   

		return results.isEmpty() ? null : results.get(0); // 삼항연산자 활용
	}
```
```java
/* 람다 예시 */
(ResultSet rs, int rowNum) -> {
					Member member = new Member(
							rs.getString("EMAIL"),
							rs.getString("PASSWORD"),
							rs.getString("NAME"),
							rs.getTimestamp("REGDATE").toLocalDateTime());
					member.setId(rs.getLong("ID"));
					return member;
				}
```
- 익명클래스나 람다는 따로 메모리를 차지안하기에 좋지만, 자주 사용시 코드가 길어진다

## 4.3 결과가 1행인 경우 사용할 수 있는 queryForObject() 메서드

```java
public int count() {
		Integer count = jdbcTemplate.queryForObject(
				"select count(*) from MEMBER", Integer.class);
		return count;
	}
```
- 결과가 한개일 경우 사용하는 쿼리 : queryForObjcet()
- 파라미터는 순서대로 sql문자열, 리턴 타입, 인덱스파라미터(가변)
- 결과가 1개 이므로 List가 아닌 하나의 타입으로만 제네릭 되어있음
- 리턴 타입 대신 RowMapper 구현체를 넣으면 RowMapper 구현체 타입으로 리턴타입 지정됨
- 결과 갯수가 1개가 아니라면 익셉션 발생
  - 0개 : EmptyResultDataAccessException
  - 2개 이상 : IncorrectResultSizeDataAccessException

## 4.4 변경 쿼리 실행
insert, update, delete 쿼리는 update() 메소드 활용
- update() 실행 결과로 변경된 행의 개수를 리턴
```java
public void update(Member member) {
		jdbcTemplate.update(
				"update MEMBER set NAME = ?, PASSWORD = ? where EMAIL = ?",
				member.getName(), member.getPassword(), member.getEmail());
	}
```

## 4.5 PreparedStatementCreator를 이용한 쿼리 실행
set() 메서드를 사용해 직접 인덱스파라미터 값 설정 가능(sql인젝션 방지 등을 위해)
사용방법 : set타입 ex. setString(파라미터 인덱스, 값)

```java
PreparedStatement pstmt = con.prepareStatement(
	"insert into MEMBER (EMAIL, PASSWORD, NAME, REGDATE) " + "values (?, ?, ?, ?)");
	pstmt.setString(1, member.getEmail());
	pstmt.setString(2, member.getPassword());
	pstmt.setString(3, member.getName());
	pstmt.setTimestamp(4, Timestamp.valueOf(member.getRegisterDateTime()));
	return pstmt;
	)
```

## 4.6 INSERT 쿼리 실행 시 KeyHolder를 이용해서 자동 생성 키값 구하기
- 키값 생성 방식은 DB에게 위임한다면 따로 query(), update() 메소드에 키 값을 지정하지 않는다 
- 자동 생성된 키 값이 궁금하다면? : KeyHolder 이용

# 5. MemberDao 테스트하기

![1](https://user-images.githubusercontent.com/69251780/164974100-56ee8df8-3bb8-4cee-9f87-44eb8f4c80b7.PNG)

# 6. 스프링의 익셉션 변환 처리

JDBC API 사용중 SQL익셉션이 발생하면 익셉션을 변환해서 발생시킨다
- 이유 : 스프링은 다양한 DB 연동 기술을 제공한다
- 그래서 각각 다른 익셉션이 생기면 개발자는 힘들다
- JPA, 하이버네이트, 마이바티스 등 다양한 DB연동을 사용해도 스프링에서 변환시켜준다
- **DataAccessException** 으로 변환시켜줌

# 7. 트랜잭션 처리

- 여러 쿼리를 한 작업에서 실행할 경우 각 쿼리는 앞의 쿼리에 의존성이 생기는 경우가 있다
- 이 때 하나라도 실패하면 뒤의 쿼리값도 문제가 생김
- 이를 위해 한 작업은 여러 쿼리 모두 성공하거나 아니면 모두 실패하거나로 관리함
- 하나라도 실패시 모든 트랜잭션을 실행 이전으로 돌리는 것 : **Rollback**
- 모든 쿼리가 성공하면 DB에 실제 결과를 반영하는 것 : **Commit**
- JDBC는 setAutoCommit(false)를 이용해 트랜잭션을 시작한다
- commit(), rollback(을 이용해 트랜잭션을 종료한다

## 7.1 @Transactional을 이요한 트랜잭션 처리
- JDBC에서 트랜잭션 처리는 너무 많은 메소드가 들어가서 문제가 생기기 쉬움
- 애노테이션을 사용하면 쉽게 트랜잭션 관리 가능
- **@Transactional** 사용하기 위해 스프링 설정이 필요
	- 플랫폼 트랜잭션 매니저 빈 설정
	- @Transactional 애노테이션 활성화 설정
	- 하지만 알아서해줌






