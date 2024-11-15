# Get the directory where this Makefile is, so we can use it below for including
DIR := $(strip $(shell dirname $(realpath $(lastword $(MAKEFILE_LIST)))))

include $(addprefix $(DIR)/vendor/github.com/openshift/build-machinery-go/make/, \
    targets/openshift/deps.mk \
)

include $(DIR)/.bingo/Variables.mk

.PHONY: verify
verify: ## Run downstream-specific verify
	$(MAKE) tidy fmt vet generate -C $(DIR)/../
	$(MAKE) manifests
	git diff --exit-code

.PHONY: manifests
manifests: $(KUSTOMIZE) $(YQ)
	$(DIR)/generate-manifests.sh

.PHONY: verify-manifests
verify-manifests: manifests
	git diff --exit-code

E2E_REGISTRY_NAME=docker-registry
E2E_REGISTRY_NAMESPACE=operator-controller-e2e
export E2E_REGISTRY_IMAGE := image-registry.openshift-image-registry.svc:5000/e2e-test-registry:devel
export LOCAL_REGISTRY_HOST := $(E2E_REGISTRY_NAME).$(E2E_REGISTRY_NAMESPACE).svc:5000
export CLUSTER_REGISTRY_HOST := $(E2E_REGISTRY_NAME).$(E2E_REGISTRY_NAMESPACE).svc:5000
.PHONY: test-e2e
test-e2e: ## Run the e2e tests.
	$(DIR)/build-test-registry.sh $(E2E_REGISTRY_NAMESPACE) $(E2E_REGISTRY_NAME) $(E2E_REGISTRY_IMAGE)
	$(MAKE) e2e -C $(DIR)/../
