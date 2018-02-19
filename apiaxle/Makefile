nginx = http://localhost:8000
api = http://localhost:3100
api_key = 00fb7fa2-0916-412b-9d50-867ceb3f505d
api_key_qps = 10

serv1 = service-1
serv2 = service-2

up:
	#docker-compose up -d $(serv1) $(serv2) nginx
	docker-compose up -d nginx

ps:
	docker-compose ps

down:
	-docker-compose down

define SETUP_COMMANDS
	api "$(serv1)" delete
	api "$(serv2)" delete
	key "$(api_key)" delete

	api "$(serv1)" create endPoint="$(serv1).mockbin.org" defaultPath="/request" protocol="https" strictSSL=false
	api "$(serv2)" create endPoint="$(serv2).mockbin.com" defaultPath="/request" protocol="https" strictSSL=false

	key "$(api_key)" create qps=$(api_key_qps)
	api "$(serv1)" linkkey "$(api_key)"
	api "$(serv2)" linkkey "$(api_key)"

	info
endef
export SETUP_COMMANDS
setup:
	echo "$$SETUP_COMMANDS" | docker-compose run --rm apiaxle-repl

service-%:
	curl $(nginx)/$@?api_key=$(api_key)
	@echo

list_services:
	curl --silent $(api)/v1/apis | jq .results[]

indices:
	$(eval services := $(shell $(MAKE) --silent list_services))
	@for service in $(services); do \
		$(MAKE) --no-print-directory $${service}; \
	done

requests:
	$(eval nrequests ?= 1)
	@for n in $(shell seq 1 $(nrequests)); do \
		$(MAKE) --no-print-directory $${service}; \
	done
