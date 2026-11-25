# Contributing

1. Fork and create a branch.
2. `node server.js` to run locally (Node 18+, no dependencies to install).
3. `npm test` must pass. New API behaviour needs a test in `test/server.test.js`.
4. Keep the CSP intact: no inline `<script>`, `style=` attributes or `on*=` handlers (a test enforces this).
5. Open a pull request describing the change.
