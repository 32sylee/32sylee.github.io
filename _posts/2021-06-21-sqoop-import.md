---
layout: post
title: "sqoop import 옵션 정리"
author: "Sooyeon"
---
# [Large Object](https://sqoop.apache.org/docs/1.4.6/SqoopUserGuide.html#_large_objects)
- --fields-terminated-by \<char>
  - 필드 구분자를 설정
- --lines-terminated-by \<char>
  - end-of-line 문자 설정
- --enclosed-by \<char>
  - 필드를 해당 문자로 감싼다
- --escaped-by \<char>
  - `--fields-terminated-by`, `--lines-terminated-by`, `--enclosed-by`에 사용되는 구분자가 필드 내용에 있는 경우, 구분이 모호해질 수 있으므로 `--escaped-by`에 오는 문자열로 escape한다고 생각했으나 아닐수도 있음🙄
- --optionally-enclosed-by \<char>
  - `--fields-terminated-by`는 모든 문자열을 enclose함. `--optionally-enclosed-by`를 사용하면 구분자가 존재하는 필드만 enclose한다.

```
Some string, with a comma. | 1 | 2 | 3
Another "string with quotes" | 4 | 5 | 6
```
테이블에 위와 같이 데이터가 있을 때,
```
$ sqoop import --fields-terminated-by , --escaped-by \\ --enclosed-by '\"'
```
를 했을 때 결과는 다음과 같다.
```
"Some string, with a comma.","1","2","3"
"Another \"string with quotes\"","4","5","6"
```
`--fields-terminated-by` 대신 `--optionally-enclosed-by`를 사용하면
```
"Some string, with a comma.",1,2,3
"Another \"string with quotes\"",4,5,6
```
필드 구분자인 ,가 존재하는 첫번째 필드에 "로 enclose된다.

그런데.. 다음과 같은 Note가 있음.
> Even though Hive supports escaping characters, it does not handle escaping of new-line character.

Hive에서는 escaping character를 지원하지만, \n는 이스케이프 처리를 안한다고 함.

Hive에 데이터를 import할 때 \n, \r, \01를 없애주는 옵션도 존재한다.
- --hive-drop-import-delims
  - Hive로 import할 때 string 필드의 \n, \r, \01를 drop
- --hive-delims-replacement \<char>
  - - Hive로 import할 때 string 필드의 \n, \r, \01를 지정된 문자로 치환한다.