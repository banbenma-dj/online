# This file is part of the LibreOffice project.
#
# This Source Code Form is subject to the terms of the Mozilla Public
# License, v. 2.0. If a copy of the MPL was not distributed with this
# file, You can obtain one at http://mozilla.org/MPL/2.0/.

FROM registry.cn-hangzhou.aliyuncs.com/library/centos:8

# Install LibreOffice run-time dependencies and required tools
# Note: CentOS uses yum instead of apt-get
RUN yum update -y && \
    yum install -y libpng fontconfig shadow-utils cpio \
               findutils nano openssl openssh-clients inotify-tools procps-ng \
               libXrender libXext xcb-util-renderutil xcb-util \
               wqy-zenhei-fonts wqy-microhei-fonts google-noto-cjk-fonts \
               libcap && \
    yum clean all

# Install Poco libraries (may need to adjust versions based on availability)
RUN yum install -y poco-devel poco-foundation poco-net poco-util poco-xml poco-crypto poco-netssl poco-json

# copy freshly built LibreOffice master and LibreOffice Online master with latest translations
COPY /instdir /

# copy the shell script which can start LibreOffice Online (loolwsd)
COPY /scripts/run-lool.sh /

# set up LibreOffice Online
RUN setcap cap_fowner,cap_mknod,cap_sys_chroot=ep /usr/bin/loolforkit && \
    setcap cap_sys_admin=ep /usr/bin/loolmount && \
    useradd --system --home-dir /opt/lool --shell /sbin/nologin lool && \
    mkdir -p /var/cache/loolwsd && chown lool: /var/cache/loolwsd && \
    rm -rf /var/cache/loolwsd/* && \
    rm -rf /opt/lool && \
    mkdir -p /opt/lool/child-roots && \
    loolwsd-systemplate-setup /opt/lool/systemplate /opt/libreoffice >/dev/null 2>&1 && \
    touch /var/log/loolwsd.log && \
    chown lool:lool /var/log/loolwsd.log && \
    chown -R lool:lool /opt/ && \
    chown -R lool:lool /etc/loolwsd

EXPOSE 9980

# switch to lool user (use numeric user id to be compatible with Kubernetes Pod Security Policies)
USER 101

CMD bash /run-lool.sh
