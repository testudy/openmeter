# A Self-Documenting Makefile: http://marmelab.com/blog/2016/02/29/auto-documented-makefile.html

.PHONY: publish-python-sdk
publish-python-sdk: ## Publish Python SDK
	$(call print-target)
	@if [ -z "$$PY_SDK_RELEASE_TAG" ]; then \
		echo "ERROR: PY_SDK_RELEASE_TAG is required"; \
		echo "Usage: PY_SDK_RELEASE_TAG=alpha make publish-python-sdk"; \
		exit 1; \
	fi
	@if [ "$$PY_SDK_RELEASE_TAG" != "alpha" ] && [ "$$PY_SDK_RELEASE_TAG" != "beta" ]; then \
		echo "ERROR: PY_SDK_RELEASE_TAG must be either 'alpha' or 'beta'"; \
		echo "Got: $$PY_SDK_RELEASE_TAG"; \
		exit 1; \
	fi

	@if [ -z "$$PY_SDK_RELEASE_VERSION" ]; then \
		LATEST_VERSION=$$(curl -s https://pypi.org/pypi/openmeter/json | grep -o '"version":"[^"]*"' | head -1 | cut -d'"' -f4); \
		if [ -z "$$LATEST_VERSION" ]; then \
			if [ "$$PY_SDK_RELEASE_TAG" = "alpha" ]; then \
				PY_SDK_RELEASE_VERSION="1.0.0a0"; \
			else \
				PY_SDK_RELEASE_VERSION="1.0.0b0"; \
			fi; \
		else \
			BASE_VERSION=$$(echo $$LATEST_VERSION | grep -o '^[0-9]*\.[0-9]*\.[0-9]*'); \
			if [ "$$PY_SDK_RELEASE_TAG" = "alpha" ]; then \
				PRE_NUM=$$(echo $$LATEST_VERSION | grep -o 'a[0-9]*' | grep -o '[0-9]*' || echo "-1"); \
				NEXT_NUM=$$((PRE_NUM + 1)); \
				PY_SDK_RELEASE_VERSION="$${BASE_VERSION}a$${NEXT_NUM}"; \
			else \
				PRE_NUM=$$(echo $$LATEST_VERSION | grep -o 'b[0-9]*' | grep -o '[0-9]*' || echo "-1"); \
				NEXT_NUM=$$((PRE_NUM + 1)); \
				PY_SDK_RELEASE_VERSION="$${BASE_VERSION}b$${NEXT_NUM}"; \
			fi; \
		fi; \
		export PY_SDK_RELEASE_VERSION; \
	fi; \
	if [ -z "$$COMMIT_SHORT_SHA" ]; then \
		COMMIT_SHORT_SHA=$$(git rev-parse --short=12 HEAD); \
	fi; \
	printf "VERSION = \"%s\"" "$${PY_SDK_RELEASE_VERSION}" > openmeter/_version.py || true; \
	printf "COMMIT = \"%s\"" "$${COMMIT_SHORT_SHA}" > openmeter/_commit.py || true; \
	poetry version "$${PY_SDK_RELEASE_VERSION}"; \
	CACHE_BUSTER="$$(date --rfc-3339=seconds)" poetry publish --build --dry-run; \
	echo "✅ Published Python SDK version $${PY_SDK_RELEASE_VERSION}"

.PHONY: help
.DEFAULT_GOAL := help
help:
	@grep -h -E '^[a-zA-Z_-]+:.*?## .*$$' $(MAKEFILE_LIST) | awk 'BEGIN {FS = ":.*?## "}; {printf "\033[36m%-30s\033[0m %s\n", $$1, $$2}'

# Variable outputting/exporting rules
var-%: ; @echo $($*)
varexport-%: ; @echo $*=$($*)

define print-target
    @printf "Executing target: \033[36m$@\033[0m\n"
endef
