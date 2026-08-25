.PHONY: deps build bench smoke list clean

# Install dependencies for all servers (Bun packages + opam packages).
deps:
	cd servers/elysia-bun && bun install
	cd servers/ocaml && opam install . --deps-only --yes

# Build every server without running the benchmark.
build:
	cd servers/elysia-bun && bun install
	cd servers/ocaml && dune build

# Full benchmark run with the defaults from servers.json.
bench:
	python3 bench.py

# Quick pipeline check: short runs, small concurrency.
smoke:
	python3 bench.py --duration 3 --warmup 1 --connections 16

list:
	python3 bench.py --list

clean:
	rm -rf servers/ocaml/_build servers/elysia-bun/node_modules results
