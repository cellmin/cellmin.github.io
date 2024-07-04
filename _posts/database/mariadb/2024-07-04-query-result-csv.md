---
layout: post
related_posts: 
title: MariaDB SELECT 쿼리 결과로 CSV 파일 생성
subtitle: MariaDB SELECT 쿼리를 CSV로 저장하기
date: 2024-07-04 11:33:00 +0900
categories: [database, mariadb]
tags: 
  - database
  - mariadb
description: >
  'SELECT 쿼리의 결과를 파일로 저장하는 방법을 알아보았습니다.
---
* toc
{:toc}

## MariaDB SELECT 쿼리 결과로 CSV 파일 생성
### 01. CSV 파일 생성 쿼리 입니다.
  -  실제로 사용하게 된다면 파일 이름에 날짜를 추가하는 등 쿼리가 좀 더 길어지겠지만 기본 내용은 아래의 내용일 것 같습니다.

~~~sql
SELECT *
  INTO OUTFILE '저장경로' -- ex) 'C:\\BACKUP\\sample1_backup.csv'
       CHARACTER SET UTF8 -- (캐릭터 셋 설정)
       FIELDS TERMINATED BY ',' OPTIONALLY ENCLOSED BY '"'
       ESCAPED BY '\\'
       LINES TERMINATED BY '\n'
  FROM sample1 t1
 WHERE 1 = 1
 /* 원하는 조건들을 추가 */
 ORDER BY t1.idx -- 원하는 순서 추가
~~~
