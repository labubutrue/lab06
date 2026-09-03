# Laboratory work VI

Packaging and release automation for the `solver` application.

The project uses CMake and CPack to generate:

- source archives: `.tar.gz`, `.zip`;
- Linux packages: `.deb`, `.rpm`;
- Windows package: `.msi`;
- macOS package: `.dmg`.

Packages are built automatically by GitHub Actions on version tags and attached to a GitHub Release.
