## Mysql데이터를 백업 받고 AWS RDS로 이전하기

---

### **1. 데이터 백업하기**

이번에는 기존의 온프레미스에서 운영하던 MariaDB의 데이터 일부를 AWS로 이전하라는 지시를 받았습니다. 추후에도 자주 할 수 있다고 생각해서 남깁니다.

기존의 온 프레미스에서 운영하던 MariaDB서버에서 특정 스키마를 백업합니다.

이는 mysqldump라는 명령어를 통해서 진행합니다.

~~~
mysqldump -u [계정명] -p -P [포트번호] -h [아이피] [스키마] > [백업파일명].sql 
~~~

이것으로 데이터 백업을 마무리 했습니다.

백업을 받을 때 데이터 스키마의 크기를 확인하기 위해서 아래의 sql 문을 이용해서 여유 공간이 있는지 확인합니다.

~~~
SELECT table_schema "Database", ROUND(SUM(data_length+index_length)/1024/1024/1024,3) "GB" FROM information_schema.TABLES GROUP BY 1;
~~~

root 사용자로 작업할 경우 / 파티션을 사용하기 때문에 주의를 합니다.

---

### **2. 데이터를 이동하기**

기존의 DB Server에서는 왜그런지 RDS에 접근이 되지 않았습니다.

또한 vsftpd가 설치되지 않았기 때문에 ftp를 통한 파일전송 역시 되지 않았습니다.

우선은 데이터를 넣는게 우선이라 생각해서 putty에 있는 pscp를 이용해서 데이터를 윈도우로 옮긴 다음 AWS RDB로 옮겼습니다.

**MariaDB Server => Window => AWS RDB**

pscp는 기존의 scp명령어와 유사합니다.

cmd를 연다음에 C:\Program Files\Putty 폴더로 이동합니다.

그후 pscp 명령어를 이용해서 파일을 다운받아 옵니다.

파일을 받을 때는 해당 DB에 과부화가 걸릴 수 있으므로 nmon과 같은 자원 모니터링 툴을 이용해서 자원이 과도하게 차지 되는지 지켜보면서 진행합니다.

~~~
#특정 서버에서 파일을 C:\temp\ 내부로 받아옵니다.
pscp [사용자]@[아이피]:[/파일위치]  C:\temp\
~~~

---

### **3. AWS RDB 인스턴스 생성**

AWS의 RDS 인스턴스를 생성해줍니다.

MariaDB의 최신 버전이 10.8버전인데 반해 현재의 AWS RDS의 MariaDB의 버전은 10.6버전입니다.

개발자와 잘이야기해서 버전에 대해 이야기를 하고 진행을 합니다.

DB인스턴스 클래스는 월별 가격을 측정하는 중요한 지표중 하나이니 성능을 잘 정해서 인스턴스를 지정해 줍니다.

db.m6g.large 와 같은 인스턴스 타입이 있는데

여기서 중간의 m6g에서 인스턴스 특징을 정합니다. 

m 버전은 기본 스탠다드 클래스

r및 x클래스는 메모리 최적화 클래스

t는 버스터블 클래스 입니다.

퍼블릭 엑세스를 허용해서 접근이 가능하게 만들어 줍니다.

보안 그룹으로 사내 IP 대역이나 특정 IP만 접근이 가능하게 설정해 줍니다.

스토리지와 이중화 여부 백업 여부등의 체크를 마치고 인스턴스를 생성해 줍니다.

----

### ***4. 각종 설정값 지정***

AWS의 RDS는 각종 설정값을 파라미터 그룹으로 지정합니다.

RDS - 파라미터 그룹 탭으로 이동 후 파라미터 그룹을 생성해 줍니다.

charaterset, time_zone을 설정해 줍니다.

그후 RDS - 데이터베이스 에 접속해서 해당 인스턴수를 수정해서 파라미터 그룹을 바꿔줍니다.

바로 적용을 하기 위해서 인스턴스 재시작을 진행해 줍니다.

**초기 셋팅이라 재시작을 했습니다만 재시작은 함부로 하지 맙시다. 꼭 주변에 물어봅시다.**

설정값이 잘 지정이 된 것을 볼 수 있습니다.

AWS RDS의 해당 mariaDB서버에 접속해서 시간 및 캐릭터셋 설정값을 확인해 줍니다.

~~~
MariaDB [(none)]> select NOW();
+---------------------+
| NOW()               |
+---------------------+
| 2022-06-06 18:59:11 |
+---------------------+
1 row in set (0.005 sec)

MariaDB [(none)]> SHOW GLOBAL VARIABLES LIKE '%zone%';
+------------------+------------+
| Variable_name    | Value      |
+------------------+------------+
| system_time_zone | UTC        |
| time_zone        | Asia/Seoul |
+------------------+------------+
2 rows in set (0.006 sec)

MariaDB [(none)]> show variables like 'c%';
+----------------------------------+---------------------------------------------+
| Variable_name                    | Value                                       |
+----------------------------------+---------------------------------------------+
| character_set_client             | euckr                                       |
| character_set_connection         | euckr                                       |
| character_set_database           | utf8mb4                                     |
| character_set_filesystem         | utf8mb4                                     |
| character_set_results            | euckr                                       |
| character_set_server             | utf8mb4                                     |
| character_set_system             | utf8mb3                                     |
| character_sets_dir               | /rdsdbbin/mariadb-10.6.7.R2/share/charsets/ |
| check_constraint_checks          | ON                                          |
| collation_connection             | euckr_korean_ci                             |
| collation_database               | utf8mb4_general_ci                          |
| collation_server                 | utf8mb4_general_ci                          |
| column_compression_threshold     | 100                                         |
| column_compression_zlib_level    | 6                                           |
| column_compression_zlib_strategy | DEFAULT_STRATEGY                            |
| column_compression_zlib_wrap     | OFF                                         |
| completion_type                  | NO_CHAIN                                    |
| concurrent_insert                | AUTO                                        |
| connect_timeout                  | 10                                          |
| core_file                        | OFF                                         |
+----------------------------------+---------------------------------------------+
20 rows in set (0.006 sec)
~~~

설정을 전부 변경했는데 일부 캐릭터셋이 아직 변경되지 않은 것 같습니다. 해당 문제에 대해서는 추후 확인해 봐야겠습니다.

---

### **5. 데이터 베이스 밀어넣기**

이제는 데이터베이스를 밀어넣으면 됩니다. 

하지만 스키마는 만들어 주고 넣어야 합니다.

데이터 베이스에 접근해서 스키마를 만들어 줍니다.

~~~
MariaDB [none]> create database [스키마명];
MariaDB [none]> exit

# 그리고 나서 해당 스키마에 데이터를 밀어넣어준다.
mysql -u [계정명] -p -P [포트번호] -h [호스트ip]  [스키마명]  <  [백업파일이름].sql
~~~

데이터를 넣고 난 다음에 적용이 잘 되었는지 접근해서 확인해 줍니다.
