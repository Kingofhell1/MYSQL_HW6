Основное ДЗ:
Создайте процедуру ИЛИ функцию, которая принимает кол-во сек и формат их в кол-во дней, часов, минут и секунд.
Пример: 123456 ->'1 days 10 hours 17 minutes 36 seconds '


    mysql> DELIMITER //
    mysql>
    mysql> CREATE PROCEDURE format_seconds(seconds INT)
     -> BEGIN
     ->     DECLARE days INT;
     ->     DECLARE hours INT;
     ->     DECLARE minutes INT;
     ->
    ->     SET days = seconds DIV (24*60*60);
    ->     SET seconds = seconds MOD (24*60*60);
    ->     SET hours = seconds DIV (60*60);
    ->     SET seconds = seconds MOD (60*60);
    ->     SET minutes = seconds DIV 60;
    ->     SET seconds = seconds MOD 60;
    ->
    ->     SELECT CONCAT(days, ' days ', hours, ' hours ', minutes, ' minutes ', seconds, ' seconds');
    -> END //
Query OK, 0 rows affected (0.02 sec)

    mysql>
     mysql> DELIMITER ;
     mysql> CALL format_seconds(123456);
    +-------------------------------------------------------------------------------------+
    | CONCAT(days, ' days ', hours, ' hours ', minutes, ' minutes ', seconds, ' seconds') |
    +-------------------------------------------------------------------------------------+
    | 1 days 10 hours 17 minutes 36 seconds                                               |
    +-------------------------------------------------------------------------------------+
1 row in set (0.00 sec)

--------------------------------------------------------------------------------



Выведите только четные числа от 1 до 10 (Через цикл).
Пример: 2,4,6,8,10
      
    mysql> DELIMITER //
     mysql> CREATE PROCEDURE even_numbers (IN count_number int, OUT result varchar(255))
    -> BEGIN
    ->   DECLARE i int DEFAULT 2;
    ->
    ->   SET result = '';
    ->
    ->   WHILE i < count_number DO
    ->     SET result = CONCAT(result, CAST(i AS char), ' ');
    ->     SET i = i + 2;
    ->   END WHILE;
    -> END //
Query OK, 0 rows affected (0.01 sec)

mysql> DELIMITER ;
mysql> SET @even = '';
Query OK, 0 rows affected (0.00 sec)


1 row in set (0.00 sec)

mysql> CALL even_numbers(10, @even);
Query OK, 0 rows affected (0.00 sec)

    mysql> SELECT @even;
    +----------+
    | @even    |
    +----------+
    | 2 4 6 8  |
    +----------+

---------------------------------------



