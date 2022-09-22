```mysql
drop procedure if exists numbers_init ;

DELIMITER //

CREATE PROCEDURE numbers_init()
BEGIN

DECLARE idx int ;
set idx = 0 ;

create table if not exists numbers (
`idx` bigint(20) NOT NULL,
 PRIMARY KEY (`idx`)
) ;
truncate table numbers ;

WHILE idx <= 10000 DO
    insert into numbers (idx) values ( idx ) ;
    SET idx = idx + 1;
END WHILE;

END //

DELIMITER ;

# [2022-09-22 14:17:28] 10,001 rows affected in 25 s 767 ms
call numbers_init() ;
drop procedure if exists numbers_init ;

select count(1), min(idx), max(idx) from numbers ;
select * from numbers order by idx desc ;
```