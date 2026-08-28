# Changelog

## [1.7.0](https://github.com/swiftaspect/gha-workflows/compare/v1.6.0...v1.7.0) (2026-08-28)


### Features

* move a `v1` tag onto every release ([788bd1d](https://github.com/swiftaspect/gha-workflows/commit/788bd1dd012d926c5b732e708a02312a56a4b527))


### Bug Fixes

* grant `checks: read` so the merge gate can read the check runs ([9a47542](https://github.com/swiftaspect/gha-workflows/commit/9a47542fe50ea0fede65913af05e182929ca964b))

## [1.6.0](https://github.com/swiftaspect/gha-workflows/compare/v1.5.0...v1.6.0) (2026-08-28)


### Features

* add a reusable dependabot auto-merge workflow ([715076a](https://github.com/swiftaspect/gha-workflows/commit/715076a8807422ef3707bf49369fd003eb86f542))

## [1.5.0](https://github.com/swiftaspect/gha-workflows/compare/v1.4.4...v1.5.0) (2026-07-03)


### Features

* Delegate the PR-gate manifest check to make check-prerelease-deps ([af48bfe](https://github.com/swiftaspect/gha-workflows/commit/af48bfee1d1bf80cbf08b2383587e03aa187b341))

## [1.4.4](https://github.com/swiftaspect/gha-workflows/compare/v1.4.3...v1.4.4) (2026-06-22)


### Bug Fixes

* **publish-tag:** Was missing necessary variable passing for this workflow to work ([c94b994](https://github.com/swiftaspect/gha-workflows/commit/c94b9949783f1954cba6608cdb3af2aa8789dbcc))

## [1.4.3](https://github.com/swiftaspect/gha-workflows/compare/v1.4.2...v1.4.3) (2026-06-08)


### Bug Fixes

* **ci:** Add GITHUB_TOKEN to CI lint and test step ([77052c3](https://github.com/swiftaspect/gha-workflows/commit/77052c3517c83d3adb6e906ac90c61c2c086cbb0))

## [1.4.2](https://github.com/swiftaspect/gha-workflows/compare/v1.4.1...v1.4.2) (2026-05-27)


### Bug Fixes

* **ci:** The `make start` for CI will sometimes need to pull from an authenticated registry just like docker ([134a019](https://github.com/swiftaspect/gha-workflows/commit/134a0194907dfbcf469ccd1f6f7dd68b3771e7ec))

## [1.4.1](https://github.com/swiftaspect/gha-workflows/compare/v1.4.0...v1.4.1) (2026-05-27)


### Bug Fixes

* **publish-tag:** The publish workflows require GITHUB_TOKEN for auth to the registries ([cda69bb](https://github.com/swiftaspect/gha-workflows/commit/cda69bbf230e1f920ee7ca031da1348518672f2b))

## [1.4.0](https://github.com/swiftaspect/gha-workflows/compare/v1.3.1...v1.4.0) (2026-05-25)


### Features

* convert GHA runs-on to specific tagged ubuntu release ([46ae633](https://github.com/swiftaspect/gha-workflows/commit/46ae633474fe1dd29086847f726f1064d2793498))


### Bug Fixes

* **ci:** Authenticate to container registry to pull and push images ([9d5b32c](https://github.com/swiftaspect/gha-workflows/commit/9d5b32cbb3de0d0997b741768828c138461e3b52))
* Update googleapis/release-please-action to latest tagged release ([b42e923](https://github.com/swiftaspect/gha-workflows/commit/b42e923fa023af64905b95b1158f93cf26dece21))

## [1.3.1](https://github.com/swiftaspect/gha-workflows/compare/v1.3.0...v1.3.1) (2026-05-18)


### Bug Fixes

* how does this work if the registry isnt authenticated to? ([dcfd864](https://github.com/swiftaspect/gha-workflows/commit/dcfd864c7fa54abc0e27978b1d92616d92a24402))

## [1.3.0](https://github.com/swiftaspect/gha-workflows/compare/v1.2.0...v1.3.0) (2026-05-18)


### Features

* simplify publishing workflows to be more agnostic ([cd0b45e](https://github.com/swiftaspect/gha-workflows/commit/cd0b45e624d26ca2aa11c6a6c4ac0115d13dc538))

## [1.2.0](https://github.com/swiftaspect/gha-workflows/compare/v1.1.1...v1.2.0) (2026-05-18)


### Features

* **ci:** Move to using `make publish` as language agnostic publishing ([166dede](https://github.com/swiftaspect/gha-workflows/commit/166dedee99f87af89dc8ea355f1a8b1d295cf62e))

## [1.1.1](https://github.com/swiftaspect/gha-workflows/compare/v1.1.0...v1.1.1) (2026-04-22)


### Bug Fixes

* **ci.yml:** Pre-release to node requires additional access ([1184a40](https://github.com/swiftaspect/gha-workflows/commit/1184a40994bd02807832b573393b01a050be0366))

## [1.1.0](https://github.com/swiftaspect/gha-workflows/compare/v1.0.0...v1.1.0) (2026-04-15)


### Features

* Support overriding makefile settings specifically in CI ([5b39190](https://github.com/swiftaspect/gha-workflows/commit/5b391909806af0c37514c15ad2029fa7559f0b25))

## 1.0.0 (2026-04-14)


### Features

* initial reusable workflows ([0a322f7](https://github.com/swiftaspect/gha-workflows/commit/0a322f7fc06b92f16205a21b3b6d96ba9d96b98b))


### Bug Fixes

* **release.yml:** restore --repo flag on gh pr merge ([9741cfe](https://github.com/swiftaspect/gha-workflows/commit/9741cfe8a8bd384afcf314c74c8fa7fdd0cf14cd))
