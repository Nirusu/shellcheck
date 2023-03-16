# Need to use Ubuntu 22.10 due since we need to bootstrap GHC 9.2.5 from GHC 9+. Ubuntu 22.04 only ships GHC 8.8.4.
FROM --platform=amd64 crazymax/osxcross:12.3-ubuntu@sha256:106636fe25ee59a05887d40d195468a0c3cb45dfd28f7228c0129272f7e46ae1 AS osxcross
FROM --platform=amd64 ubuntu:22.10@sha256:8c8f22bacbfba9678edddcc3b6d1e56cce02eff9cbf27ab46116de30cb79a3d6 AS build

# Install osxcross
COPY --from=osxcross /osxcross /osxcross
RUN apt-get update && apt-get install -y clang lld libc6-dev
ENV PATH="/osxcross/bin:$PATH"
ENV LD_LIBRARY_PATH="/osxcross/lib:$LD_LIBRARY_PATH"

# Set build target
ENV TARGET aarch64-apple-darwin21.4
ENV TARGETNAME darwin-aarch64

# Build dependencies
USER root
ENV DEBIAN_FRONTEND noninteractive
RUN apt-get update && apt-get dist-upgrade -y && apt-get install -y ghc automake autoconf llvm curl alex happy xz-utils build-essential

# Build GHC
WORKDIR /ghc
RUN curl -L "https://downloads.haskell.org/~ghc/9.2.5/ghc-9.2.5-src.tar.xz" | tar xJ --strip-components=1
RUN ./configure --host x86_64-linux-gnu --build x86_64-linux-gnu --target "$TARGET"
RUN cp mk/flavours/quick-cross.mk mk/build.mk && make -j "$(nproc)"
RUN make install
RUN curl -L "https://downloads.haskell.org/~cabal/cabal-install-3.9.0.0/cabal-install-3.9-x86_64-linux-alpine.tar.xz" | tar xJv -C /usr/local/bin

# Due to an apparent cabal bug, we specify our options directly to cabal
# It won't reuse caches if ghc-options are specified in ~/.cabal/config
ENV CABALOPTS "--with-ghc=$TARGET-ghc;--with-hc-pkg=$TARGET-ghc-pkg"

# Prebuild the dependencies
RUN cabal update && IFS=';' && cabal install --dependencies-only $CABALOPTS ShellCheck

# Copy the build script
COPY build /usr/bin

WORKDIR /scratch
ENTRYPOINT ["/usr/bin/build"]
