---
title: Aider Benchmark Harness
tags: [benchmark, aider, llm, evaluation, exercism, docker]
category: benchmark
created: 2024-07-01
updated: 2026-10-20
source: external
type: index
status: stable
lifecycle: active
---

# Aider Benchmark Harness

---

## 💡 概述

Aider uses benchmarks to quantitatively measure how well it works
with various LLMs.
This directory holds the harness and tools needed to run the benchmarking suite.

The benchmark is based on the [Exercism](https://github.com/exercism/python) coding exercises.
This
benchmark evaluates how effectively aider and LLMs can translate a
natural language coding request into executable code saved into
files that pass unit tests.
It provides an end-to-end evaluation of not just
the LLM's coding ability, but also its capacity to *edit existing code*
and *format those code edits* so that aider can save the
edits to the local source files.

See [this writeup for a longer discussion about the benchmark](https://aider.chat/2024/12/21/polyglot.html).

---

## ⚡ 功能

- End-to-end evaluation of LLM coding ability with natural language requests
- Measures ability to edit existing code and format code edits correctly
- Runs exercises in random order with configurable parallelism via `--threads`
- Generates YAML summary reports with pass rates, error counts, costs, and timing
- Supports filtering exercises by keywords (`--keywords`) and limiting test count (`--num-tests`)
- Configurable model settings via `--read-model-settings=<filename.yml>`

---

## 📊 指标

Key statistics reported in the benchmark YAML output:

| Metric | Description |
|--------|-------------|
| `pass_rate_1`, `pass_rate_2`, ... | Percent of tasks with all tests passing (multiple entries depending on `--tries`) |
| `percent_cases_well_formed` | Percentage of well-formed responses |
| `error_outputs` | Count of error outputs |
| `num_malformed_responses` | Count of malformed responses |
| `num_with_malformed_responses` | Count of cases with malformed responses |
| `user_asks` | Count of user asks |
| `lazy_comments` | Count of lazy comments |
| `syntax_errors` | Count of syntax errors |
| `indentation_errors` | Count of indentation errors |
| `exhausted_context_windows` | Count of exhausted context windows |
| `test_timeouts` | Count of test timeouts |
| `seconds_per_case` | Average seconds per test case |
| `total_cost` | Total cost of the benchmark run |

Example YAML record from a benchmark report:

```yaml
- dirname: 2024-07-04-14-32-08--claude-3.5-sonnet-diff-continue
  test_cases: 225
  model: claude-3.5-sonnet
  edit_format: diff
  commit_hash: 35f21b5
  pass_rate_1: 57.1
  pass_rate_2: 77.4
  percent_cases_well_formed: 99.2
  error_outputs: 23
  num_malformed_responses: 4
  num_with_malformed_responses: 1
  user_asks: 2
  lazy_comments: 0
  syntax_errors: 1
  indentation_errors: 0
  exhausted_context_windows: 0
  test_timeouts: 1
  command: aider --sonnet
  date: 2024-07-04
  versions: 0.42.1-dev
  seconds_per_case: 17.6
  total_cost: 3.6346
```

---

## 🏛️ 设计

The benchmark is intended to be run *inside a docker container*.
This is because the benchmarking harness will be
taking code written by an LLM
and executing it without any human review or supervision!
The LLM could generate dangerous python that harms your system, like this: `import os; os.system("sudo rm -rf /")`.
Running inside a docker container helps limit the damage that could be done.

The yaml report also includes all the settings which were in effect for the benchmark run.
It also reports the git hash of the repo at the time that the benchmark was
run, with `(dirty)` if there were uncommitted changes.
It's good practice to commit the repo before starting a benchmark run.
This way the `model`, `edit_format` and `commit_hash`
should be enough to reliably reproduce any benchmark run.

---

## 📁 目录

「待补充」

---

## 🔗 资源

- [Exercism Python exercises](https://github.com/exercism/python) — source of the coding exercises
- [Polyglot benchmark writeup](https://aider.chat/2024/12/21/polyglot.html) — longer discussion about the benchmark
- [Polyglot benchmark repo](https://github.com/Aider-AI/polyglot-benchmark) — exercise repository
- [Aider leaderboard data files](https://github.com/Aider-AI/aider/blob/main/aider/website/_data/) — examples of benchmark report yaml
- [Model settings documentation](https://aider.chat/docs/config/adv-model-settings.html#model-settings)

---

## 🚀 快速开始

There are 3 main tasks involved in benchmarking aider:

1. Install and setup for benchmarking.
2. Run the benchmark to measure performance across all the exercises.
3. Generate a summary report of how many of the exercises succeeded or failed.

### Setup for benchmarking

First, prepare all the groundwork for running the benchmarks.
These steps only need to be done once.

```
# Clone the aider repo
git clone https://github.com/Aider-AI/aider.git

# Create the scratch dir to hold benchmarking results inside the main aider dir:
cd aider
mkdir tmp.benchmarks

# Clone the repo with the exercises
git clone https://github.com/Aider-AI/polyglot-benchmark tmp.benchmarks/polyglot-benchmark

# Build the docker container
./benchmark/docker_build.sh
```

### Running the benchmark

Launch the docker container and run the benchmark inside it:

```
# Launch the docker container
./benchmark/docker.sh

# Inside the container, install aider as a development build.
# This way you're running the code that you cloned above, including any local changes.
pip install -e .[dev]

# Run the benchmark:
./benchmark/benchmark.py a-helpful-name-for-this-run --model gpt-3.5-turbo --edit-format whole --threads 10 --exercises-dir polyglot-benchmark
```

The above will create a folder `tmp.benchmarks/YYYY-MM-DD-HH-MM-SS--a-helpful-name-for-this-run` with benchmarking results.
Run like this, the script will run all the exercises in a random order.

You can run `./benchmark/benchmark.py --help` for a list of all the arguments, but here are the most useful to keep in mind:

- `--model` is the name of the model, same as you would pass directly to `aider`.
- `--edit-format` is the name of the edit format, same as you would pass directly to `aider`. When working with an experimental LLM, I recommend starting with `whole`
- `--threads` specifies how many exercises to benchmark in parallel. Start with a single thread if you are working out the kinks on your benchmarking setup or working with a new model, etc. Once you are getting reliable results, you can speed up the process by running with more threads. 10 works well against the OpenAI APIs.
- `--num-tests` specifies how many of the tests to run before stopping. This is another way to start gently as you debug your benchmarking setup.
- `--keywords` filters the tests to run to only the ones whose name match the supplied argument (similar to `pytest -k xxxx`).
- `--read-model-settings=<filename.yml>` specify model settings, see here: https://aider.chat/docs/config/adv-model-settings.html#model-settings

### Benchmark report

You can generate stats about any benchmark, including ones which are still running.
You don't need to run this inside the docker container, as it is just
collecting stats not executing unsafe python.

```
# Generate stats for a specific benchmarking directory
./benchmark/benchmark.py --stats tmp.benchmarks/YYYY-MM-DD-HH-MM-SS--a-helpful-name-for-this-run
```

---

## ⚙️ 技术栈

- Python (benchmark harness scripts)
- Docker (sandboxed execution environment)
- Bash (setup and container scripts)
- YAML (benchmark report format)
- Exercism Python exercises (test corpus)