# Governance

Model Diet is an early-stage, pre-1.0 open-source project. Its repository is
hosted in the `modeldiet` GitHub organization. At the 0.1.0 release, PyPI lists
one maintainer, and the repository does not publish a multi-maintainer
governance roster. This document describes the current lightweight process;
it does not imply that a foundation, company, or formal steering group
operates the project.

## Maintainer responsibility

The current maintainer is responsible for project direction, repository
administration, reviewing and merging changes, responding to issues, and
preparing releases. The maintainer should follow the published contribution,
security, and code-of-conduct policies.

## Technical decisions

Propose substantial technical changes in a GitHub issue or pull request with
the problem, alternatives, compatibility effects, and validation evidence.
Small fixes can be proposed directly as pull requests. The maintainer evaluates
changes for correctness, scope, compatibility, maintenance cost, and alignment
with the documented API. Discussion is welcome; the maintainer makes the final
decision while the project has a single maintainer.

## Issues and pull requests

Bug reports should include a minimal reproduction, expected behavior, observed
behavior, and relevant Python and dependency versions. Pull requests should be
focused, tested, and explain user-visible effects. Maintainers may ask for
changes, defer work, or close proposals that are out of scope or lack enough
information.

## Releases

The maintainer chooses release contents and timing, checks tests and package
artifacts, updates the changelog and version, and publishes distributions.
Git tags and GitHub Releases are created deliberately; merging code alone does
not promise a release. Users should report release problems through GitHub
issues, except security issues, which must follow [SECURITY.md](SECURITY.md).

## Security

Please report suspected vulnerabilities privately using the process in
[SECURITY.md](SECURITY.md). Do not post exploit details in a public issue before
maintainers have had a chance to assess the report.

## Future maintainers and governance changes

The maintainer may invite contributors to take on review, triage, or release
responsibilities based on sustained, constructive contributions and mutual
agreement. As the contributor base grows, maintainers may propose a more formal
decision process in an issue or pull request. Changes to this policy should be
discussed publicly and recorded in the repository.
