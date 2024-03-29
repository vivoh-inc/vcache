FROM debian:bullseye-slim

ARG  PKG_COMMIT=ffc59a345217b599fd49f7f0442b5f653fbe6fc2
ARG  VARNISH_VERSION=7.2.1
ARG  DIST_SHA512=7b9b837a8bafdf5798e81bc38163457b3bca16d933a9492800cdd2cde35c9b524a10b7e5ec931217e11d72f32feb05157a7eecfd9cf2c5856e717b634e51d089
ARG  VARNISH_MODULES_VERSION=0.21.0
ARG  VARNISH_MODULES_SHA512SUM=a442f58968b471d713c99a94e5b80302c07ea163d3d5022d768eb0b39ab081f18744fd529b04283b0c6ec942f362197935d8ef1aa04f26eff10a81425a63bd35
ARG  VMOD_DYNAMIC_VERSION=2.8.0
ARG  VMOD_DYNAMIC_COMMIT=5c702fa6c3a88882a2678f75161692762e7d6c47
ARG  VMOD_DYNAMIC_SHA512SUM=3503ae09bae731213d5a6823af9fb758bcbcaf06678a2a0efc0b35d9f1b18ab46e02f02b75db8a4858bb2b623e76ea253e65ef2ae3ab076558b52b414996d33a
ARG  TOOLBOX_COMMIT=96bab07cf58b6e04824ffec608199f1780ff0d04
ENV  VMOD_DEPS="automake curl libtool make pkg-config python3-sphinx"

ENV VARNISH_SIZE 100M

RUN set -e; \
    BASE_PKGS="curl dpkg-dev debhelper devscripts equivs git pkg-config apt-utils fakeroot libgetdns-dev libssl-dev"; \
    export DEBIAN_FRONTEND=noninteractive; \
    export DEBCONF_NONINTERACTIVE_SEEN=true; \
    mkdir -p /work/varnish /pkgs; \
    apt-get update; \
    apt-get install -y $BASE_PKGS libgetdns10 libmhash-dev; \
    # varnish
    cd /work/varnish; \
    git clone https://github.com/varnishcache/pkg-varnish-cache.git; \
    cd pkg-varnish-cache; \
    git checkout ffc59a345217b599fd49f7f0442b5f653fbe6fc2; \
    rm -rf .git; \
    curl -f https://varnish-cache.org/downloads/varnish-7.2.1.tgz -o $tmpdir/orig.tgz; \
    echo "7b9b837a8bafdf5798e81bc38163457b3bca16d933a9492800cdd2cde35c9b524a10b7e5ec931217e11d72f32feb05157a7eecfd9cf2c5856e717b634e51d089  $tmpdir/orig.tgz" | sha512sum -c -; \
    tar xavf $tmpdir/orig.tgz --strip 1; \
    sed -i -e "s|@VERSION@|$VARNISH_VERSION|"  "debian/changelog"; \
    mk-build-deps --install --tool="apt-get -o Debug::pkgProblemResolver=yes --yes" debian/control; \
    sed -i '' debian/varnish*; \
    dpkg-buildpackage -us -uc -j"$(nproc)"; \
    apt-get -y install ../*.deb; \
    mv ../*dev*.deb /pkgs; \
    \
    git clone https://github.com/varnish/toolbox.git; \
    cd toolbox; \
    git checkout $TOOLBOX_COMMIT; \
    cp install-vmod/install-vmod /usr/local/bin/; \
    \
    # varnish-modules
    install-vmod https://github.com/varnish/varnish-modules/releases/download/$VARNISH_MODULES_VERSION/varnish-modules-$VARNISH_MODULES_VERSION.tar.gz $VARNISH_MODULES_SHA512SUM; \
    \
    # vmod-dynamic
    # install-vmod https://github.com/nigoroll/libvmod-dynamic/archive/$VMOD_DYNAMIC_COMMIT.tar.gz $VMOD_DYNAMIC_SHA512SUM; \
    # \
	cd ..; \
	git clone https://code.uplex.de/uplex-varnish/libvmod-crypto.git; \
	cd libvmod-crypto; \
	git checkout fc9291afa67dc5f1baab0c4e62069dd56a1ae09f; \
	./bootstrap; \
	make; \
	make install; \
	cd ..; \
	git clone https://github.com/varnish/libvmod-digest.git; \
	cd libvmod-digest; \
	./autogen.sh; \
	./configure; \
	make; \
	make install; \
	cd ..; \
	git clone https://code.uplex.de/uplex-varnish/libvmod-re.git; \
	cd libvmod-re; \
	git checkout 2c683cd46e7ad025b84713749d43b1adffabe7af; \
	./autogen.sh; \
	./configure; \
	make; \
	make install; \
    # clean up
    apt-get -y purge --auto-remove varnish-build-deps $BASE_PKGS; \
    # add python run environment
	apt-get install -y python3 python3-pip python3-venv expect; \
    rm -rf /var/lib/apt/lists/* /work/ /usr/lib/varnish/vmods/libvmod_*.la; \
    chown varnish /var/lib/varnish;

COPY ./etc/varnish /etc/varnish
VOLUME /etc/varnish

COPY ./etc/varnishstats /etc/varnishstats

COPY ./scripts/ /usr/local/bin/
ENTRYPOINT ["/usr/local/bin/docker-varnish-entrypoint"]


EXPOSE 80 8443 9131 6082
WORKDIR /etc/varnish
CMD []
