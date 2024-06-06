ARG BASE_IMAGE="eosphorosai/dbgpt:latest"

FROM ${BASE_IMAGE}

RUN pip install dashscope
RUN apt-get update && apt-get install -y wget gnupg lsb-release net-tools

# Add MySQL APT repository
RUN wget https://dev.mysql.com/get/mysql-apt-config_0.8.17-1_all.deb && \
    dpkg -i mysql-apt-config_0.8.17-1_all.deb && \
    apt-get clean

# Configure MySQL APT repository for Ubuntu
RUN echo "deb http://repo.mysql.com/apt/ubuntu/ jammy mysql-5.7" | tee /etc/apt/sources.list.d/mysql.list && \
    mkdir -p /etc/apt/keyrings && \
    wget -qO - https://repo.mysql.com/RPM-GPG-KEY-mysql | gpg --dearmor -o /etc/apt/keyrings/mysql.gpg && \
    echo "deb [signed-by=/etc/apt/keyrings/mysql.gpg] http://repo.mysql.com/apt/ubuntu/ jammy mysql-apt-config" | tee /etc/apt/sources.list.d/mysql.list && \
    apt-get update

# Install MySQL server
RUN apt-get update && apt-get install -y mysql-server && apt-get clean

# Remote access
RUN sed -i 's/bind-address\s*=\s*127.0.0.1/bind-address = 0.0.0.0/g' /etc/mysql/mysql.conf.d/mysqld.cnf && \
    echo "[mysqld]\ncharacter_set_server=utf8mb4\ncollation-server=utf8mb4_unicode_ci\ninit_connect='SET NAMES utf8mb4'\n[mysql]\ndefault-character-set=utf8mb4\n[client]\ndefault-character-set=utf8mb4\n" >> /etc/mysql/my.cnf

# Init sql
RUN mkdir /docker-entrypoint-initdb.d && \
    echo "USE mysql;\nUPDATE user SET Host='%' WHERE User='root';\nFLUSH PRIVILEGES;" > /docker-entrypoint-initdb.d/init.sql

ENV MYSQL_ROOT_PASSWORD=aa123456
ENV LOCAL_DB_PASSWORD="$MYSQL_ROOT_PASSWORD"

RUN cp /app/assets/schema/dbgpt.sql /docker-entrypoint-initdb.d/

COPY docker/allinone/allinone-entrypoint.sh /usr/local/bin/allinone-entrypoint.sh
COPY docker/examples/sqls/*_mysql.sql /docker-entrypoint-initdb.d/

ENTRYPOINT ["/usr/local/bin/allinone-entrypoint.sh"]