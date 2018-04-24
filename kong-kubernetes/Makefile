
curl = curl --silent
# "api" and "proxy" get updated with each makefile call
api =
export api
proxy =
export proxy
scheme = http
relname = meaningfulname
helm_chart = stable/kong
serv1 = service-1
serv2 = service-2

username = user1
api_key = 28cfebcc-45d9-11e8-a501-002590aa11ec

up:
	helm install --name $(relname) $(helm_chart) \
		--values values.yaml

#		--set image.tag=0.13.0 \
#		--set admin.type=NodePort \
#		--set admin.useTLS=false \
#		--set proxy.type=NodePort \
#		--set proxy.useTLS=false

down:
	helm delete --purge $(relname)

update:
	helm upgrade $(relname) $(helm_chart) \
		--values values.yaml

status:
	helm status $(relname)

api_endpoint:
	$(eval host := $(shell kubectl get nodes --namespace default -o jsonpath='{.items[0].status.addresses[0].address}'))
	$(eval port := $(shell kubectl get svc --namespace default $(relname)-kong-admin -o jsonpath='{.spec.ports[0].nodePort}'))
	$(eval api := $(scheme)://$(host):$(port))

proxy_endpoint:
	$(eval host := $(shell kubectl get nodes --namespace default -o jsonpath='{.items[0].status.addresses[0].address}'))
	$(eval port := $(shell kubectl get svc --namespace default $(relname)-kong-proxy -o jsonpath='{.spec.ports[0].nodePort}'))
	$(eval proxy := $(scheme)://$(host):$(port))

endpoint: api_endpoint proxy_endpoint
	echo $(api)
	echo $(proxy)

list_apis: api_endpoint
	$(curl) $(api)/apis | jq .

define PROVISION_COMMANDS
	ansible-galaxy install --force --role-file=/requirements.yml
	ansible-playbook \
		--extra-vars="kong_admin_api_url=$(api)" \
		/playbook.yml
endef
export PROVISION_COMMANDS
ansible: api_endpoint
	$(eval pwd := $(shell dirname $(realpath $(lastword $(MAKEFILE_LIST)))))
	docker run --rm -it \
		--volume "$(pwd)/ansible-requirements.yml:/requirements.yml:ro" \
		--volume "$(pwd)/ansible-playbook.yml:/playbook.yml:ro" \
		williamyeh/ansible:alpine3 \
			/bin/sh -eux -c "$$PROVISION_COMMANDS"

setup: api_endpoint ansible
	# rate-limiting
	$(curl) $(api)/plugins -X PUT \
		--data 'name=rate-limiting' \
		--data 'config.second=1' \
		--data 'config.limit_by=consumer'

service-%: proxy_endpoint
	$(curl) $(proxy)/$@ \
		--header 'apikey: $(api_key)'

requests:
	$(eval nrequests ?= 1)
	@for n in $(shell seq 1 $(nrequests)); do \
		$(MAKE) --no-print-directory $${service}; \
	done
