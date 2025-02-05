MAKEFLAGS += --warn-undefined-variables
SHELL := bash
.SHELLFLAGS := -eu -o pipefail -c
.DELETE_ON_ERROR:
.SUFFIXES:
.PHONY: help clean check_clean
SHELL := /bin/sh

 # Include personal settings
 include ../settings.sh


########################################################################################################################
#  Review each command in the targets. 


########################################################################################################################

init: mkdirs create-namespaces configure-namespace _create-policy-and-rbac

mkdirs:
	@rm -rf artifacts
	@mkdir -p artifacts/trust
	@mkdir -p artifacts/trust/bundles

create-namespaces:
	@kubectl apply -f namespaces/jetstack-secure.yaml

configure-namespace:
	@echo "Creating docker registry secret. You must have the credentials file to create a secret"
	@echo
	@kubectl -n 'jetstack-secure' create secret docker-registry venafi-jetstack-enterprise-key \
		--docker-password="$$(cat ${JS_ENTERPRISE_CREDENTIALS_FILE})" \
		--docker-email='$(JS_DOCKER_EMAIL)' \
		--docker-server='eu.gcr.io' \
		--docker-username='_json_key' \
		--dry-run='client' \
		--save-config=true -o yaml | kubectl apply -f -

_create-policy-and-rbac:
	@kubectl apply -f config/cert-policy-rbac.yaml
	@kubectl apply -f config/cert-policy.yaml


_helm_init:
	@echo 'Setting up helm with the correct repos'
	@helm repo add jetstack https://charts.jetstack.io
	@helm repo update
	
install-cert-manager-trust: _helm_init
	@echo 'Installing Jetstack cert-manager trust operator.....'
	@cp templates/values.yaml artifacts/trust/values.yaml 

	@helm upgrade \
	    --install -f artifacts/trust/values.yaml cert-manager-trust jetstack/cert-manager-trust \
  		--namespace jetstack-secure \
  		--version ${JS_CERT_MANAGER_TRUST_VERSION} \
		--wait

_remove-policy-and-rbac:
	@kubectl delete -f config/cert-policy-rbac.yaml || true
	@kubectl delete -f config/cert-policy.yaml || true

remove-cert-manager-trust:
	@helm uninstall -n jetstack-secure cert-manager-trust || true

clean: _remove-policy-and-rbac remove-cert-manager-trust


foo:
	@kubectl apply -f config/cert-policy-rbac.yaml
	@kubectl apply -f config/cert-policy.yaml
	@kubectl apply -f config/tmp.yaml 

bar:
	@kubectl delete -f config/cert-policy-rbac.yaml
	@kubectl delete -f config/cert-policy.yaml
	@kubectl delete -f config/tmp.yaml 

