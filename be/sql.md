# sql

### 접속

```bash
cd /usr/local/mysql/bin
# path 환경변수 설정해주기
# $ sudo vim /etc/paths
#      /usr/local/mysql/bin 추가하기
# $ mysql -V 혹은 $ echo $PATH 로 잘 등록됐는지 확인


# root
sudo ./mysql -p
./mysql -uroot -p

# 특정 유저, db 에 접속할 때 
# -h host -u user -p password 옵션
# connectdb 는 database 이름~!
./mysql -h127.0.0.1 -uconnectuser -p connectdb
```

### 유저 생성 및 권한 부여

```bash
CREATE USER connectuser@localhost IDENTIFIED BY 'connect123!@#';

GRANT ALL PRIVILEGES ON connectdb.* TO 'connectuser'@'localhost';

FLUSH PRIVILEGES;
```

### 현재 버전과 날짜 구하기

```bash
SELECT VERSION(), CURRENT_DATE;
```

## SQL

### SELECT

```bash
show databases; # 데이터베이스 목록 보여줌
use [database name]; # 이 데이터베이스를 사용할 거야

show tables; # 테이블 있나없나
desc [table name]; # 테이블 구조 확인

select * from [table name];

select [culmn name (alias)] ,... from [table name];
select deptno 부서번호, name 부서명 from department;

select concat( empno, '-', deptno) '사번-부서번호' from employee;

select distinct deptno from employee;

select empno, name, job 
from employee 
order by name desc;

select name, hiredate from employee
where hiredate < 1981;

select name, deptno from employee 
where deptno = 30 and salary<1500;

select name, deptno from employee
where deptno in (10, 30);

select name, job from employee
where name like '%A%'; # 이름에 'A'가 포함인 경우

where name like 'A%'; # 'A' 로 시작

where name like '%A'; # 'A' 로 끝

where name like '_A%'; # 이름에 두번째에 'A'가 포함인 경우

where name like '__A%'; # 이름에 세번째에 'A'가 포함인 경우

# 테이블 없는 함수 조회
select upper('Seoul'), ucase('EeedgEI'); # 대문자로 바꿔
select lower('EEEseiei'), lcase('eEIEIEI'); # 소문자로 바꿔줘

select lower(name) from employee;

select substring('happy day', 3, 2); # 3인덱스에서부터 2개

select lpad('hi', 5,'!'); # 5길이 중에 공백이 생길 경우 왼쪽을 ! 로 채워줘
select lpad('hi', 4,'?'); # 4길이 중에 공백이 생길 경우 오른쪽을 ? 로 채워줘

select lpad(name,5, '+') from employee;

select ltrim('  hello'), rtrim('  eiei  '), trim(' eieie   '); # 공백 없애줘
select trim(both 'x' from 'xxxhellxxx');

# select (CAST 형변환)
# 타입 : BINARY, CHAR, DATE, DATETIME,TIME, SIGNED INTEGER, UNSIGNED INTEGER
select cast(now() as date);
select cast(1-2 as unsigned);

# select (그룹함수)
select count(*) from employee;
select avg(salary) from employee;
select avg(salary), sum(salary) from employee where deptno >= 30;

select deptno, avg(salary), sum(salary) from employee 
group by deptno;
```

### INSERT/UPDATE/DELETE&#x20;

```bash
# insert
# INSERT INTO 테이블 이름(필드1, 필드2,,,)
# VALUES (값1, 값2,..)

insert into role values(200, 'CEO');
insert into role(role_id) values(201); # 명시되지 않을 경우 null 들어감

# update
# UPDATE 테이블 명
# SET 필드1=필드1 값, 필드2=필드2 값,,,
# WHERE 조건 식 // 조건식 없을 경우 전체를 다 업데이트;; 주의!
update role
set description='CTO'
where role_id = 200;

# delete
# DELETE 
# FROM 테이블 명 
# WHERE 조건 // 조건식 없을 경우 전체 삭제

```

### CREATE

```bash
# create
# CREATE TABLE 테이블 명 (
# 필드명1 타입 [NULL | NOT NULL][DEFAULT][AUTO_INCREMENT],
# 필드명2 타입 [NULL | NOT NULL][DEFAULT][AUTO_INCREMENT],
# .....
# PRIMARY KEY(필드명)
# );
create table EMPLOYEE2(
    -> empno integer not null primary key,
    -> name varchar(10),
    -> job varchar(9),
    -> boss integer,
    -> hiredate varchar(12),
    -> salary decimal(7,2),
    -> comm decimal(7,2),
    -> deptno integer);
    

# alter 테이블 컬럼 추가/삭
# ALTER TABLE 테이블명
# ADD 필드명 타입 [NULL | NOT NULL][DEFAULT][AUTO_INCREMENT];
#
# ALTER TABLE 테이블 명
# DROP 필드명;
#
# ALTER TABLE 테이블 명
# CHANGE 필드명 새필드명 타입 [NULL | NOT NULL][DEFAULT][AUTO_INCREMENT];
# 
# ALTER TABLE 테이블 명
# RENAME 새테이블 명;


alter table EMPLOYEE2
    -> add birthday integer;
    
alter table EMPLOYEE2
    -> drop birthday;
    
alter table EMPLOYEE2
    -> change birthday birth integer;
    
alter table EMPLOYEE2
    -> rename employee3;

```
