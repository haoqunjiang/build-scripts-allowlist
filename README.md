# build-scripts-allowlist

An up-to-date list of common NPM packages that need to run lifecycle scripts during installation.

## FAQ

### How can I use the list?

This package only supports PNPM 10. We are working on supporting other package managers through `@lavamoat/allow-scripts`.

In your project using PNPM 10, add the following field to your `package.json`:

```json
"pnpm": {
  "configDependencies": {
    "build-scripts-allowlist": "0.20250131.0+sha512-vuePnd+0F3PtuyFIHT1XsfOK7/BGkK1QmIMRHHPo+6LW7Wr/37QsQV3R/UztxoFWHHm9pSbCUovigSmeapfdLA=="
  },
  "onlyBuiltDependenciesFile": "node_modules/.pnpm-config/build-scripts-allowlist/common.json"
}
```

This package uses a date-based version number to indicate the last time the list was updated so you can easily see if the list is outdated.

> [!IMPORTANT]
> If you are reading the documentation on NPM, please go to the [GitHub repository](https://github.com/haoqunjiang/build-scripts-allowlist) to see the up-to-date version number & hash used in the `configDependencies` field.

### What are build scripts? Are they dangerous? Why do we need to block them?

Build scripts are scripts that run during the installation of a package. They are defined in the `package.json` file of the package. The most common build scripts are `preinstall`, `install`, and `postinstall`[^1], which will be executed in that order. These scripts are used to set up the package, compile native code, download binary files, etc.

In practice, the scripts can do anything, including downloading and executing arbitrary code from the internet. This makes them a security risk.

PNPM 10 and Bun block build scripts by default, and both have mechanisms to allowlist packages that need to run build scripts.

### Why do we need an allowlist?

Blocking build scripts by default is a good security measure, but it can break packages that rely on build scripts for their essential functionality. For example, packages that download binary files or compile native code will not work if their build scripts are blocked.

By allowlisting the packages that with build scripts essential for their functionality and are known to do no harm, we can block build scripts for all other packages while still allowing these packages to work.

Of course this is not a perfect solution. The allowlist can be incomplete, and currently benign packages can be compromised in the future. Also, the allowlist can be used to smuggle malicious code into the project by adding a malicious package to the allowlist. So it is important to keep the allowlist up-to-date and to only add packages that are widely used and have a good reputation.

### Does the list include all packages that run build scripts?

No.

- Only packages that are commonly used in production environments are included. There's currently no strict criteria for inclusion. The initial list is a trimmed down version of the list of packages that are included in [Bun's `default-trusted-dependencies.txt`](https://github.com/oven-sh/bun/blob/c0e1da7280a3cd58796dd09696767e119de58ec1/src/install/default-trusted-dependencies.txt).
- Only those with build scripts that are essential for the package to work are included. Build scripts that are used for optional features or for development purposes are not considered essential.

### How is the list maintained?

The list is maintained manually. If you find a package that should be included, please open an issue or a pull request.

### Common Use Cases of Postinstall Scripts and Alternative Methods

- Download binary files. Alternative: optional dependencies with platform-specific binaries.
- Check for dependency version compatibility. Alternative: `peerDependencies`. For monorepos, use PNPM `catalog`, [Yarn Constraints](https://yarnpkg.com/features/constraints). Also, run time checks are still needed even with postinstall checks, so postinstall checks should be safe to remove.
- Install tooling in local development environment. Alternative: encourage users to explicitly set a `prepare` script in their `package.json` to install the tooling.
- Printing funding or attribution messages. Alternative: configure the [`funding` field in `package.json`](https://docs.npmjs.com/cli/v11/configuring-npm/package-json#funding) to let `npm` print a "x packages are looking for funding." message. Anyway, this is not a critical feature and should not be a blocker for installing the package.

## Inspirations

- Bun's documentation on [`trustedDependencies`](https://bun.sh/docs/install/lifecycle#trusteddependencies) and its [`default-trusted-dependencies.txt`](https://github.com/oven-sh/bun/blob/c0e1da7280a3cd58796dd09696767e119de58ec1/src/install/default-trusted-dependencies.txt)
- [`can-i-ignore-scripts`](https://github.com/naugtur/can-i-ignore-scripts)
- [`@lavamoat/allow-scripts`](https://www.npmjs.com/package/@lavamoat/allow-scripts)
- [PNPM's discussion thread on blocking lifecycle scripts by default](https://github.com/orgs/pnpm/discussions/8918)

[^1]: Note that this list does not screen packages with only `prepare` scripts, because `prepare` scripts are run only when the package is installed as a dependency from a git repository, not from the npm registry.
