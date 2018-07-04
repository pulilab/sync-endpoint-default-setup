FROM osixia/openldap:1.2.1

ARG BUILD_ENV
ENV BUILD_ENV=${BUILD_ENV:-"prod"}

ENV PREFIX=/opt
ENV OPENLDAP_INSTALL_DIR=${PREFIX}/ldap
ENV OPENLDAP_VERSION=2.4.46

# Temporary install deps
RUN apt-get update
RUN apt-get install -y wget build-essential git libtool libdb-dev groff groff-base openssl libssl-dev nano

# Build tmp OpenLDAP
RUN mkdir -p ${PREFIX}
WORKDIR ${PREFIX}
RUN wget ftp://ftp.openldap.org/pub/OpenLDAP/openldap-release/openldap-${OPENLDAP_VERSION}.tgz
RUN tar xzf openldap-${OPENLDAP_VERSION}.tgz
RUN mv openldap-${OPENLDAP_VERSION} ldap
WORKDIR ${OPENLDAP_INSTALL_DIR}
RUN env CPPFLAGS="-I/usr/include/openssl" LDFLAGS="-L/usr/lib/ssl" ./configure --prefix=${PREFIX} --enable-modules --with-tls=openssl
RUN make depend
RUN make
RUN make install

# Build pbkdf2 OpenLDAP
RUN mkdir -p ${OPENLDAP_INSTALL_DIR}/contrib/slapd-modules/passwd
WORKDIR ${OPENLDAP_INSTALL_DIR}/contrib/slapd-modules/passwd
RUN rm -rf pbkdf2
RUN git clone https://github.com/nullcode/openldap-pbkdf2 pbkdf2
WORKDIR ${OPENLDAP_INSTALL_DIR}/contrib/slapd-modules/passwd/pbkdf2
RUN make
RUN make install

# install pbkdf2 lib
RUN cp /opt/libexec/openldap/pw-pbkdf2.so /usr/lib/ldap/pw-pbkdf2.so

# Remove tmp OpenLDAP build
RUN rm -rf ${OPENLDAP_INSTALL_DIR}
RUN rm -rf /usr/local/libexec

# Remove installed deps
RUN apt-get remove -y wget build-essential git libtool libdb-dev groff groff-base
RUN apt-get autoclean && \
  rm -rf /var/lib/apt/lists/* && \
  rm -rf /usr/share/locale/* && \
  rm -rf /usr/share/man/* && \
  rm -rf /usr/share/doc/*

# copy ldif files into the container
COPY bootstrap.ldif /container/service/slapd/assets/config/bootstrap/ldif/custom/bootstrap.ldif
COPY changerootPw.ldif /container/service/slapd/assets/changerootPw.ldif
COPY changePw.ldif /container/service/slapd/assets/changePw.ldif
COPY loadHasher.ldif /container/service/slapd/assets/loadHasher.ldif
COPY userAccount.ldif /container/service/slapd/assets/userAccount.ldif

WORKDIR /
