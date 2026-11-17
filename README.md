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
2. Run the workflow **Release** with a higher version.
3. Release it before the old key expires, so devices receive it by `apt upgrade` while they still trust the old one. While both keys are valid the repository can be signed with both, see the key documentation of the APT repository.

## Local build

```sh
sudo apt install build-essential debhelper devscripts fakeroot lintian
dpkg-buildpackage -us -uc -b
lintian ../picompose-archive-keyring_*.deb
```

## Releases

`debian/changelog` holds the releases and is part of the package. You do not edit it by hand, the release workflow writes it.

- **Release:** Start the workflow **Release** (Actions, Release, Run workflow) and enter a version like `1.2.3` that is higher than the last release. It writes the changelog entry from the notes GitHub generates from the merged pull requests, commits `debian/changelog` to `main`, tags that commit `v<version>`, creates the GitHub release and builds and publishes the package to the suite `trixie`. A release created by hand in the GitHub UI is not published.
- **Push to `main`:** The package gets the version `<last release>+git<timestamp>.<sha>` and is published to the suite `trixie-nightly`. The changelog entry of this build, made from the title and description of the pull request, is only part of the package. Every build overwrites the previous one.
- **Any other branch:** `<last release>+dev<timestamp>.<sha>`. Nothing is published, the packages are kept as workflow artifacts for 14 days. Use a branch per change to try it on a device.

## License

This project is released under the [MIT License](LICENSE).
