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
| `/etc/apt/sources.list.d/picompose.sources` | Source entry for the suite `trixie`. Edit `Suites:` to use `trixie-nightly` or `trixie-test`. It is a conffile, so edits survive upgrades. |

## Updating the key

The file `picompose-archive-keyring.gpg` is the public key published by the repository at `https://florian-asche.github.io/PiCompose-Repository/picompose-archive-keyring.gpg`. When the signing key is extended or replaced:

1. Download the new file over `picompose-archive-keyring.gpg`.
2. Raise the version in `debian/changelog` and release it.
3. Release it before the old key expires, so devices receive it by `apt upgrade` while they still trust the old one. While both keys are valid the repository can be signed with both, see the key documentation of the APT repository.

## Local build

```sh
sudo apt install build-essential debhelper devscripts fakeroot lintian
dpkg-buildpackage -us -uc -b
lintian ../picompose-archive-keyring_*.deb
```

## Releases

A tag `v<version>` that matches the first entry of `debian/changelog` publishes the package to the suite `trixie`. Pushes to `main` go to `trixie-nightly` and pushes to the branch `test` to `trixie-test`.

## License

This project is released under the [MIT License](LICENSE).
