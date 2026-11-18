# PiCompose-Package-Archive-Keyring

[![Version](https://github.com/florian-asche/PiCompose-Package-Archive-Keyring/actions/workflows/version.yml/badge.svg?branch=main)](https://github.com/florian-asche/PiCompose-Package-Archive-Keyring/actions/workflows/version.yml) [![GitHub Release Version](https://img.shields.io/github/v/release/florian-asche/PiCompose-Package-Archive-Keyring?label=version)](https://github.com/florian-asche/PiCompose-Package-Archive-Keyring/releases) [![GitHub License](https://img.shields.io/github/license/florian-asche/PiCompose-Package-Archive-Keyring)](https://github.com/florian-asche/PiCompose-Package-Archive-Keyring/blob/main/LICENSE)

Package `picompose-archive-keyring` for the [PiCompose APT repository](https://github.com/florian-asche/PiCompose-Repository). It installs the GPG key that signs the repository and the apt source entry for the stable suite `trixie`.

Because the key is a package, it can be renewed or replaced with a normal `apt upgrade`. Without it, every device would need the new key copied by hand.

## Install

The first install needs the key and the source entry by hand, see [docs/apt_repository.md](https://github.com/florian-asche/PiCompose-Repository/blob/main/docs/apt_repository.md). Afterwards:

```sh
sudo apt install picompose-archive-keyring
```

If you created `/etc/apt/sources.list.d/picompose.sources` yourself, dpkg asks whether to replace it. In scripts and image builds use `-o Dpkg::Options::=--force-confnew`.

## Files

| File | Content |
| --- | --- |
| `/usr/share/keyrings/picompose-archive-keyring.gpg` | Public key of the repository, referenced by `Signed-By` |
| `/etc/apt/sources.list.d/picompose.sources` | Source entry for the suite `trixie`. Edit `Suites:` to use `trixie-nightly`. It is a conffile, so edits survive upgrades. |

## Updating the key

The file `picompose-archive-keyring.gpg` is the public key published by the repository at `https://florian-asche.github.io/PiCompose-Repository/picompose-archive-keyring.gpg`. When the signing key is extended or replaced:

1. Download the new file over `picompose-archive-keyring.gpg` and merge it with a pull request.
2. Merge the pull request with the label `minor` or `patch`, then run the workflow **Release**.
3. Release it before the old key expires, so devices receive it by `apt upgrade` while they still trust the old one. While both keys are valid the repository can be signed with both, see the key documentation of the APT repository.

## Local build

```sh
sudo apt install build-essential debhelper devscripts fakeroot lintian
dpkg-buildpackage -us -uc -b
lintian ../picompose-archive-keyring_*.deb
```

## Releases

The version of the package is the first entry of `debian/changelog`, which is part of the package (`apt changelog picompose-archive-keyring` shows the whole history). You do not edit it by hand, the workflows write it.

1. **Pull request:** Every pull request needs exactly one of the labels `major`, `minor` and `patch`. It decides how the version is raised when it is merged. Tick one box under "Version bump" in the description or set the label by hand. The check "Version label" fails until there is exactly one. Make it required in the branch protection of `main`.
2. **Merge to `main`:** The workflow "Version" raises the version by the highest label of the merged pull requests, adds the pull request titles to `debian/changelog` and commits it to `main` (the commit "Version 1.3.0"). It fails if a pull request has no or more than one label, or if a commit has no pull request. Start it by hand and choose a level to get past that.
3. **Pre-release:** The workflow creates the tag `v1.3.0` as a **pre-release**, with the release notes that GitHub generates from the template in `.github/release.yml` for the pull requests since the last tag.
4. **Build:** The workflow "Build and publish a version" is started on the tag. It builds the package from the tag and publishes it to the suite `trixie-nightly`.
5. **Release:** In the GitHub UI, open the pre-release and turn it into a release (untick "Set as a pre-release"). That starts the workflow "Build and publish a version" for the release, which publishes the same version to the suite `trixie` and generates the release notes again for all pull requests since the last release.
6. **Any other branch:** The package gets the version `<version>+dev<timestamp>.<sha>`. Nothing is published, the packages are kept as workflow artifacts for 14 days. Use a branch per change to try it on a device.

## License

This project is released under the [MIT License](LICENSE).

The key file is the public key that the PiCompose APT repository publishes.
