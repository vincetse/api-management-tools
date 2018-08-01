nginx = http://localhost:8000

up:
	docker-compose up -d

down logs:
	-docker-compose $@

service-%:
	curl $(nginx)/$@

requests:
	$(eval connections ?= 128)
	$(eval threads ?= 1)
	docker run --net=host irus/wrk2 \
		--threads $(threads) \
		--connections $(connections) \
		--duration 60s \
		--rate 20 \
		--latency \
		$(nginx)/$${service}

foo:
	$(eval nrequests ?= 1)
	@for n in $(shell seq 1 $(nrequests)); do \
		$(MAKE) --no-print-directory $${service}; \
	done
