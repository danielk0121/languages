- character set 과 collation
- 
- 캐릭터셋은 말 그대로 문자의 집합으로, 정확히는 문자가 어떤 코드로 저장될지 하는 문자와 encoding 의 집합이다
- 콜레이션은 dbms 에서 이 캐릭터셋의 데이터를 처리할때 필요한 규칙의 정의이다. 이는 정렬, 검색, 변환, 비교 등의 상황에서 필요하다
- 예를 들어보면
- utf-8 은 캐릭터셋
- mysql 에서 utf-8 캐릭터셋의 콜레이션은 utf8_bin, utf8_czech_ci, utf8_general_ci 등이 있다
- bin 은 binary_collation : 무조건 인코딩된 byte stream 값으로 문자를 비교
- ci 는 case-insensitive-collation : 대소문자 구분하지 않겠다는 뜻

```sql
# 1. binary 함수를 사용해서 like 조건에 대소문자가 적용되도록 하는 방법
# => 스키마를 변경하지 않아도 된다
# like 검색 시 대소문자 구분이 무시되는 기본 설정을 유지함

# where binary(컬럼명) like ‘%Abc%’

# 2. varchar 컬럼을 binary charset 으로 변경하는 방법
# 스키마를 변경함
# like 검색 시 대소문자가 구분이 됨
# 이 경우 대소문자 구분 없이 like 검색을 하려면 함수를 사용해야 됨

# where lower(컬럼명) like lower('%Abc%')

# alter table mytable_bak modify value varchar(150) character set utf8 collate utf8mb3_bin default '' not null ;
```