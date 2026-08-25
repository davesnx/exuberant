.PHONY: first deps switches build bench smoke list clean distclean

OCAML_SERVERS = dream httpcats httpun-eio httpaf
OCAML_VERSION ?= 5.3.0

# Start here: get a single OCaml server (dream) built in its own local opam
# switch and benchmarked in a short run next to the Bun baseline.
# Needs opam initialized once (opam init -a).
first:
	cd servers/dream && ([ -d _opam ] || opam switch create . $(OCAML_VERSION) --deps-only --yes) \
	  && opam exec -- dune build ./main.exe
	cd servers/elysia-bun && bun install
	python3 bench.py --servers elysia-bun,dream --duration 5 --warmup 2 --connections 32

# Install/resolve dependencies for all servers (Bun packages + opam switches).
deps: switches
	cd servers/elysia-bun && bun install

# One local opam switch (./_opam) per OCaml server: each server's dependency
# versions are solved per project and never collide with the others.
switches:
	@for s in $(OCAML_SERVERS); do \
	  echo "== opam switch: $$s"; \
	  (cd servers/$$s && ([ -d _opam ] || opam switch create . $(OCAML_VERSION) --deps-only --yes)) || exit 1; \
	done

# Build every server without running the benchmark.
build:
	cd servers/elysia-bun && bun install
	@for s in $(OCAML_SERVERS); do \
	  echo "== dune build: $$s"; \
	  (cd servers/$$s && opam exec -- dune build ./main.exe) || exit 1; \
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

# Also remove the per-server opam switches (expensive to recreate).
distclean: clean
	@for s in $(OCAML_SERVERS); do rm -rf servers/$$s/_opam; done
