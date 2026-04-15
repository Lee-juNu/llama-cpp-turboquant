# Usage: make start-gemma | make start-gpt-oss | make bench-gemma | make bench-gpt-oss | make build | make stop | make docker-gemma-up | make docker-gemma-down | make docker-gemma-logs

BUILD_DIR := build
SERVER    := $(BUILD_DIR)/bin/llama-server
BENCH     := $(BUILD_DIR)/bin/llama-bench
DOCKER_COMPOSE := docker compose -f compose.gemma.yml
GEMMA_MODEL   := models/gemma-4-31B-it-UD-Q4_K_XL.gguf
GEMMA_MMPROJ  := models/mmproj-gemma-4-31B-it-F16.gguf
GEMMA_MEDIA_PATH := /media
GEMMA_PORT    := 8081
GEMMA_CTX     := 131072
GPT_OSS_MODEL := models/UD-Q4_K_XL/gpt-oss-120b-UD-Q4_K_XL-00001-of-00002.gguf
GPT_OSS_PORT  := 8082
GPT_OSS_CTX   := 131072

.PHONY: build start-gemma start-gpt-oss bench-gemma bench-gpt-oss stop docker-gemma-up docker-gemma-down docker-gemma-logs

build:
	cmake --build $(BUILD_DIR) --config Release -j "$$(nproc)"

start-gemma:
	$(SERVER) \
	  -m $(GEMMA_MODEL) \
	  -c $(GEMMA_CTX) \
	  -ngl 99 \
	  --no-mmap \
	  -fa on \
	  -ctk turbo3 \
	  -ctv turbo3 \
	  -b 2048 \
	  -ub 512 \
	  --host 0.0.0.0 \
	  --port $(GEMMA_PORT)

bench-gemma:
	$(BENCH) \
	  -m $(GEMMA_MODEL) \
	  -ngl 99 -fa 1 \
	  -ctk turbo3 -ctv turbo3 \
	  -mmp 0 -b 2048 -ub 512 \
	  -p 512,4096,16384 -n 128

start-gpt-oss:
	$(SERVER) \
	  -m $(GPT_OSS_MODEL) \
	  -c $(GPT_OSS_CTX) \
	  -ngl 99 \
	  --no-mmap \
	  -fa on \
	  -ctk q8_0 \
	  -ctv q8_0 \
	  -b 2048 \
	  -ub 512 \
	  --host 0.0.0.0 \
	  --port $(GPT_OSS_PORT)

bench-gpt-oss:
	$(BENCH) \
	  -m $(GPT_OSS_MODEL) \
	  -ngl 99 -fa 1 \
	  -ctk q8_0 -ctv q8_0 \
	  -mmp 0 -b 2048 -ub 512 \
	  -p 512,4096,16384 -n 128

stop:
	@lsof -ti tcp:$(GEMMA_PORT) | xargs -r kill && echo "stopped gemma" || echo "nothing on port $(GEMMA_PORT)"
	@lsof -ti tcp:$(GPT_OSS_PORT) | xargs -r kill && echo "stopped gpt-oss" || echo "nothing on port $(GPT_OSS_PORT)"

docker-gemma-up:
	@mmproj="$(GEMMA_MMPROJ)"; \
	if [ ! -f "$$mmproj" ]; then \
	  found=$$(ls models/mmproj-gemma*.gguf 2>/dev/null | head -n1); \
	  if [ -n "$$found" ]; then \
	    echo "auto-detected mmproj: $$found"; \
	    mmproj="$$found"; \
	  else \
	    echo "missing mmproj: $$mmproj"; \
	    echo "to generate: python3 convert_hf_to_gguf.py /path/to/gemma-4-31b-it --mmproj --outtype f16 --outfile models/mmproj-gemma-4-31B-it-F16.gguf"; \
	    exit 1; \
	  fi; \
	fi; \
	GEMMA_MODEL=/$(GEMMA_MODEL) GEMMA_MMPROJ=/$$mmproj GEMMA_MEDIA_PATH=$(GEMMA_MEDIA_PATH) GEMMA_CTX=$(GEMMA_CTX) $(DOCKER_COMPOSE) up -d --build gemma-server

docker-gemma-down:
	$(DOCKER_COMPOSE) down

docker-gemma-logs:
	$(DOCKER_COMPOSE) logs -f gemma-server
