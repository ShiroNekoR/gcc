FROM buildpack-deps:jessie

# https://gcc.gnu.org/mirrors.html
ENV GPG_KEYS \
	B215C1633BCA0477615F1B35A5B3A004745C015A \
	B3C42148A44E6983B3E4CC0793FA9B1AB75C61B8 \
	90AA470469D3965A87A5DCB494D03953902C9419 \
	80F98B2E0DAB6C8281BDF541A7C8C3B2F71EDF1C \
	7F74F97C103468EE5D750B583AB00996FC26A641 \
	33C235A34C46AA3FFB293709A328C3A2C3C45C06
RUN set -xe \
	&& for key in $GPG_KEYS; do \
		gpg --keyserver ha.pool.sks-keyservers.net --recv-keys "$key"; \
	done

# Last Modified: 2016-08-22
ENV GCC_VERSION 6.2.0
# Docker EOL: 2017-08-22

# "download_prerequisites" pulls down a bunch of tarballs and extracts them,
# but then leaves the tarballs themselves lying around
RUN buildDeps='flex' \
	&& set -x \
	&& apt-get update && apt-get install -y $buildDeps --no-install-recommends \
	&& rm -r /var/lib/apt/lists/* \
	&& curl -fSL "http://ftpmirror.gnu.org/gcc/gcc-$GCC_VERSION/gcc-$GCC_VERSION.tar.bz2" -o gcc.tar.bz2 \
	&& curl -fSL "http://ftpmirror.gnu.org/gcc/gcc-$GCC_VERSION/gcc-$GCC_VERSION.tar.bz2.sig" -o gcc.tar.bz2.sig \
	&& gpg --batch --verify gcc.tar.bz2.sig gcc.tar.bz2 \
	&& mkdir -p /usr/src/gcc \
	&& tar -xf gcc.tar.bz2 -C /usr/src/gcc --strip-components=1 \
	&& rm gcc.tar.bz2* \
	&& cd /usr/src/gcc \
	&& ./contrib/download_prerequisites \
	&& { rm *.tar.* || true; } \
	&& dir="$(mktemp -d)" \
	&& cd "$dir" \
	&& /usr/src/gcc/configure \
		--disable-multilib \
		--enable-languages=c,c++,fortran,go \
	&& make -j"$(nproc)" \
	&& make install-strip \
	&& cd .. \
	&& rm -rf "$dir" \
	&& apt-get purge -y --auto-remove $buildDeps

# gcc installs .so files in /usr/local/lib64...
RUN echo '/usr/local/lib64' > /etc/ld.so.conf.d/local-lib64.conf \
	&& ldconfig -v

# ensure that alternatives are pointing to the new compiler and that old one is no longer used
RUN set -x \
	&& dpkg-divert --divert /usr/bin/gcc.orig --rename /usr/bin/gcc \
	&& dpkg-divert --divert /usr/bin/g++.orig --rename /usr/bin/g++ \
	&& dpkg-divert --divert /usr/bin/gfortran.orig --rename /usr/bin/gfortran \
	&& update-alternatives --install /usr/bin/cc cc /usr/local/bin/gcc 999