#### Создать процедуру, которая решает следующую задачу
#### Выбрать для одного пользователя 5 #### пользователей в случайной комбинации, которые удовлетворяют хотя бы одному критерию:
* а) из одного города
* б) состоят в одной группе
* в) друзья друзей




  
*    mysql> CREATE PROCEDURE users_5
    -> (/n
    
    ->  IN id_user_find INT
    -> )
    -> BEGIN
    ->
    ->     SELECT t.id
    ->     FROM
    ->     (
    ->          SELECT id
    ->          FROM users u
    ->          INNER JOIN profiles p
    ->          ON u.id = p.user_id
    ->          AND u.id <> id_user_find
    ->          AND (
    ->                  SELECT p1.hometown
    ->                  FROM users u1
    ->                  INNER JOIN profiles p1
    ->                  ON u1.id = p1.user_id
    ->                  AND u1.id = id_user_find
    ->          ) = p.hometown
    ->          UNION
    ->          SELECT DISTINCT u.id
    ->          FROM users u
    ->          INNER JOIN users_communities uc
    ->          ON u.id = uc.user_id
    ->          WHERE uc.community_id IN
    ->          (
    ->                  SELECT community_id
    ->                  FROM users_communities
    ->                  WHERE users_communities.user_id = id_user_find
    ->          )
    ->          UNION
    ->          SELECT id
    ->          FROM users
    ->          WHERE users.id IN (
    ->                  (
    ->                          SELECT initiator_user_id AS id
    ->                          FROM friend_requests
    ->                          WHERE status='approved'
    ->                          AND target_user_id IN (
    ->                                  SELECT initiator_user_id AS id
    ->                                  FROM friend_requests
    ->                                  WHERE target_user_id = id_user_find AND status='approved'
    ->                                  UNION ALL
    ->                                  SELECT target_user_id
    ->                                  FROM friend_requests
    ->                                  WHERE initiator_user_id = id_user_find AND status='approved'
    ->                          )
    ->                          UNION
    ->                          SELECT target_user_id
    ->                          FROM friend_requests
    ->                          WHERE status='approved'
    ->                          AND initiator_user_id IN (
    ->                                  SELECT initiator_user_id AS id
    ->                                  FROM friend_requests
    ->                                  WHERE target_user_id = id_user_find AND status='approved'
    ->                                  UNION ALL
    ->                                  SELECT target_user_id
    ->                                  FROM friend_requests
    ->                                  WHERE initiator_user_id = id_user_find AND status='approved'
    ->                          )
    ->                  )
    ->          )
    ->  ) t
    ->     ORDER BY RAND()
    ->     LIMIT 5;
    ->
    -> END //
Query OK, 0 rows affected (0.02 sec)

    mysql> DELIMITER ;
    mysql> CALL users_5(4);
        +----+
        | id |
        +----+
        |  1 |
        |  9 |
        |  4 |
        | 10 |
        |  2 |
        +----+
5 rows in set (0.00 sec)

Query OK, 0 rows affected (0.01 sec)

     mysql> CALL users_5(6);
        +----+
        | id |
        +----+
        |  9 |
        |  8 |
        | 10 |
        |  7 |
        |  6 |
        +----+
5 rows in set (0.00 sec)

Query OK, 0 rows affected (0.01 sec)

-------------------------------------------------------------------------------

Создать функцию, вычисляющей коэффициент популярности пользователя (по количеству друзей)

    mysql> DELIMITER //
    mysql> CREATE FUNCTION get_popularity_coefficient(
    ->  user_id INT
    -> )
    -> RETURNS INT DETERMINISTIC
    -> BEGIN
    ->  DECLARE result INT DEFAULT 0;
    ->  SELECT
    ->          (
    ->                  SELECT count(f.id)
    ->                  FROM (
    ->                          SELECT fr1.initiator_user_id AS id
    ->                          FROM friend_requests fr1
    ->                          WHERE fr1.target_user_id = u.id AND fr1.status='approved'
    ->                          UNION
    ->                          SELECT fr2.target_user_id
    ->                          FROM friend_requests fr2
    ->                          WHERE fr2.initiator_user_id = u.id AND fr2.status='approved'
    ->                  ) f
    ->          ) AS `count_friends` INTO result
    ->  FROM users u
    ->     WHERE u.id = user_id;
    ->
    ->     RETURN result;
    -> END //
Query OK, 0 rows affected (0.02 sec)

    mysql> DELIMITER ;
    mysql> SELECT get_popularity_coefficient(1);
    +-------------------------------+
    | get_popularity_coefficient(1) |
    +-------------------------------+
    |                             3 |
    +-------------------------------+
1 row in set (0.00 sec)

    mysql> SELECT get_popularity_coefficient(1), get_popularity_coefficient(5), get_popularity_coefficient(20);
    +-------------------------------+-------------------------------+--------------------------------+
    | get_popularity_coefficient(1) | get_popularity_coefficient(5) | get_popularity_coefficient(20) |
    +-------------------------------+-------------------------------+--------------------------------+
    |                             3 |                             1 |                              0 |
    +-------------------------------+-------------------------------+--------------------------------+
1 row in set (0.00 sec)



