# google-workspace-mcp

A standalone copy of the Google Workspace MCP server from
[gemini-cli-extensions/workspace](https://github.com/gemini-cli-extensions/workspace)
with one local patch: **direct OAuth mode**.

The original repository is a Gemini CLI extension. This repository keeps
only the MCP server (`workspace-server/`) and the build, lint and test
configuration it needs, so it can be used with any MCP client (Claude
Code, Codex, DeepSeek Harness, ...). For documentation of the tools
themselves, see the original repository.

## History

- `chore: import workspace-server from upstream 089927e` — upstream code
  as-is, without the Gemini CLI extension files (commands, skills, docs,
  cloud function).
- `feat: direct OAuth mode without cloud function` — the patch.

Everything else is upstream code. To update, re-import a newer upstream
commit and re-apply the patch.

## The patch: direct OAuth mode

Upstream ships a public OAuth client ID without a secret. Token exchange
and refresh go through the authors' Google Cloud Function, which holds
the secret. Your OAuth codes and refresh tokens therefore pass through a
third-party service.

With the patch, if `WORKSPACE_CLIENT_SECRET` is set, the server talks to
Google directly using your own OAuth client:

- sign-in uses the loopback redirect (`http://localhost:<port>`) instead
  of the cloud function;
- access tokens are refreshed with Google locally;
- without a GUI browser, the sign-in URL is printed instead of failing
  (useful on WSL2 and over SSH with port forwarding);
- `headless-login` runs the same direct flow.

Without `WORKSPACE_CLIENT_SECRET` the server behaves exactly like
upstream.

Changed files: `workspace-server/src/auth/AuthManager.ts`,
`workspace-server/src/cli/headless-login.ts`,
`workspace-server/src/utils/config.ts`.

## Usage

1. In Google Cloud Console, enable the APIs you need (Drive, Docs,
   Sheets, Slides, Calendar, ...) and create an OAuth client of type
   **Desktop app**.
2. Build:

   ```sh
   npm ci
   npm run build
   ```

3. Set the client credentials and sign in once:

   ```sh
   export WORKSPACE_CLIENT_ID=<client id>
   export WORKSPACE_CLIENT_SECRET=<client secret>
   node workspace-server/dist/headless-login.js        # --force to re-auth
   ```

4. Register the server in your MCP client as a stdio command with the
   same environment, for example in Claude Code:

   ```sh
   claude mcp add -s user google-workspace \
     -e WORKSPACE_CLIENT_ID=... -e WORKSPACE_CLIENT_SECRET=... \
     -- node /path/to/workspace-server/dist/index.js --use-dot-names
   ```

Useful upstream options:

- `WORKSPACE_FEATURE_OVERRIDES` — enable or disable tool groups, e.g.
  `gmail.read:off,gmail.write:off,chat.read:off,chat.write:off`;
- `GEMINI_CLI_WORKSPACE_FORCE_FILE_STORAGE=true` — store tokens in a
  file instead of the OS keychain (headless machines).

## License

Apache License 2.0, same as upstream; see [LICENSE](LICENSE). The files
listed above were modified from the original.
