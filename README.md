# PiCompose-Package-Archive-Keyring

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

The version of the package is the first entry of `debian/changelog`, which is part of the package. You do not edit it by hand, the workflows write it.

1. **Pull request:** Every pull request needs exactly one of the labels `major`, `minor` and `patch`. It decides how the version is raised when it is merged. Tick one box under "Version bump" in the description or set the label by hand. The check "Version label" fails until there is exactly one.
2. **Merge to `main`:** The workflow raises the version in `debian/changelog` by the highest label of the merged pull requests, adds the titles of the pull requests as changelog entry, commits `debian/changelog` to `main` (the commit "Version 1.3.0"), builds exactly that version and publishes it to the suite `trixie-nightly`. No tag is created.
3. **Release:** Start the workflow **Release** (Actions, Release, Run workflow). It tags the newest commit of `main` with `v<version>` of `debian/changelog`, creates the GitHub release and publishes that version to the suite `trixie`. Tags only exist for releases. A release created by hand in the GitHub UI is not published.
4. **Any other branch:** The package gets the version `<version>+dev<timestamp>.<sha>`. Nothing is published, the packages are kept as workflow artifacts for 14 days. Use a branch per change to try it on a device.

## License

This project is released under the [MIT License](LICENSE).
