
# Récupère le tag passé en argument
TAG ?= v1.40.0

all: clone build clean

# Clone le repo LocalAI sur le tag spécifié
clone:
	@echo "\033[34m Cloning LocalAI repository...\033[0m"
	@set -e;
	@git clone --branch $(TAG) https://github.com/mudler/LocalAI.git \
	&& echo "\033[32m Repository successfully cloned\033[0m" \
	|| echo "\033[31m Error during the repository clone\033[0m"

# Construit l'image Docker à partir du tag spécifié
build:
	@echo "\033[34m Building Docker Image\033[0m"
	@set -e;
	@docker buildx create --use
	@docker buildx build --platform linux/amd64,linux/arm64 -t localai:$(TAG) LocalAI \
	&& echo "\033[32m Build successful\033[0m" \
	|| echo "\033[31m Error during build\033[0m"

startcontainer: build
	@echo "\033[34m Test 1: Show models\033[0m"
	@set -e;
	@docker run --name localai-$(TAG) --rm -p 8080:8080 -d -e HUGGINGFACEHUB_API_TOKEN=<TOKEN> -e PRELOAD_MODELS='[{"url":"github:go-skynet/model-gallery/gpt4all-j.yaml"}]' localai:$(TAG)
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

# Nettoie le répertoire de travail
clean:
	@echo "\033[34m Cleaning working directory\033[0m"
	@rm -rf LocalAI
	@rm -rf models
	@echo "\033[34m Cleaning Docker build context\033[0m"
	@docker buildx prune --all --force \
	&& echo "\033[32m Docker build context cleaned\033[0m" \
	|| echo "\033[31m Error during Docker build context cleanup\033[0m"

# Pushes the Docker image to GitHub Container Registry (ghcr.io)
push:
	@echo "\033[34m Pushing Docker image to ghcr.io\033[0m"
	@set -e;
	@docker tag localai:$(TAG) ghcr.io/ducretje/localai-image:$(TAG)
	@docker push ghcr.io/ducretje/localai-image:$(TAG) \
	&& echo "\033[32m Docker image pushed to ghcr.io\033[0m" \
	|| echo "\033[31m Error during Docker image push\033[0m"

.PHONY: all clone build clean test1 test2 push
