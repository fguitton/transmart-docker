FROM debian:jessie
MAINTAINER Denny Verbeeck <dverbeec@its.jnj.com>

RUN apt-get update && apt-get install -y --no-install-recommends curl unzip make
RUN curl http://library.transmartfoundation.org/release/release16_1_0_artifacts/transmart-data-release-16.1.zip -o transmart-data-release-16.1.zip && \
    unzip transmart-data-release-16.1.zip && \
    mv transmart-data-release-16.1 transmart-data && \
    rm transmart-data-release-16.1.zip

WORKDIR /transmart-data

ENV TABLESPACES=dummy

RUN apt-get install -y libcairo-dev gfortran g++ libreadline-dev libxt-dev libpango1.0-dev libprotoc-dev \
    texlive-fonts-recommended tex-gyre liblz4-tool pv
    
RUN make -C R install_packages

RUN apt-get purge -y --auto-remove curl unzip make g++ && \
    rm -rf /var/lib/apt/lists/*
    
RUN sed -i s/deamon/daemon/ /transmart-data/R/Rserv_nodaemon.conf
RUN echo remote enable >> /transmart-data/R/Rserv_nodaemon.conf

VOLUME ["/transmart-data/R"]

EXPOSE 6311

CMD ["/transmart-data/R/root/lib/R/bin/R","CMD","Rserve.dbg","--no-save","--RS-conf","/transmart-data/R/Rserv_nodaemon.conf"]
    