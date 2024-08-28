ARG OS_BASE_IMAGE_NAME=sles
ARG OS_BASE_IMAGE_REPO=armdocker.rnd.ericsson.se/proj-ldc/common_base_os_release
ARG OS_BASE_IMAGE_TAG

ARG CBOS_REPO_URL=arm.sero.gic.ericsson.se/artifactory/proj-ldc-repo-rpm-local/common_base_os/sles/${OS_BASE_IMAGE_TAG}
ARG CBOS_DEVEL_REPO_URL=arm.sero.gic.ericsson.se/artifactory/proj-ldc-repo-rpm-local/common_base_os/sles/${OS_BASE_IMAGE_TAG}_devel
ARG ADP_DEV_REPO_URL=arm.sero.gic.ericsson.se/artifactory/proj-ldc-repo-rpm-local/adp-dev/adp-build-env/latest

FROM ${OS_BASE_IMAGE_REPO}/${OS_BASE_IMAGE_NAME}:${OS_BASE_IMAGE_TAG} as builder

ARG CBOS_REPO_URL
ARG CBOS_DEVEL_REPO_URL
ARG ADP_DEV_REPO_URL

##
RUN zypper addrepo -p 50 -C -G -f https://${CBOS_REPO_URL} repo-oss-cbos && \
    zypper addrepo -p 50 -C -G -f https://${CBOS_DEVEL_REPO_URL} repo-oss-cbosdevel && \
    zypper addrepo -p 50 -C -G -f https://${ADP_DEV_REPO_URL} repo-oss-adpdev &&\
    zypper --gpg-auto-import-keys refresh -f -r repo-oss-cbos

#

RUN zypper install -y wget curl
#
COPY . /

#
RUN chmod 755 /build.sh
RUN /build.sh
## Use a multi-stage build

FROM ${OS_BASE_IMAGE_REPO}/${OS_BASE_IMAGE_NAME}:${OS_BASE_IMAGE_TAG}

ARG CBOS_REPO_URL

RUN zypper addrepo -p 50 -C -G -f https://${CBOS_REPO_URL} repo-cbos

##
ENV PATH=$PATH:/usr/local/luajit/bin:/usr/local/nginx/sbin:/usr/local/nginx/bin
##
ENV LUA_PATH="/usr/local/share/luajit-2.1.0-beta3/?.lua;/usr/local/share/lua/5.1/?.lua;/usr/local/lib/lua/?.lua;;"
ENV LUA_CPATH="/usr/local/lib/lua/?/?.so;/usr/local/lib/lua/?.so;;"
##
COPY --from=builder /usr/local /usr/local
COPY --from=builder /opt /opt
COPY --from=builder /etc/nginx /etc/nginx

##
RUN zypper refresh
##    bash \
RUN  zypper install -y  libyajl2 liblmdb-0_9_30 catatonit pcre zlib ca-certificates patch libxml2 shadow glibc

RUN   ln -s /usr/local/nginx/sbin/nginx /sbin/nginx

RUN groupadd -rg 101 www-data

RUN  useradd www-data -r -u 101 -d /usr/local/nginx -g101

RUN  writeDirs=( \
    /var/log/nginx \
    /var/lib/nginx/body \
    /var/lib/nginx/fastcgi \
    /var/lib/nginx/proxy \
    /var/lib/nginx/scgi \
    /var/lib/nginx/uwsgi \
    /var/log/audit \
  ); \
  for dir in "${writeDirs[@]}"; do \
    mkdir -p ${dir}; \
    chown -R www-data.www-data ${dir}; \
  done


CMD ["nginx", "-g", "daemon off;"]
