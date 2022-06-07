## Mysql스키마 명 변경하기

---

이번에는 Mysql에서 데이터베이스명을 변경하는 방법을 알아본다.

개발용 DB를 생성하는데 데이터베이스명 앞에 "dev_" 라는 접미사가 있어 이를 제거해 달라는 지시를 받아서 나중에도 다시 알아보기 위해서 작성해 둔다.

AWS RDS에서 진행을 하는 것이기 때문에 mysqldump 명령어를 통해서 작업을 진행하면 데이터 비용이 청구될 것 같아서 RENAME 을 이용하는 방법을 사용한다.

데이터베이스명을 바로 변경하는 방법은 없다.

스키마를 생성해 준 다음 원래 데이터베이스에 있는 테이블을 옮겨야 한다.
바꾸고자 하는 데이터베이스 명을 지어준다.

~~~
mysql> CREATE DATABASE [새로운 데이터 베이스 이름];
~~~

그리고 RENAME를 이용해서 테이블을 이동해 주어야 한다.

~~~
RENAME TABLE [기존데이터베이스명].[테이블명] TO [새로운 데이터베이스 명].[테이블명];
~~~

하지만 테이블이 너무 많다면 어떻게 해야 하는가?

여기에 좋은 해결책을 제시해준 블로그가 있어서 해당 내용을 첨부한다.

[해당 블로그](https://ryean.tistory.com/41)

~~~
SELECT concat('RENAME TABLE ',TABLE_SCHEMA,'.',TABLE_NAME,' TO ','[새DB명].',TABLE_NAME,';')
FROM information_schema.tables
WHERE TABLE_SCHEMA LIKE '[기존DB명]';
~~~

~~~
출력 결과
+----------------------------------------------------------------------------------------+
| concat('RENAME TABLE ',TABLE_SCHEMA,'.',TABLE_NAME,' TO ','[새DB].',TABLE_NAME,';')    |
+----------------------------------------------------------------------------------------+
| RENAME TABLE [기존DB].[테이블1] TO   [새DB].[테이블1];                                      |
| RENAME TABLE [기존DB].[테이블2] TO   [새DB].[테이블2];                                      |
| RENAME TABLE [기존DB].[테이블3] TO   [새DB].[테이블3];                                      |
| RENAME TABLE [기존DB].[테이블4] TO   [새DB].[테이블4];                                      |
| RENAME TABLE [기존DB].[테이블5] TO   [새DB].[테이블5];                                      |
| RENAME TABLE [기존DB].[테이블6] TO   [새DB].[테이블6];                                      |
| RENAME TABLE [기존DB].[테이블7] TO   [새DB].[테이블7];                                      |
| RENAME TABLE [기존DB].[테이블8] TO   [새DB].[테이블8];                                      |
| RENAME TABLE [기존DB].[테이블9] TO   [새DB].[테이블9];                                      |
| RENAME TABLE [기존DB].[테이블10] TO  [새DB].[테이블10];                                     |
| RENAME TABLE [기존DB].[테이블11] TO  [새DB].[테이블11];                                     |
| RENAME TABLE [기존DB].[테이블12] TO  [새DB].[테이블12];                                     |
| RENAME TABLE [기존DB].[테이블13] TO  [새DB].[테이블13];                                     |
+----------------------------------------------------------------------------------------+
~~~

해당 명령을 실행하면 자동으로 기존 DB명에서 새 DB로 이동하는 RENAME 구문을 만들어서 출력해준다.

그대로 복붙을 해서 테이블을 옮긴 다음에 정상적으로 옮겨진 것이 확인되면 기존의 DB는 제거한다.

~~~
DROP DATABASE [기존DB명]
~~~