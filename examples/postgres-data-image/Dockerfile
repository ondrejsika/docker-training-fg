FROM mysql:5.6
RUN cp -r /var/lib/mysql /mysql
RUN sed -i 's/\/var\/lib\/mysql/\/mysql/' /etc/mysql/mysql.conf.d/mysqld.cnf