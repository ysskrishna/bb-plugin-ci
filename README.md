# bb-plugin-ci

Shared GitHub Actions workflows for [BB](https://www.npmjs.com/package/bb-app) plugins.

The first workflow will warn a plugin author before a new BB release falls outside
the plugin's `engines.bb` range: it runs the plugin's checks against that BB and
opens one `bb-compat` issue with the result. Usage is documented here once the
workflow lands.

## License

[MIT](LICENSE)
