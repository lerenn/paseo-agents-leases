# Agents Leases

A [Paseo](https://paseo.sh) plugin: agents take named leases with timeouts, waiting queues and takeover, so only one agent at a time works on a PR, a QA environment or any shared resource. See [OVERVIEW.md](OVERVIEW.md).

## Development

```sh
npm ci
npm run typecheck && npm test && npm run build
```

CI runs the same through Dagger (`dagger call all`) in a `ci` GitHub Actions job.

## License

MIT
