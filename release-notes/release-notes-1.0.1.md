# File::DistDir 1.0.1 Release Notes

## Overview

This release adds two new capabilities: the ability to construct a
`File::DistDir` object by naming an installed Perl module rather than
a distribution, and a `find` method and corresponding CLI command for
pattern-based file search within a share directory. A new runtime
dependency on `Module::Metadata` is introduced.

---

## New Features

### Module-Based Constructor

`File::DistDir->new` now accepts a `module` parameter as an
alternative to `distribution`:

```perl
my $dist = File::DistDir->new( module => 'BLM::Startup::RedisSession' );
```

The module name is validated syntactically, then resolved to its
installed path via `Module::Metadata`. The owning distribution is
identified by walking all `ExtUtils::Installed` packlists. However, before
traversing packlists convert module name to a distribution name and
see if that exists. The constructor dies if the module is not
installed, if no packlist claims ownership, or if more than one
distribution claims the same file. The resolved distribution name is
derived by converting `::` to `-` in the top-level package name from
the packlist.

Exactly one of `distribution` and `module` must be supplied; providing
both is an error.

### `find(PATTERN)` Method

A new `find` method searches the share directory recursively for
regular files whose names match a simple glob pattern supporting `*`
and `?`. Symlinks and directories are excluded from results.

```perl
my @xml_files = $dist->find('*.xml');
```

If the pattern contains a `/`, the entire relative path is matched
rather than just the basename.

### `find` CLI Command

The `dist-dir` command gains a `find` subcommand that delegates to
`File::DistDir->find` and prints one relative path per line:

```
dist-dir -d MyDist find '*.xml'
dist-dir -m My::Module find 'config/*.yml'
```

### `--module` / `-m` CLI Option

The CLI now accepts `--module`/`-m` as a mutually exclusive
alternative to `--distribution`/`-d`. Previously, `--distribution`
was mandatory. All subcommands that previously required `--distribution`
now accept either option.

---

## Dependency Changes

`Module::Metadata` (>= 1.000038) is now a declared runtime dependency,
added to `requires` and `cpanfile`. It is used by the module-based
constructor to locate an installed module's path before packlist
scanning.

---

## Build Changes

### `MODULINO_ENABLED` Guard in `bash-completion.mk`

The `MODULINO` variable assignment, its build rule, and the
`bash-completion` Make target are now conditional on
`MODULINO_ENABLED=1`. The top-level `Makefile` sets
`MODULINO_ENABLED ?= 1`, preserving existing default behavior while
allowing projects to opt out cleanly.

### LLM Role Files

Two new files, `.roles/CPAN-Maker-Bootstrapper-Role-LLM/role.mk` and
`.roles/CPAN-Maker-Bootstrapper-Role-LLM/role.yml`, and a new prompt
file `.prompts/change-log.prompt` have been added to the repository.
These are build-tooling and workflow support files; they are not
distributed in the CPAN tarball and do not affect the installed module.

---

## Test Coverage

A new test file `t/02-find-module.t` covers module-based construction
and `find` behavior.

---

## ChangeLog Review

No inconsistencies or errors were found. One minor observation:

- The ChangeLog entry for `.roles/CPAN-Maker-Bootstrapper-Role-LLM/`
  lists the new dependency (`+Module::Metadata 1.000038`) under those
  role files rather than under `requires` or `cpanfile`, which could
  mislead a reader into thinking that entry is scoped to the role
  rather than being a general runtime requirement. The dependency is
  correctly recorded under `requires` as well, so this is a formatting
  ambiguity rather than an omission.

- `.prompts/change-log.prompt` appears in the git status as an added
  file but is not mentioned in the ChangeLog. It is a developer
  tooling file with no effect on the distribution.

---

**Commit message:** Add module-based constructor and `find` command to
`File::DistDir`, with a new `Module::Metadata` runtime dependency.
