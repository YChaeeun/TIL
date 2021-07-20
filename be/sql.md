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

### SQL

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

```



