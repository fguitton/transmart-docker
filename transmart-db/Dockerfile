FROM debian:jessie
MAINTAINER Denny Verbeeck <dverbeec@its.jnj.com>

RUN apt-get update && apt-get install -y --no-install-recommends curl unzip make
RUN curl http://library.transmartfoundation.org/release/release16_1_0_artifacts/transmart-data-release-16.1.zip -o transmart-data-release-16.1.zip && \
    unzip transmart-data-release-16.1.zip && \
    mv transmart-data-release-16.1 transmart-data && \
    rm transmart-data-release-16.1.zip

#### PostgreSQL Setup
# Add the PostgreSQL PGP key to verify their Debian packages.
# It should be the same key as https://www.postgresql.org/media/keys/ACCC4CF8.asc
RUN apt-get install -y locales && rm -rf /var/lib/apt/lists/* \
	&& localedef -i en_US -c -f UTF-8 -A /usr/share/locale/locale.alias en_US.UTF-8
ENV LANG en_US.utf8

RUN apt-key adv --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys B97B0AFCAA1A47F044F244A07FCC7D46ACCC4CF8

# Add PostgreSQL's repository. It contains the most recent stable release
#     of PostgreSQL, ``9.3``.
RUN echo "deb http://apt.postgresql.org/pub/repos/apt/ precise-pgdg main" > /etc/apt/sources.list.d/pgdg.list

# Install PostgreSQL 9.3
RUN apt-get update && apt-get install -y postgresql-9.3 postgresql-client-9.3 postgresql-contrib-9.3

# Note: The official Debian and Ubuntu images automatically ``apt-get clean``
# after each ``apt-get``

# Run the rest of the commands as the ``postgres`` user created by the ``postgres-9.3`` package when it was ``apt-get installed``
USER postgres

# Adjust PostgreSQL configuration so that remote connections to the
# database are possible.
RUN echo "host all  all    0.0.0.0/0  md5" >> /etc/postgresql/9.3/main/pg_hba.conf

# And add ``listen_addresses`` to ``/etc/postgresql/9.3/main/postgresql.conf``
RUN echo "listen_addresses='*'" >> /etc/postgresql/9.3/main/postgresql.conf

# Create a PostgreSQL role named ``docker`` with ``docker`` as the password
RUN /etc/init.d/postgresql start && \
    psql --command "CREATE USER docker WITH SUPERUSER PASSWORD 'docker';" && \
	/etc/init.d/postgresql stop

#### Load transmart db

USER root

RUN apt-get install -y --no-install-recommends php5-cli php5-json sudo

RUN cd transmart-data && make -C env /var/lib/postgresql/tablespaces ../vars && \
    . ./vars && \
	/etc/init.d/postgresql start && \
    make postgres && \
    /etc/init.d/postgresql stop
    
RUN rm -fR transmart-data-release-16.1.zip transmart-data && \
    apt-get purge -y --auto-remove curl unzip make php5-cli php5-json

VOLUME ["/var/lib/postgresql/tablespaces", "/var/log/postgresql", "/var/lib/postgresql"]

EXPOSE 5432

USER postgres

# Increase the checkpoint_segments parameter, this increases performance of large data loads like expression values
RUN echo "checkpoint_segments = 30" >> /etc/postgresql/9.3/main/postgresql.conf

CMD ["/usr/lib/postgresql/9.3/bin/postgres", "-D", "/var/lib/postgresql/9.3/main", "-c", "config_file=/etc/postgresql/9.3/main/postgresql.conf"]
