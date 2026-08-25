.PHONY: deps lock build bench smoke list clean

OCAML_SERVERS = dream httpcats httpun-eio httpaf

# Install/resolve dependencies for all servers (Bun packages + dune locks).
deps: lock
	cd servers/elysia-bun && bun install

# (Re)generate each OCaml server's own dune.lock. Each server is a standalone
# dune project, so dependency versions are solved per server and never collide.
lock:
	@for s in $(OCAML_SERVERS); do \
	  echo "== dune pkg lock: $$s"; \
	  (cd servers/$$s && dune pkg lock) || exit 1; \
	done

# Build every server without running the benchmark.
build:
	cd servers/elysia-bun && bun install
	@for s in $(OCAML_SERVERS); do \
	  echo "== dune build: $$s"; \
	  (cd servers/$$s && ([ -d dune.lock ] || dune pkg lock) && dune build ./main.exe) || exit 1; \
	done

# Full benchmark run with the defaults from servers.json.
bench:
	python3 bench.py

# Quick pipeline check: short runs, small concurrency.
smoke:
	python3 bench.py --duration 3 --warmup 1 --connections 16

list:
	python3 bench.py --list

clean:
	rm -rf servers/elysia-bun/node_modules results
	@for s in $(OCAML_SERVERS); do rm -rf servers/$$s/_build; done
