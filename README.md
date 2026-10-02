# regex-redos-checker

Browser-based checker for catastrophic backtracking and ReDoS-prone regex patterns. Paste a pattern, get findings on the structures most likely to cause exponential or polynomial blowup, plus an evil-input synthesizer that produces a minimal string to demonstrate the worst case.

**Live demo:** https://0xelitesystem.github.io/regex-redos-checker/

Single HTML file, pentest-report aesthetic (default light theme, dark toggle). No build step, no dependencies, no network calls.

## Use

1. Paste the regex body, without slashes, into the pattern field and tick any flags (i, g, m, s, u).
2. Click "Analyze" to get findings per detector with a severity tag.
3. Optionally paste a string into Test Input to time the pattern against it.
4. Try the sample buttons (Nested quantifier, Alt overlap, Optional rep, Email validator, Safe pattern) to see each case.

## Why this exists

A single badly nested quantifier can hang a server on one crafted input, and that kind of pattern often passes code review. This is a single HTML file that flags the common ReDoS structures in your browser, with no tracking and no dependencies. MIT licensed.

## What it checks

Five detectors, applied to the parsed pattern:

1. **Nested quantifiers**, `(a+)+`, `(a*)*`, `(a+)*` and variants. The classic exponential case.
2. **Quantified group with overlapping alternatives**, `(a|a)*`, `(a|ab)*`, `(\w|\d)*`. Polynomial; bad for long inputs.
3. **Quantified group followed by quantifier on same character class**, `a*a*b`, `\d+\d+`. Polynomial.
4. **Greedy quantifier with no anchor and a frequently-failing tail**, `.*foo`, `.+\d`. Catastrophic on long non-matching input.
5. **Lookarounds wrapping quantified groups**, `(?=(a+)+b)`. Often missed because lookaround "doesn't consume" but still backtracks.

## Evil-input synthesis

For each detected vulnerable structure, the tool generates a minimal demonstration string and runs the regex against it on a controlled-size input (default 10 to 30 chars; user-adjustable up to a safe cap). Reports the elapsed wall time. Catastrophic patterns will hit the cap and be killed by the detector before the page hangs.

## Benchmarking

Optional manual test input field with a stopwatch. Useful for side-by-side comparison of a flagged pattern against a proposed safer rewrite.

## Severity scale

| Tag | Meaning |
|-----|---------|
| critical | Exponential blowup demonstrated on input ≤ 30 chars |
| high | Polynomial blowup demonstrated; viable DoS vector on user input |
| medium | Vulnerable structure present but synthesized input did not trigger blowup at tested size |
| low | Pattern is inefficient but not exploitable |
| info | No vulnerable structure detected; pattern looks safe |

## What this is NOT

Not a static-analysis equivalent of `recheck` or `safe-regex`. It implements a small set of detectors that catch the most common production ReDoS bugs; sophisticated patterns may slip past. It also runs in the JavaScript regex engine, which has different backtracking behavior from PCRE, RE2, .NET, or Python's `re`, a pattern that's safe in JS may still be dangerous in Java. Use it as a first pass on patterns shipped to JavaScript runtimes (browsers, Node, edge workers).

## Privacy

Patterns and test inputs stay in the browser. No analytics, no storage. The one exception: your light or dark theme choice is saved in localStorage under the key `theme`.

## Run locally

```
git clone https://github.com/0xelitesystem/regex-redos-checker
cd regex-redos-checker
```

Open `index.html` in a browser, or serve the folder with `python -m http.server` and visit http://localhost:8000.

## Build

No build step. It is a single `index.html` file.

## Samples

Five samples are wired to header buttons: a benign pattern (passes), a classic nested-quantifier case, a polynomial overlap, an unanchored greedy with failing tail, and a real-world example pulled from a published CVE.

## Related repositories

Part of a 10-repo security audit set.

Browser-based audit tools:
- [iam-policy-analyzer](https://github.com/0xelitesystem/iam-policy-analyzer)
- [terraform-security-linter](https://github.com/0xelitesystem/terraform-security-linter)
- [kubernetes-manifest-security-scanner](https://github.com/0xelitesystem/kubernetes-manifest-security-scanner)
- [session-cookie-auditor](https://github.com/0xelitesystem/session-cookie-auditor)

Reference collections:
- [incident-response-runbooks](https://github.com/0xelitesystem/incident-response-runbooks)
- [ai-llm-security-audit](https://github.com/0xelitesystem/ai-llm-security-audit)
- [api-security-audit-checklist](https://github.com/0xelitesystem/api-security-audit-checklist)
- [secrets-leak-response-runbook](https://github.com/0xelitesystem/secrets-leak-response-runbook)
- [threat-modeling-worksheets](https://github.com/0xelitesystem/threat-modeling-worksheets)

## More

Part of a catalog of single-file browser tools and plain-language references, all MIT licensed and dependency-free: [0xelitesystem.github.io](https://0xelitesystem.github.io/). Built by [elitesystem.ai](https://elitesystem.ai).

## License

MIT. See [LICENSE](LICENSE).
