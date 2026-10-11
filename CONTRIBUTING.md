# Contributing to MirrorLoop

These rules apply to everyone who writes code here, humans and AI agents alike.
Most of them are enforced by tooling: if `make check` fails, the change is not merged.

## Workflow

```bash
make install     # set up the environment
make hooks       # install pre-commit and pre-push hooks (once)
make fix         # auto-format and auto-fix lint errors
make check       # format check, lint, type check, tests: must pass before a PR
make bench       # benchmarks: required for performance changes
```

Keep each pull request focused on one change. Do not add dependencies without discussing them in an issue first.

## Code rules

### 1. No comments

Code explains itself through names, types and structure.
A comment is allowed only when it states something the code cannot: a non-obvious reason, a workaround for an external bug, a link to a paper.
It describes why, never what. Commented-out code is never committed.

### 2. Readable code

- Small functions that do one thing.
- Early returns instead of deep nesting.
- If a block needs a comment to be understood, extract it into a named function.

### 3. Clear names

- Names describe meaning, not type: `confidence_by_answer`, not `d` or `data2`.
- Functions are verbs, values are nouns, booleans read as questions (`is_calibrated`).
- No abbreviations a newcomer would have to look up.

### 4. Functional style

- Pure functions by default: output depends only on input, no hidden state.
- Never mutate arguments. Return new values.
- Immutable data: `@dataclass(frozen=True, slots=True)`, `tuple`, `frozenset`, `NamedTuple`.
- Comprehensions, `map`, `filter`, `functools`, `itertools` over loops with accumulators.
- Side effects (I/O, randomness, time, logging) live at the edges; the core stays pure.
- Readability wins over purity: if a loop is clearer than a fold, use the loop.

### 5. Performance and benchmarks

- JIT-compile numerical hot paths when possible (`numba.njit`, `jax.jit`, `torch.compile`, whichever fits). Keep JIT functions pure, small and fully typed.
- No performance change without a benchmark. Every optimization (JIT, vectorization, algorithm swap) comes with a benchmark in `benchmarks/` written with `pytest-benchmark`.
- The pull request description contains before/after numbers from `make bench`. Claims without numbers are not accepted.
- Benchmarks exclude compilation time: call the function once before measuring and report compile time separately.
- `make bench-compare` fails on a slowdown of more than 10% against the saved baseline. Such a regression needs an explicit justification.
- Benchmarks are not part of `make check` because they are noisy.

### 6. Strict typing

Precise types prevent whole classes of bugs. Typing is checked by `ty`; missing annotations are caught by `ruff` (`ANN` rules).

- **`Any` is forbidden**, explicit or implicit (unannotated functions, bare `dict`, `list`, `Callable`).
- Annotate signatures and class fields only: every function and method has full parameter and return annotations, and every class attribute (dataclass field, `NamedTuple` field, `TypedDict` key) is typed.
- Do not annotate local variables; the checker infers them. Add an annotation to a local only when inference cannot work (for example an empty collection).
- Use `collections.abc` instead of `typing` for abstract types: `Sequence`, `Mapping`, `Iterable`, `Callable`, `MutableSequence`, `MutableMapping`, and so on.
- Choose by mutability:
  - read-only mapping: `Mapping[K, V]`
  - mapping the function modifies: `MutableMapping[K, V]`
  - read-only sequence: `Sequence[T]`; iterated once: `Iterable[T]`
  - concrete `dict` / `list` only for values the function creates and owns
- Parameters take the most general type, return values the most specific one.
- Prefer precise structure over loose containers: `TypedDict`, frozen dataclasses, `Protocol`, `Literal`, `NewType`, generics.
- Arrays carry their dtype: `NDArray[np.float32]`.
- Unknown input is `object`, narrowed with `isinstance` or a validator at the boundary.
- Suppress only with a rule-specific `# ty: ignore[rule-name]` and a reason in the pull request description. Blanket ignores are forbidden.

### 7. Tests

- New code and behavior changes come with tests. Every bug fix starts with a failing regression test.
- `pytest`, with the source layout mirrored: `mirror_loop/foo.py` is tested in `tests/test_foo.py`.
- Prefer property-based tests (`hypothesis`) for pure functions wherever an invariant can be stated.
- Tests are deterministic: fixed seeds, no network, no wall-clock time, no dependence on order.
- Test names describe behavior: `test_confidence_is_clipped_to_unit_interval`.
- JIT code is tested compiled and with JIT disabled (for example `NUMBA_DISABLE_JIT=1`) so failures give readable tracebacks.

## For AI agents

Run `make check` before finishing and fix everything it reports.
Never silence the checks (`# noqa`, blanket `# ty: ignore`, skipped or deleted tests) and never bypass hooks with `--no-verify`.
