# MirrorLoop

## Setup

Requires `uv`, `make` and `git`.

```bash
make install
```

## Commands

```bash
make check          # format check, lint, types, tests
make fix            # auto-format and fix lint errors
make bench          # run benchmarks, save baseline
make bench-compare  # fail on >10% slowdown vs baseline
make hooks-check    # validate and run git hooks
make secrets        # scan git history for secrets
make update         # upgrade dependencies and hooks
```

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md).

## License

See [LICENSE](LICENSE).

## Citation

```bibtex
@software{mirror_loop,
  author = {Safaryan, Samvel},
  title = {MirrorLoop},
  year = {2026},
  url = {https://github.com/whoiam1101/mirror-loop}
}
```
