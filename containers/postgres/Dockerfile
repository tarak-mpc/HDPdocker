FROM postgres
RUN apt-get update -y
RUN apt-get install -y wget
ARG DDL_URL
ADD pg_hba.conf /pg_hba.conf
ADD scripts/* /docker-entrypoint-initdb.d/
RUN mkdir /home/postgres
RUN wget ${DDL_URL} -P /home/postgres
RUN chown postgres /pg_hba.conf /home/postgres
RUN chown postgres /var/lib/postgresql/data
