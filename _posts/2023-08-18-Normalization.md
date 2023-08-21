---
title: "[DB] Normalization "
author: "ckh"
date: "2023-08-18 11:41:08 +0800"
categories: ["DB"]
tags: ["DB"]  
---

# DB
DataBase를 쓰는 이유는 
1. 어플리케이션이 자신의 상태와 관계없이 데이터를 보관하고 싶을 때.
 * 하드 디스크등의 저장소에 데이터를 read/write 한다.
2. 데이터를 효율적으로 관리(read/write)하고 싶을 때
 * 예를들어, 데이터를 파일로 read/write해도 저장소 역할이 가능하다. 단, 효율적으로 원하는 데이터를 read/write하고싶다면, DB가 필요하다.
  * > 100만개의 데이터를 저장하고, 이중 하나의 데이터만 필요할 때, 파일 read/write는 내용을 다 직접 한줄한줄(또는 특정 byte 만큼) 하드디스크에서 읽어와야 한다.
    > 이는 심각한 성능의 저하를 불러온다. DB는 자체적으로 read를 염두한 저장이 구현되어 위의 방법보다 훨씬 빠르고 효율적인 방법으로 데이터를 찾아오며, 쿼리 기능과 index등의 기술을 통해
    > 추가적으로 최적화도 가능하다.
  
# RDBMS란
![image](https://github.com/ckh7488/ckh7488.github.io/assets/75701998/b980280a-fd1e-434b-b31d-d5a34295021a)
관계형 데이터 베이스(RDB)는 표 형식으로 데이터를 저장하는 DB를 말한다.  
RDBMS는 RDB를 관리하는 시스템으로, 유저 인터페이스이다. 이 유저 인터페이스를 통하여 우리는 일관적으로 ``table`` 형식으로 데이터를 추상화 한다.
![image](https://github.com/ckh7488/ckh7488.github.io/assets/75701998/2615bfc7-860a-481c-83f5-60b0b892751f)

# Normalization
``목적`` : CRUD 작업이 명확하게 실행되기 위해 Normalization을 함.
* CRUD 작업 시 중복되는 항목이 있는 경우 모든 table을 다 확인해야하는 경우가 있다.
* 특히, table이 복잡하고 row가 많을 경우, 계산의 복잡도가 빠르게 증가하므로, 이를 막기 위해 진행한다.

## example
| StudentID | StudentName | CourseName | CourseInstructor |
|-----------|-------------|------------|------------------|
| 1         | Alice       | Math       | Mr. Smith        |
| 1         | Alice       | History    | Mrs. Jones       |
| 2         | Bob         | Math       | Mr. Smith        |  
위의 표를 보면, 학생이름, 과목, 선생 모두 중복된 부분이 존재한다.  

### 문제점
1. 삽입 이상 : 새로운 학생을 추가시, 무조건적으로 과목명과 선생이 붙어야 한다.
2. 갱신 이상 : MR.smith가 과목을 Math -> science 로 변경해야 할 경우, 모든 row에서 조건을 만족하는 row를 찾아서, 한번에 변경해줘야 한다.
 * 최악의 경우, 테이블 전체를 lock해서 진행해야 하며, 이 경우 심각한 성능저하가 생긴다.
3. 삭제 이상 : 만약 Bob이 Math수업을 안듣는다고 했을 때, Bob이라는 학생의 모든 정보가 없어진다.
위의 목적에서 설명했던 것 처럼, 이러한 당연한 작업들이 정상적으로 작동하기 위해서는 Normalization을 해줘야 한다.

### 정규화  
정규화(Normalization)은 제1,2,3 정규화로 나뉘는데, 직관적으로 보면  
> * 하나의 속성은 하나의 데이터만 가져야 한다.
> * 중복되거나 불필요한 데이터를 제거한다.
>  * 이 과정에서, 각 테이블의 row는 unique하도록 만들어야 좋다.  
> * 테이블의 속성(column)에서, 어떤 속성이 primary key 말고 다른 속성 하나에만 종속되면, 테이블을 나눈다.
이렇게 세가지가 주요하다고 볼 수 있다.

위의 ``example``을 보면
1. 학생 이름에만 학생ID에 종속된다.
2. 선생은 여러 과목을 가르칠 수 있다. unique한 row가 되려면, 선생과 과목을 하나의 primary key로 묶는다.
3. StudentID와 coureseID는 다대다 관계이다. 학생도 여러과목을 수강가능하고, 과목도 정원까지 학생을 수용한다. 연관 테이블을 만들어 연결시킨다.  
 *  N:M 관계의 경우 1:N 과 M:1의 관계를 만들어주는 연관 테이블을 만드는 경우가 많다.
#### Students
| StudentID | StudentName |
|-----------|-------------|
| 1         | Alice       |
| 2         | Bob         |

#### Courses
| CourseID | CourseName | CourseInstructor |
|----------|------------|------------------|
| 1        | Math       | Mr. Smith        |
| 2        | History    | Mrs. Jones       |

#### Enrollment
| StudentID | CourseID |
|-----------|----------|
| 1         | 1        |
| 1         | 2        |
| 2         | 1        |

로 표현 할 수 있다.  
