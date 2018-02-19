
curl = curl --silent
api = http://localhost:8001
proxy = http://localhost:8000
serv1 = service-1
serv2 = service-2

username = user1
api_key = c06b38dd-9c15-4306-8e43-3be0fbaf9f0a

up:
	cd docker-kong/compose && docker-compose up -d

down:
	cd docker-kong/compose && docker-compose down

list_apis:
	$(curl) $(api)/apis | jq .

setup_user:
	# add user
	$(curl) $(api)/consumers/$(user) -X DELETE
	$(curl) $(api)/consumers/ -X PUT --data 'username=$(user)'
	$(curl) $(api)/consumers/$(user)/key-auth -X POST \
		--data 'key=$(api_key)'

setup_service:
	# delete the API if it exists
	$(curl) $(api)/apis/$(service) -X DELETE
	# upsert the API
	$(curl) $(api)/apis -X PUT \
		--data 'name=$(service)' \
		--data 'uris=/$(service)' \
		--data 'upstream_url=https://$(service).mockbin.org/requests'
	# set up key auth
	$(curl) $(api)/apis/$(service)/plugins -X POST \
		--data 'name=key-auth' \
		--data 'config.hide_credentials=true'

setup:
	$(MAKE) setup_service service=$(serv1)
	$(MAKE) setup_service service=$(serv2)
	$(MAKE) setup_user username=$(user)
	# rate-limiting
	$(curl) $(api)/plugins -X PUT \
		--data 'name=rate-limiting' \
		--data 'config.second=5' \
		--data 'config.limit_by=consumer'

service-%:
	$(curl) $(proxy)/$@ \
		--header 'apikey: $(api_key)'

requests:
	$(eval nrequests ?= 1)
	@for n in $(shell seq 1 $(nrequests)); do \
		$(MAKE) --no-print-directory $${service}; \
	done
