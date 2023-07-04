
## MySql 인덱스

###### 정보의 출처
###### 1. real my sql 1권
###### 2. 패스트 캠퍼스 강의
###### 3. inno db index structure :
- http://d.jcole.us/blog/files/innodb/20130109/72dpi/B_Tree_Detailed_Page_Structure.png


</br>

### B+ 트리

![스크린샷 2023-07-04 오후 10 36 09](https://github.com/dkGithup2022/mysql_index_araboja/assets/104286091/c7bb47b2-8d1b-4b01-8e1e-3f13fe259236)

ㄴ> innoDB index structure 라는 글에서 가져왔습니다  :  http://d.jcole.us/blog/files/innodb/20130109/72dpi/B_Tree_Detailed_Page_Structure.png

</br>

#### WHY B+ Tree

1. balanced tree 이고 항상 같은 depth 를 보장한다.

   -> 조회에 균일한 비용이 소모된다.

   -> 같은 depth 를 탐색한다 == 같은 수의 페이지 io 가 필요하다는 뜻이기 때문


</br>

2. 중간 단계 노드는 값을 가지고 있지 않고, 리프 노드에만 값이 정렬되어 있다.

   -> range 스캔은 { b+ 트리 탐색 }  + { 순차 탐색 } 으로 수행이 가능하다.


</br>


</br>

3. 중간 노드와 리프 노드는 아래처럼 생겼다.

- 리프노드
  ![스크린샷 2023-07-04 오후 10 44 53](https://github.com/dkGithup2022/mysql_index_araboja/assets/104286091/5cbb5029-2850-4a3a-b5db-d56b598efb70)

- 중간 노드
  ![스크린샷 2023-07-04 오후 10 45 40](https://github.com/dkGithup2022/mysql_index_araboja/assets/104286091/33377cda-c1df-4d9c-b015-da5ebc200a7d)



</br>


##### 클러스터링 인덱스

#### 1. 구조

InnoDB 는 클러스터링 인덱스라는 정책을 pk 생성의 규칙으로 삼는다.


![스크린샷 2023-07-04 오후 10 53 01](https://github.com/dkGithup2022/mysql_index_araboja/assets/104286091/3a788671-0d59-4664-bac0-8f74348fb54a)

규칙은 아래와 같다.

1. pk 값에 대해 테이블의 원본 로우를 정렬한다.
2. secondary 인덱스 ( pk 가 아닌 인덱스 ) 는 원본에 대한 참조가 아닌 pk 값을 가지고 있는다.


</br>

#### 2. 장단점


- 장점
    - pk 에 대한 검색이 아주 빠르다.
    - 커버링 인덱스 ,인덱스 지정 값에 대한 칼럼만 조회, 에서 이득을 볼 수 있다.


- 단점
    - pk 의 크기가 크면 해당 테이블의 전체 인덱스 페이지에 로우가 많이 안들어간다.
    - 인덱스 검색 이후, pk 를 통해 원본을 다시 검색해야한다.
    - pk 변경에 대한 비용이 아주 크다. ( 물리적 위치의 변경 필요 )


</br> 

#### 3. 주의점


1. 키의 크기

   index 저장을 위한 b+ 트리 구조에서, 각 페이지는 16kb 의 용량을 가지고 있음.(default 기준)

   해당 페이지를 구성할때, pk 의 크기가 아주 크다면 한 페이지에 들어가는 index 의 갯수가 적어지고 검색에 상대적으로 많은 page io 가 필요할 것임. 따라서 너무 큰 pk 값은 좋지 않다.


2. pk 설정 주의 사항

   pk 값은 자동으로 clustering 인덱스로 지정되며 조회에 대한 이득을 가지고, CUD 에 오버헤드를 가진다.

   자동 지정된 pk 를 사용하면 의도치 않은 쓰기, 수정 오버헤드를 가질 수 있고, clustering 인덱스의 이득을 보지 못한다.

   따라서 real mysql 의 저자는 아래의 규칙을 추천함.


        - 비즈니스 로직상 pk 가 될 수 있는 칼럼이 있다면 pk 로 직접 명시.
        - 키가 너무 커지거나, 마땅히 지정할 키가 없다면 id 를 위한 칼럼 생성 후 pk 지정.
        - 어떤 경우에서건 자동 지정된 pk 를 쓰지는 말자.

</br>

</br>

### 인덱스 선정 요소

#### 키값의 크기

1. 페이지 크기와 깊이

    - index 크기가 작아야 페이지 하나에 많이 들어감
    - 전채 로우 갯수 대비 페이지 수가 적어야 적은 수의 page io 탐색으로 b+ 트리 탐색을 마칠 수 있음.
    - 따라서 너무 큰 index 는 좋지 않음 .

</br>


#### 선택도 ( 기수성 )

1. country(10개) 검색과 city(100000 개) 검색의 인덱스.

어떤 테이블에 100000 개의 데이터가 들어있는데, country 가 korea 인 데이터는 1000 개, city가 seoul인 데이터는 10 개의 데이터가 있다고 하자.

```sql

select * from local_tmp 
where city = 'seoul' and country = 'korea'

```

라는 쿼리를 수행 할 때,

country 에 인덱스가 걸리면 1000 개의 로우에 대한 필터가 필요하다.
city 에 인덱스가 걸리면 10개의 로우에 대한 필터가 필요하다.


따라서 애초에 중복값을 많이 포함하지 않은 곳에 인덱스를 거는 것이 유리하다.


</br>

</br>


##### 복합 인덱스 그려보기

아래와 같은 ddl 에 의해 만들어진 인덱스는 아래 이미지 처럼 정렬된다.

```sql

alter table some_table
add index idx_two_index( index1, index2) ;

```

</br>



![스크린샷 2023-07-04 오후 11 26 07](https://github.com/dkGithup2022/mysql_index_araboja/assets/104286091/dbf62cb0-5ff9-4069-b20c-c8dcdb014ae0)


1. 순서가 중요한 이유

- 위의 정렬 상태에서  index1 에 대해 조회하거나, index 1, index2 두 칼럼에 대해 조회하면 온전히 인덱스를 탈 수 있다.
- 하지만 index2 만 읽는 다면 정상적으로

2. 그래도 읽히긴 한다 ( 스킵 스캔 )

- 인덱스의 스킵 스캔은 아래와 같은 상황에서 동작한다.

```sql

alter table some_table
add index idx_two_index( index1, index2) ;

-- (1) 멀티 칼럼 인덱스가 걸려 있고 


select * from some_table where index2 > 10;

-- index2 에 대한 칼럼 조건만 있는 경우

```

- 이러고 쿼리 explain 을 찍어보면

```text

id : 1 
table : some_table
type: range
key : idx_two_index
extra : using where; Using index for skip scan

```

이라고 나옴. mysql 옵티마이저가 index1 칼럼의 유니크한 값들에 대해 각각 두번째 인덱스 조건으로 검색을 수행한다.



### 커버링 인덱스



### 부록 : 외래키의 생성과 관리

#### 부모가 락에 걸리는 경우

#### 자식이 락에 걸리는 경우 