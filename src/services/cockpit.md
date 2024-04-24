# Cockpit

Cockpit is a web-based server manager that makes it easy to administer your GNU/Linux servers via a web browser. It is a free and open-source software that provides a friendly interface for system administrators.

Cockpit can be accessed at [secureroamcompanion.org](http://secureroamcompanion.org).

## Configuration

To configure Cockpit, follow these steps:

1. Open a web browser and navigate to [secureroamcompanion.org](http://secureroamcompanion.org).
2. Log in with the default credentials:
   - Username: `secureroamcompanion`
   - Password: `test`
3. Follow the on-screen instructions to configure Cockpit.

For more information on how to configure Cockpit, see the [Cockpit documentation](https://cockpit-project.org/).

## Development

The Cockpit Project is a software interface that is divided into various packages, each providing specific features and/or code. Each package consists of one or more files placed in a directory or its subdirectories, with a mandatory `manifest.json` file. The layout of package files follows certain naming conventions and uses data directories from the XDG Base Directory Specification to locate packages.

The `manifest.json` file contains information about the package such as its name, priority, version, content security policy, and requirements. It also specifies where components of the package should appear in Cockpit's user interface.

Cockpit provides a modular architecture that allows you to extend its functionality by adding new modules. To add a new module to Cockpit, follow these steps:

1. Install `gettext`, `nodejs`, `npm` and `make` packages on your system.
2. Clone the Cockpit repository from GitHub:

   ```bash
   git clone https://github.com/cockpit-project/starter-kit.git
   cd starter-kit
   make
   ```

3. Edit the `manifest.json` file to define the module's name, description, and other metadata.
4. Add your module's code to the `src` directory.
5. Build the module:

   ```bash
   make
   ```

Up-to-date information on how to add a new module to Cockpit can be found in the [Cockpit documentation](https://cockpit-project.org/guide/latest/development.html).

### Make commands

- `make`: Builds the module.
- `make watch`: Watches for changes in the module's code and rebuilds the module automatically.
- `make clean`: Removes the build artifacts.
- `make install`: Installs the module on your system.
- `make devel-install`: Installs the module for development purposes.
- `make devel-uninstall`: Uninstalls the module for development purposes.
- `make dist`: Creates a distribution tarball.
- `make srpm`: Creates a source RPM package.
- `make rpm`: Creates a binary RPM package.
- `make vm`: Builds a virtual machine image for testing.
- `make check`: Runs the browser integration tests.
- `make codecheck`: Runs the static code analysis.
