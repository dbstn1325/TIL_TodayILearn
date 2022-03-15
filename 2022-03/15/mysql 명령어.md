# mysql 명령어 (root 기준)

## **진입**
**mysql 로그인 (root)**

- mysql -u root -p

**비밀번호 변경 (root)**

- mysql -u root -p 기존암호 password ‘변경암호’

---
## **database**

조회

- show databases;
    

사용

- use user_db


**삭제**

- drop database [database_name];

---
## **Tables**

조회

- show tables;


해당 테이블 조회

* desc user;

해당 테이블 전체 정보 조회

* select * from user;

테이블 초기화 (=테이블 내용 비우기)

* truncate [테이블명]

테이블 삭제

* drop table [테이블명]




[https://snowple.tistory.com/161](https://snowple.tistory.com/161)

