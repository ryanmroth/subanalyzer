# subanalyzer

A passive reconnaissance pipeline that chains [subfinder](https://github.com/projectdiscovery/subfinder),
[httpx](https://github.com/projectdiscovery/httpx), and [gowitness](https://github.com/sensepost/gowitness)
into a single command. It enumerates subdomains, probes which are alive, checks
for exposed `.git` repositories, and captures screenshots of live hosts.

## Pipeline

1. `subfinder` enumerates subdomains for the target domain (passive sources).
2. `httpx` probes the results and records live hosts with status code and title.
3. `httpx` requests `/.git/HEAD` on each live host and reports only those that
   return `200` with a valid git ref in the body (content-validated, low noise).
4. `gowitness` captures a screenshot of every live host.

## Requirements

- `bash`. The script avoids bash 4 features, so the stock macOS bash (3.2) works
  as well as newer versions on Linux.
- Go 1.24 or newer if installing the tools from source.
- The three tools on your `PATH`:

  ```
  go install -v github.com/projectdiscovery/subfinder/v2/cmd/subfinder@latest
  go install -v github.com/projectdiscovery/httpx/cmd/httpx@latest
  go install github.com/sensepost/gowitness@latest
  ```

  Make sure `$(go env GOPATH)/bin` is on your `PATH`.

> Note: `httpx` is also the name of a Python HTTP client that installs its own
> `httpx` CLI. The script verifies the `httpx` on your `PATH` is the
> ProjectDiscovery tool and aborts if it is not.

## Installation

```bash
git clone https://github.com/ryanmroth/subanalyzer.git
cd subanalyzer
chmod +x subanalyzer.sh
```

Optionally symlink it onto your `PATH`:

```bash
ln -s "$PWD/subanalyzer.sh" ~/.local/bin/subanalyzer
```

## Usage

```bash
./subanalyzer.sh <domain> [output-dir]
```

Examples:

```bash
./subanalyzer.sh example.com
./subanalyzer.sh example.com ./engagements/example
```

If no output directory is given, results are written to `<domain>-results/`.

### Environment overrides

| Variable        | Default | Purpose                          |
|-----------------|---------|----------------------------------|
| `HTTPX_THREADS` | 50      | httpx concurrency (liveness)     |
| `HTTPX_TIMEOUT` | 10      | httpx timeout in seconds         |
| `GIT_THREADS`   | 30      | httpx concurrency (.git check)   |
| `GIT_TIMEOUT`   | 5       | httpx timeout for the .git check |
| `SHOT_X`        | 1280    | screenshot width in pixels       |
| `SHOT_Y`        | 720     | screenshot height in pixels      |

```bash
HTTPX_THREADS=100 SHOT_X=1920 SHOT_Y=1080 ./subanalyzer.sh example.com
```

## Output

| File                    | Contents                                  |
|-------------------------|-------------------------------------------|
| `subdomains.txt`        | All enumerated subdomains                 |
| `alive_subdomains.txt`  | Live hosts with status code and title     |
| `clean_urls.txt`        | Live host URLs only                       |
| `git_found.txt`         | Hosts with a validated exposed `.git`     |
| `screenshots/`          | PNG screenshots of live hosts             |

## Caveats

- The `.git` check requires `/.git/HEAD` to return the literal `ref: refs/heads/`
  string. Repositories that serve a packed-refs HEAD without that string will be
  missed, so the count can under-report. Drop the `-mr` flag in the script to
  match on `200` alone and triage manually.
- subfinder's passive sources are far more effective with API keys configured.
  See the subfinder documentation for the provider config file.

## Legal

This tool is for authorized security testing only. Run it only against domains
you own or have explicit written permission to test. Unauthorized scanning may
be illegal in your jurisdiction. You are responsible for your use of it.

## License

MIT. See [LICENSE](LICENSE).
