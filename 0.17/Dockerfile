FROM debian:stretch-slim as builder

RUN set -ex \
	&& apt-get update \
	&& apt-get install -qq --no-install-recommends ca-certificates dirmngr gpg wget \
  && apt-get install -y curl gnupg \
  && apt-get clean \
  && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* \
  && gpg --batch --keyserver hkps://keys.openpgp.org --recv-keys B42F6819007F00F88E364FD4036A9C25BF357DD4

ENV LITECOIN_VERSION=0.17.1
ENV	LITECOIN_URL=https://download.litecoin.org/litecoin-$LITECOIN_VERSION/linux/litecoin-$LITECOIN_VERSION-x86_64-linux-gnu.tar.gz \
	LITECOIN_SHA256=9cab11ba75ea4fb64474d4fea5c5b6851f9a25fe9b1d4f7fc9c12b9f190fed07 \
	LITECOIN_ASC_URL=https://download.litecoin.org/litecoin-$LITECOIN_VERSION/SHA256SUMS.asc \
	LITECOIN_PGP_KEY=fe3348877809386c

RUN set -ex \
	&& cd /tmp \
	&& wget -qO litecoin.tar.gz "$LITECOIN_URL" \
	&& echo "$LITECOIN_SHA256 litecoin.tar.gz" | sha256sum -c - \
	&& gpg --no-tty --keyserver keyserver.ubuntu.com --recv-keys "$LITECOIN_PGP_KEY" \
	&& wget -qO litecoin.asc "$LITECOIN_ASC_URL" \
	&& gpg --verify litecoin.asc \
	&& tar -xzvf litecoin.tar.gz -C /usr/local --strip-components=1 --exclude=*-qt


FROM debian:stretch-slim
COPY --from=builder /usr/local/bin/litecoind /usr/local/bin/litecoin-cli /usr/local/bin/
RUN groupadd -r litecoin && useradd -r -m -g litecoin litecoin \
	&& ln -s /usr/local/bin/litecoin-cli /usr/local/bin/c


ENV GOSU_VERSION=1.11

RUN curl -o /usr/local/bin/gosu -fSL https://github.com/tianon/gosu/releases/download/${GOSU_VERSION}/gosu-$(dpkg --print-architecture) \
  && curl -o /usr/local/bin/gosu.asc -fSL https://github.com/tianon/gosu/releases/download/${GOSU_VERSION}/gosu-$(dpkg --print-architecture).asc \
  && gpg --verify /usr/local/bin/gosu.asc \
  && rm /usr/local/bin/gosu.asc \
  && chmod +x /usr/local/bin/gosu

ENV LITECOIN_DATA=/home/litecoin/.litecoin


COPY docker-entrypoint.sh /entrypoint.sh

VOLUME ["/home/litecoin/.litecoin"]

EXPOSE 9332 9333 19332 19333 19444

USER litecoin

ENTRYPOINT ["/entrypoint.sh"]

CMD ["litecoind"]
