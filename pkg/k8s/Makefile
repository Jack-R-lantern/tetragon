# renovate: datasource=docker
BUILDER_IMAGE=docker.io/library/golang:1.22.5@sha256:1a9b9cc9929106f9a24359581bcf35c7a6a3be442c1c53dc12c41a106c1daca8
GO ?= go

all: generate

.PHONY: help
help:
	@echo 'Generated files:'
	@echo '    generate          - generate kubebuilder files'

.PHONY: generate
generate:
	docker container run --rm \
		--volume $(CURDIR)/../..:/src \
		--user "$(shell id -u):$(shell id -g)" \
		--env GOCACHE=/go/.cache \
		$(BUILDER_IMAGE) \
		make -C /src/pkg/k8s __do_generate

.PHONY: __do_generate
__do_generate:
	./tools/controller-gen crd paths=./apis/... output:dir=apis/cilium.io/client/crds/v1alpha1
	chmod +x vendor/k8s.io/code-generator/generate-internal-groups.sh
	# Do two invocations of generate-groups.sh, one with --plural-exceptions, one without
	export GOPATH=$$(go env GOPATH); \
	  bash vendor/k8s.io/code-generator/generate-groups.sh deepcopy,defaulter \
	  github.com/cilium/tetragon/pkg/k8s/client \
	  github.com/cilium/tetragon/pkg/k8s/apis \
	  cilium.io:v1alpha1 \
	  --go-header-file hack/custom-boilerplate.go.txt \
	  -o . \
	  --trim-path-prefix github.com/cilium/tetragon/pkg/k8s \
	  ; \
	  bash vendor/k8s.io/code-generator/generate-groups.sh lister,informer,client \
	  github.com/cilium/tetragon/pkg/k8s/client \
	  github.com/cilium/tetragon/pkg/k8s/apis \
	  cilium.io:v1alpha1 \
	  --go-header-file hack/custom-boilerplate.go.txt \
	  -o . \
	  --trim-path-prefix github.com/cilium/tetragon/pkg/k8s \
	  --plural-exceptions TracingPolicyNamespaced:TracingPoliciesNamespaced \
	  --plural-exceptions PodInfo:PodInfo \

.PHONY: vendor
vendor:
	$(GO) mod tidy
	$(GO) mod vendor
	$(GO) mod verify
