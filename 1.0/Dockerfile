FROM registry.access.redhat.com/ubi8/s2i-core

# This image provides a Common Lisp environment based on QuickLisp and
# that you can use to run your Common Lisp applications.

MAINTAINER The container-lisp hackers @ github.com/container-lisp

EXPOSE 8080

LABEL io.k8s.description="Platform for building and running Common Lisp applications" \
      io.k8s.display-name="Common Lisp" \
      io.openshift.expose-services="8080:http" \
      io.openshift.tags="builder,cl,lisp,sbcl" \
      io.openshift.s2i.scripts-url=image://$STI_SCRIPTS_PATH

# Copy the S2I scripts from the specific language image to $STI_SCRIPTS_PATH
COPY ./s2i/bin/ $STI_SCRIPTS_PATH

ENV HOME=/opt/app-root \
    LC_ALL=C.utf8 \
    LANG=C.utf8 \
    LANGUAGE=C.utf8 \
    LISP_VERSION=1.0 \
    SBCL_VERSION=1.5.4 \
    LIBEV_VERSION=4.27

RUN rm /etc/rhsm-host && \
    yum install -y \
        bzip2 git make patch automake autoconf libtool gcc gcc-c++ libuv && \
    yum clean -y all && \
    locale -a && \
    gcc --version

WORKDIR $HOME
COPY ./root/ /

# Build libev from source since it is not available in ubi8-base nor ubi8-app-stream repos
RUN curl -O "http://dist.schmorp.de/libev/Attic/libev-${LIBEV_VERSION}.tar.gz" && \
    tar xvf libev-${LIBEV_VERSION}.tar.gz && \
    cd libev-${LIBEV_VERSION} && \
    chmod +x autogen.sh && \
    ./autogen.sh && \
    ./configure --prefix=/usr && \
    make && \
    make install && \
    cd "$HOME" && \
    rm -rf libev*

RUN curl -L -O "https://downloads.sourceforge.net/project/sbcl/sbcl/${SBCL_VERSION}/sbcl-${SBCL_VERSION}-x86-64-linux-binary.tar.bz2" && \
    curl -L -O https://beta.quicklisp.org/quicklisp.lisp && \
    tar -xvf sbcl-${SBCL_VERSION}-x86-64-linux-binary.tar.bz2 && \
    cd sbcl-${SBCL_VERSION}-x86-64-linux && \
    ./install.sh && \
    cd .. && \
    rm -rf sbcl-${SBCL_VERSION}-x86-64-linux-binary.tar.bz2 \
       sbcl-${SBCL_VERSION}-x86-64-linux && \
    sbcl --non-interactive --disable-debugger --load install.lisp

RUN usermod -d ${HOME} default && \
    chown -R 1001:0 ${HOME} && \
    chmod -R g+rwX,o= ${HOME}

USER 1001

# Set the default CMD to print the usage of the language image
CMD $STI_SCRIPTS_PATH/usage
