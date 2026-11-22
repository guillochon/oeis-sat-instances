# OEIS SAT instances

This repository collects SAT encodings of open terms of sequences in the
[On-Line Encyclopedia of Integer Sequences](https://oeis.org). Each instance decides one unknown term, or one
bound on it. A fast enough solver turns an answer into a new OEIS term.

Every sequence ships:

* a **generator** that rebuilds every instance from scratch (so the encoding can be audited and the large CNFs
  need not be trusted as opaque files);
* a **verifier** that turns a SAT model back into the combinatorial object and checks it independently;
* **calibration instances** whose answers are already known, so solver speed can be measured on the same
  encoding before any claim about the open one;
* the **open instance(s)**, with what SAT and UNSAT would each mean.

## Sequences

| sequence | question | open instance | status |
|---|---|---|---|
| [A067782](sequences/A067782) | least depth of an n-input sorting network | does an 18-input network of depth 10 exist? | open: a(18) ∈ {10, 11} |

## Why these make good solver tests

A SAT answer is a certificate. The verifier rebuilds the object (a sorting network, a coloring, a matrix…) and
checks it directly, usually in seconds, without trusting the solver or the CNF. Each ladder has both
known-SAT and known-UNSAT rungs. A solver can show its speed on instances whose models anyone can check
instantly, and a wrong SAT claim on a known-UNSAT rung is caught at once. UNSAT answers are the exception: they
need a DRAT/LRAT proof to be checkable (see below).

## Getting the instances

Instances are in each sequence's `cnf/` folder, compressed with xz, with a `SHA256SUMS` of the uncompressed
files. Any instance can also be regenerated locally with the sequence's generator (see its `make_set.sh`); the
output is deterministic and matches the checksums. Future instances over GitHub's 100 MB file limit (compressed)
will be attached to a release tagged `A######-v1` instead.

```bash
xz -dk sortnet_n18_d10.cnf.xz      # -> DIMACS CNF
```

## Claiming a result

* **SAT:** run the sequence's verifier on the solver's model. It extracts the object and checks it without
  trusting the CNF. A verified model is a complete certificate.
* **UNSAT:** an UNSAT answer by itself cannot be checked. Produce a DRAT or LRAT proof and check it with an
  independent checker (`drat-trim`, `cake_lpr`, `lrat-check`). Then please open an issue with the solver,
  version, runtime and the proof's size and checksum.

## Layout and adding a sequence

```
sequences/
  A067782/
    README.md          question, encoding, what SAT/UNSAT mean, calibration table, references
    gen_*.py           generator: parameters -> DIMACS CNF (+ a .map.json for decoding)
    decode_verify.py   model -> object -> independent check
    validate.py        regenerates small cases and checks them against known OEIS terms
    make_set.sh        exact commands that produced the published instances
    cnf/               *.cnf.xz, *.map.json, SHA256SUMS
```

To add a sequence, create `sequences/A######/` with the same pieces and add a row to the table above (instances
over 100 MB compressed go to a release tagged `A######-v1`). Instance files are named
`<problem>_<parameters>.cnf`, and their header comment lines (`c ...`) say what the instance asks.

## License

Code: MIT (see `LICENSE`). The generated CNF instances are released into the public domain (CC0).

Maintained by James Guillochon.
