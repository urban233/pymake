# pymake

A zero-dependency, production-ready task runner that replicates the ergonomics of GNU `make` or `invoke` using only the Python standard library (Python ≥ 3.8).

Tasks are plain Python functions. The CLI dispatcher automatically parses positional and keyword arguments, routes execution, and generates a formatted help menu derived directly from your function docstrings.

## Philosophy

* **Portability**: Relies purely on the Python standard library. No third-party dependencies required.
* **Composability**: Tasks are ordinary Python functions. Calling one task from another requires no special API.
* **Discoverability**: Running `./pymake.sh --help` automatically renders a formatted menu of available tasks.
* **Fail-fast**: Shell commands raise immediately on non-zero exit codes (consistent with GNU Make semantics).

## Installation

There is nothing to install via `pip`. Simply drop the required files into the root of your repository:

1. `pymakefile.py` (The task definitions and runner logic)
2. `pymake.sh` (POSIX wrapper)
3. `pymake.bat` (Windows wrapper)

Ensure the shell scripts are executable:

```bash
chmod +x pymake.sh

```

### Environment Integration (Optional)

If you rely on isolated environments (like Conda, Mamba, or standard `venv`), you can create a `.env_path` file in the same directory. Place the absolute path to your environment's root inside this file.

The `pymake.bat` and `pymake.sh` scripts will automatically detect this file and execute `pymakefile.py` using the specified environment's Python interpreter. If `.env_path` is missing, it falls back to the system's default `python` or `python3`.

## Usage

Tasks are executed by passing the function name to the wrapper script. Keyword arguments can be passed in `key=value` or `--key=value` formats.

```bash
# Print the auto-generated help menu
./pymake.sh --help

# Run a task with default arguments
./pymake.sh test

# Run a task passing keyword arguments
./pymake.sh test verbose=true match=api

```

## Defining Tasks

To add new tasks, simply open `pymakefile.py` and define a new Python function decorated with `@task`.

Use the `run()` helper to execute shell commands. By default, `run()` echoes the command to `stdout` and fails fast if the command exits with a non-zero status.

```python
from pymakefile import task, run

@task
def clean(cache: str = "true") -> None:
    """Remove build artifacts and compiled Python files.
    
    Args:
        cache: Pass cache=false to preserve .pytest_cache directories.
    """
    run("rm -rf build/ dist/ *.egg-info")
    
    if cache.lower() == "true":
        run("find . -type d -name '__pycache__' -exec rm -r {} +", check=False)

```

### The `run` API

The `run` function acts as the primary interface for invoking external processes.

```python
def run(
    cmd: str,
    *,
    check: bool = True,
    capture: bool = False,
    env: Optional[Dict[str, str]] = None,
) -> Optional[str]:

```

* **`cmd`**: The shell command string to execute.
* **`check`**: If `True` (default), raises an error and halts execution on a non-zero exit code. Set to `False` to tolerate failures.
* **`capture`**: If `True`, silences terminal output and returns the stripped `stdout` as a string.
* **`env`**: An optional dictionary of environment variables to pass to the subprocess.
