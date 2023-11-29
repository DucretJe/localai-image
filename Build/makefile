
# Set tag for the docker image
TAG ?= v1.40.0
# Set suffix if needed
TEST_SUFFIX ?= ""
# Set architecture for the docker image
ARCH ?= "linux/arm64/v8"
# Set docker username
DOCKER_USERNAME ?= "DucretJe"
# Set docker password
DOCKER_PASSWORD ?= "PASSWORD"

publish: build clean
test: test2 clean

login:
	@echo "\033[34m Logging in to Docker Hub...\033[0m"
	@set -e;
	@echo $(DOCKER_PASSWORD) | docker login ghcr.io -u $(DOCKER_USERNAME) --password-stdin \
	&& echo "\033[32m Successfully logged in to Docker Hub\033[0m" \
	|| echo "\033[31m Error during Docker Hub login\033[0m"

clone:
	@echo "\033[34m Cloning LocalAI repository...\033[0m"
	@set -e;
	@cd $(WORKDIR)
	@git clone --branch $(TAG) https://github.com/mudler/LocalAI.git \
	&& echo "\033[32m Repository successfully cloned\033[0m" \
	|| echo "\033[31m Error during the repository clone\033[0m"

build: login clone
	@echo "\033[34m Building Docker Image\033[0m"
	@set -e;
	@docker buildx create --use
	@docker buildx build --load --platform $(ARCH) --cache-from=type=gha --cache-to=type=gha,mode=max -t ghcr.io/DucretJe/localai-image:$(TAG)$(TEST_SUFFIX) LocalAI \
	&& echo "\033[32m Build successful\033[0m" \
	|| echo "\033[31m Error during build\033[0m"

startcontainer: build
	@echo "\033[34m Test 1: Start Container\033[0m"
	@set -e;
	@docker run --name localai-$(TAG) --rm -p 8080:8080 --platform $(ARCH) -d -e PRELOAD_MODELS='[{"url":"github:go-skynet/model-gallery/gpt4all-j.yaml"}]' --pull always ghcr.io/DucretJe/localai-image:$(TAG)$(TEST_SUFFIX)
	@echo "\033[34m󰔟 Waiting for container to become ready...\033[0m"
	@/bin/bash -c ' \
		start_time=$$(date +%s); \
		while [ $$(( $$(date +%s) - $$start_time )) -le 240 ]; do \
			status=$$(docker inspect --format="{{.State.Health.Status}}" localai-$(TAG)); \
			echo -ne "\r\033[K\033[33mContainer is: $$status ($$(( $$(date +%s) - $$start_time ))s)\033[0m"; \
			if [ "$$status" = "healthy" ]; then \
				echo "\033[32m\n Successfully started container\033[0m"; \
				exit 0; \
			fi; \
			sleep 1; \
		done; \
		echo "\033[31m Failed to start container after 240s\033[0m"; \
		exit 1; \
		'

# Test 1: Show models
test1: startcontainer
	@echo "\033[34m Test 1: List models\033[0m"
	@set -e;
	@curl http://localhost:8080/v1/models \
	&& echo "\033[32m\n Successfully listed models\033[0m" \
	|| echo "\033[31m\n Failed to list models\033[0m"

# Test 2: Basic prompt
test2: test1
	@echo "\033[34m Test 2: Basic prompt\033[0m"
	@set -e;
	@mkdir -p models
	@echo "\033[34m Starting conversation\033[0m"
	@curl -X POST http://localhost:8080/v1/chat/completions -H "Content-Type: application/json" \
	-d '{"model": "gpt4all-j", "messages": [{"role": "user", "content": "How are you?"}],"temperature": 0.9}' | grep -q '"created":' \
	&& echo "\033[32m\n Successfully prompted LocalAI\033[0m" \
	|| echo "\033[31m\n Failed to to prompt LocalAI\033[0m"

clean:
	@echo "\033[34m Cleaning working directory\033[0m"
	@rm -rf LocalAI
	@rm -rf models
	@echo "\033[34m Cleaning Docker build context\033[0m"
	@docker buildx prune --all --force \
	&& echo "\033[32m Docker build context cleaned\033[0m" \
	|| echo "\033[31m Error during Docker build context cleanup\033[0m"

.PHONY: all test publish clone build startcontainer clean test1 test2 push
