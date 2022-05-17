---
layout: single
title:  "[DataBase] - SQL Reference Conventions"
category: [Database, Redshift]
tag: [Database, Redshift]
---

# 개요 

SQL Document를 보면 쿼리 규칙에 대해서 아래와 같이 표시되곤 한다.  
`[`, `{` 같은 것들이 무엇을 뜻하는 것일까? 오늘은 이에 대해서 살펴보자.

```sql
[ WITH with_subquery [, ...] ]
SELECT
[ TOP number | [ ALL | DISTINCT ]
* | expression [ AS output_name ] [, ...] ]
[ FROM table_reference [, ...] ]
[ WHERE condition ]
[ GROUP BY expression [, ...] ]
[ HAVING condition ]
[ { UNION | ALL | INTERSECT | EXCEPT | MINUS } query ]
[ ORDER BY expression [ ASC | DESC ] ]
[ LIMIT { number | ALL } ]
[ OFFSET start ]
```

<br>

# SQL Reference Conventions

규칙은 플랫폼별로 거의 동일한 방식으로 구성되어 있는 듯하다.  
이글에서는 요즘 사용중인 Redshift를 기준으로 Convention을 정리한다.  
- Link : [Redshift Reference Convention](https://docs.aws.amazon.com/redshift/latest/dg/c_SQL_reference_conventions.html)


|Character|Description|
|----|----|
|대문자|대문자로 적힌 문자는 key words 이다.|
|[ ]|Brackets(대괄호)는 선택적으로 넣을 수 있는 인자를 뜻한다. 만약 대괄호안에 여러 인수가 있다면 이중에 원하는 것을 선택할 수 있음을 뜻한다. 또한 문서 위에 있는 SELECT와 같이 대괄호가 여러 줄에 거쳐 존재한다면 Parser는 나열된 대괄호문 순으로 query가 구성되어 있을것이라 예상한다는 것을 나타낸다.|
|{ }|Braces(중괄호)는 괄호안에 있는 인수 중 하나를 선택해야함을 나타낸다.|
| \| |파이프는 인수중에서 하나를 선택할 수 있음을 뜻한다.| 
|*italics*|기울여진 단어는 placeholders(아무것도 없을때 그냥 들어가있는 값)를 뜻한다. 규칙에 italics체의 단어가 있다면 이 값 대신 무조건 필요한 내용을 채워 넣어야 한다.|
|...|줄임표는 이전에 나온 element를 반복할 수 있음을 뜻한다.|
|'|작은 따옴표로 묶인 단어는 따옴표를 입력해야 한다는 것을 의미한다.|

<br>


# Expressions

Expression(식)은 Reference 문서에 자주 등장하고는 한다.  
몇가지 예시들이 어떻게 해석되는지에 대해 살펴보자.  

## Compound expressions
- compound expression(복합 표현식)은 일반 표현식끼리 operator로 결합된 표현식이다.
- compound expression에 포함된 일반 표현식은 반드시 numeric value를 return 해야한다.
- compound expression은 괄호를 통해 중첩될 수 있다.

![alt](../../../assets/images/2022-05-17-SQL_reference_conventions/image1.png)

## Expression lists

- 표현식의 여러가지 조합이며 where이나 group by에 흔히 볼 수 있는 방식이다.

```
expression , expression , ... | (expression, expression, ...)
```

```
(1, 5, 10)
('THESE', 'ARE', 'STRINGS')
(('one', 'two', 'three'), ('blue', 'yellow', 'green'))
```

```sql
select * from venue
where (venuecity, venuestate) in (('Miami', 'FL'), ('Tampa', 'FL'))
order by venueid;

venueid |        venuename        | venuecity | venuestate | venueseats
---------+-------------------------+-----------+------------+------------
28 | American Airlines Arena | Miami     | FL         |          0
54 | St. Pete Times Forum    | Tampa     | FL         |          0
91 | Raymond James Stadium   | Tampa     | FL         |      65647
(3 rows)
```

<br>

# Example

위의 내용으로 Window Function 을 살펴보자. 문법을 아래표와 같이 해석할수 있다.


|구문|의미|
|----|----|
|*function* (*expression*) OVER (|*function*에는 함수를 채워 넣어야한다. 소괄호 안에는 그에 맞는 *expression*이 필요하다. 그리고 뒤에는 **OVER**라는 키워드가 뒤따른다.|
|[ PARTITION BY *expr_list* ] | PARTITION BY 구문이 들어갈 수도 있다. 그리고 *expr_list* 를 채워 넣어야한다.|
|[ ORDER BY *order_list* [ *frame_clause* ] ] )|ORDER BY 구문이 들어갈 수도 있다. 위에 대괄호가 이미 존재해서 Parser는 PARTITION BY 다음에 ORDER BY가 올것이라 예측한다. *order_list* 를 채워야 하고 그 안에 *frame_caluse* 를 채워 넣어야 할 수도 있다. |

