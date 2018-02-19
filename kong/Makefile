
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

define PROVISION_COMMANDS
	ansible-galaxy install --force --role-file=/requirements.yml
	ansible-playbook \
		--extra-vars="kong_admin_api_url=http://172.17.0.1:8001" \
		/playbook.yml
endef
export PROVISION_COMMANDS
ansible:
	$(eval pwd := $(shell dirname $(realpath $(lastword $(MAKEFILE_LIST)))))
	docker run --rm -it \
		--volume "$(pwd)/ansible-requirements.yml:/requirements.yml:ro" \
		--volume "$(pwd)/ansible-playbook.yml:/playbook.yml:ro" \
		williamyeh/ansible:alpine3 \
			/bin/sh -eux -c "$$PROVISION_COMMANDS"

setup: ansible
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
