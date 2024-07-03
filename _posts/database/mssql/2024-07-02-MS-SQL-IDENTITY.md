---
layout: post
related_posts: 
title: "MS-SQL의 자동 증가 값(IDENTITY)설정"
subtitle: "MS-SQL에서의 AUTO INCREMENT와 같은 기능은 무엇인가?"
date: 2024-07-02 19:27:00 +0900
categories: [database, mssql]
tags:
  - DATABASE
  - MS-SQL
description: >
  'MS-SQL의 IDENTITY에 대하여 알아보자
---
* toc
{:toc}

## MS-SQL의 자동 증가 값(IDENTITY)이란?

#### 0. 참조
 [MSSQL 자동 증가 값(IDENTITY) 설정 방법](https://travelpark.tistory.com/50)<br>
 [기존 칼럼을 IDENTITY 속성으로 변경하기](https://chozzahacker.blogspot.com/2014/05/alteridentity.html)

#### 1. 서론
-  MSSQL에서는 AUTO INCREMENT가 아니라 IDENTITY란 용어를 사용합니다.
-  IDENTITY는 자동 증가 값을 의미합니다.

#### 2. IDENTITY 지정
-  DDL을 통해 IDENTITY를 지정하는 방법은 테이블을 생성 할 때, 함께 선언을 해야 합니다.
~~~sql
//file: 'test.sql'

CREATE TABLE SAMPLE1
(
	  [S_IDX] [bigint] IDENTITY(1,1) NOT NULL,
)
~~~

*  이미 테이블이 생성되어 있고, 해당 테이블의 특정 칼럼에 IDENTITY를 지정하는 방법은 SSMS를 이용하는 수 밖에 없습니다. 
```
※ 번외) SSMS를 이용하여 IDENTITY를 지정하는 로직
1) 임시로 TMP 테이블을 생성
2) 추가하는 TABLE의 데이터를 TMP 테이블로 이동
3) 기존의 테이블을 DROP
4) 임시로 만든 TMP 테이블의 이름을 수정
```
- SSMS를 통해 IDENTITY를 지정하는 로직은 다음과 같습니다. 
~~~sql
//file: 'test.sql'

CREATE TABLE DBO.TMP_TBL1(
  COL1 INT NOT NULL IDENTITY (1, 1),
  COL2 DATETIME NULL
)
ON [PRIMARY]
GO

ALTER TABLE DBO.TMP_TBL1 SET (LOCK_ESCALATION = TABLE)
GO

SET IDENTITY_INSERT DBO.TMP_TBL1 ON
GO

IF EXISTS(SELECT * FROM DBO.TBL1)
        EXEC('INSERT INTO DBO.TMP_TBL1 (COL1, COL2)
        SELECT COL1, COL2 FROM DBO.TBL1 WITH (HOLDLOCK TABLOCKX)')
GO

SET IDENTITY_INSERT DBO.TMP_TBL1 OFF
GO

DROP TABLE DBO.TBL1
GO

EXECUTE SP_RENAME N'DBO.TMP_TBL1', N'TBL1', 'OBJECT';
GO
~~~

#### 3. IDENTITY를 임의로 수정 
-  IDENTITY를 수동으로 값을 수정하는 방법은 다음과 같습니다. 

~~~sql
//file: 'test.sql'

-- 증가값을 수동으로 지정할 수 있도록 수정
SET IDENTITY_INSERT [테이블명] ON;

-- 원하는 값 입력
INSERT INTO IDENT('[테이블명]') VALUES (100)

-- 증가값을 자동 지정으로 세팅 변경
SET IDENTITY_INSERT [테이블명] OFF;
~~~

#### 4. IDENTITY를 원하는 값으로 세팅
-  DELETE 문을 이용해 삭제를 하더라도 IDENTITY는 다시 그 값을 이용할 수 없습니다.
-  아래의 내용과 같이 현재의 INDEX 값을 확인 하고 원하는 값으로 세팅할 수 있습니다. 

~~~sql
//file: 'test.sql'

-- 현재 id값 확인
SELECT IDENT_CURRENT('[테이블명]')  

-- 증가값을 수동으로 지정할 수 있도록 수정
SET IDENTITY_INSERT [테이블명] ON;

-- 원하는 값으로 초기값 세팅
DBCC CHECKIDENT('[테이블명]', RESEED, 초기값)
ex) DBCC CHECKIDENT('[데이터베이스명].[dbo].[테이블명]', RESEED, 0)

-- 증가값 자동 지정으로 세팅 변경
SET IDENTITY_INSERT [테이블명] OFF;
~~~

*  모든 데이터를 삭제 후, 아예 초기화를 시키고 싶을 때는, 위의 내용과 같이 DBCC CHECKIDENT('\[데이터베이스명].\[dbo].\[테이블명]', RESEED, 0); 이런 식으로 초기값을 0으로 설정하면 됩니다.

해당 내용을 통해 MS-SQL에서 AUTO-INCREMENT와 같은 기능인 IDENTITY를 알아 보았습니다.
하지만 참조한 블로그의 내용과 같이 오픈 전 테스트 이거나, 오픈을 위한 DB 세팅을 제외하면 쓰지 않는 것이 좋을 것 같습니다.
