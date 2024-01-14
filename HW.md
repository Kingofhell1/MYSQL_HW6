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




  
*     mysql> DELIMITER //
      mysql> CREATE PROCEDURE find_person(IN find_id int)
        -> BEGIN
        ->   SELECT b.user_id, u.firstname, u.lastname
        ->   FROM
        ->     (SELECT user_id FROM
        ->       (SELECT p.user_id
        ->         FROM profiles p
        ->         WHERE p.hometown = (SELECT hometown FROM profiles WHERE PROFILES.user_id = find_id)
        ->       UNION
        ->       SELECT id
        ->         FROM communities
        ->         WHERE id IN (SELECT community_id FROM users_communities WHERE user_id = find_id)) a
        ->     UNION
        ->     SELECT target_user_id
        ->       FROM friend_requests
        ->       WHERE initiator_user_id IN (
        ->         SELECT
        ->            target_user_id
        ->          FROM friend_requests
        ->          WHERE initiator_user_id = find_id)) b
        ->   LEFT JOIN users u
        ->     ON b.user_id = u.id
        ->   ORDER BY RAND()
        ->   LIMIT 5;
        -> END //
------------------------------------------------------
      mysql> CALL find_person(3);
            +---------+-----------+----------+
            | user_id | firstname | lastname |
            +---------+-----------+----------+
            |       3 | Unique    | Windler  |
            |       2 | Frederik  | Upton    |
            |       5 | Frederick | Effertz  |
            |       8 | Jaida     | Kilback  |
            |      10 | Jordyn    | Jerde    |
            +---------+-----------+----------+
5 rows in set (0.04 sec)


Query OK, 0 rows affected (0.01 sec)

     mysql> CALL find_person(5);
    +---------+-----------+----------+
    | user_id | firstname | lastname |
    +---------+-----------+----------+
    |       1 | Reuben    | Nienow   |
    |       2 | Frederik  | Upton    |
    |       6 | Victoria  | Medhurst |
    |      10 | Jordyn    | Jerde    |
    |       3 | Unique    | Windler  |
    +---------+-----------+----------+

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



