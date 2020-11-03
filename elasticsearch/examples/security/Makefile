default: test

include ../../../helpers/examples.mk

RELEASE := helm-es-security
ELASTICSEARCH_IMAGE := docker.elastic.co/elasticsearch/elasticsearch:$(STACK_VERSION)

install:
	helm upgrade --wait --timeout=600 --install --values ./security.yml $(RELEASE) ../../

purge:
	kubectl delete secrets elastic-credentials elastic-certificates elastic-certificate-pem elastic-certificate-crt|| true
	helm del --purge $(RELEASE)

test: secrets install goss

pull-elasticsearch-image:
	docker pull $(ELASTICSEARCH_IMAGE)

secrets:
	docker rm -f elastic-helm-charts-certs || true
	rm -f elastic-certificates.p12 elastic-certificate.pem elastic-certificate.crt elastic-stack-ca.p12 || true
	docker run --name elastic-helm-charts-certs -i -w /tmp \
		docker.elastic.co/elasticsearch/elasticsearch:7.9.1 \
		/bin/sh -c " \
			elasticsearch-certutil ca --out /tmp/elastic-stack-ca.p12 --pass '' && \
			elasticsearch-certutil cert --name security-master --dns security-master --ca /tmp/elastic-stack-ca.p12 --pass '' --ca-pass '' --out /tmp/elastic-certificates.p12" && \
	docker cp elastic-helm-charts-certs:/tmp/elastic-certificates.p12 ./ && \
	docker rm -f elastic-helm-charts-certs && \
	kubectl create secret generic elastic-certificates -n helx --from-file=elastic-certificates.p12 && \
	rm -f elastic-certificates.p12 elastic-certificate.pem elastic-certificate.crt elastic-stack-ca.p12
