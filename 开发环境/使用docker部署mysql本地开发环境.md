docker run -p 3308:3306 -v /d/dev-enviroment/:/var/lib/mysql -d -e MYSQL_ROOT_PASSWORD=1234 mysql

docker exec -it 01873d bash

mysql -uroot -p1234

GRANT ALL ON *.* TO 'root'@'%';
flush privileges;

ALTER USER 'root'@'localhost' IDENTIFIED BY 'password' PASSWORD EXPIRE NEVER;
ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY '1234';
flush privileges;