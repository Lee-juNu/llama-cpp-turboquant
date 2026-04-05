# Usage: make start-gemma | make bench-gemma | make build | make stop

BUILD_DIR := build
SERVER    := $(BUILD_DIR)/bin/llama-server
BENCH     := $(BUILD_DIR)/bin/llama-bench
GEMMA_MODEL := models/gemma-4-31B-it-UD-Q4_K_XL.gguf
GEMMA_PORT  := 8081
GEMMA_CTX   := 131072

.PHONY: build start-gemma bench-gemma stop

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

stop:
	@lsof -ti tcp:$(GEMMA_PORT) | xargs -r kill && echo "stopped" || echo "nothing on port $(GEMMA_PORT)"