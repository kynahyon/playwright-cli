## Commit Convention

Semantic commit messages: `label(scope): description`

Labels: `fix`, `feat`, `chore`, `docs`, `test`, `devops`

```bash
git checkout -b fix-39562
# ... make changes ...
git add <changed-files>
git commit -m "$(cat <<'EOF'
fix(proxy): handle SOCKS proxy authentication

Fixes: https://github.com/microsoft/playwright/issues/39562
EOF
)"
git push origin fix-39562
gh pr create --repo microsoft/playwright --head username:fix-39562 \
  --title "fix(proxy): handle SOCKS proxy authentication" \
  --body "$(cat <<'EOF'
## Summary
- <describe the change very! briefly>

Fixes https://github.com/microsoft/playwright/issues/39562
EOF
)"
```

Never add Co-Authored-By agents in commit message.
Branch naming for issue fixes: `fix-<issue-number>`
Branch naming for features: `feat-<short-description>`

## Personal Notes

- Upstream repo: https://github.com/microsoft/playwright-cli
- My fork is for learning browser automation internals; not intended for production use.
- Sync with upstream periodically: `git fetch upstream && git rebase upstream/main`
- Focus areas: proxy handling, network interception, and CDP protocol internals.
- When debugging CDP messages, use `DEBUG=pw:protocol` env var to dump raw protocol traffic.
- When debugging network interception specifically, `DEBUG=pw:network` is more focused and less noisy.
- Useful combo for proxy issues: `DEBUG=pw:protocol,pw:network node script.js 2>&1 | grep -i proxy`
- Useful combo for request interception: `DEBUG=pw:network node script.js 2>&1 | grep -i intercept`
- For verbose chromium-level network logs: set `PWDEBUG=1` alongside `DEBUG=pw:network` to get browser console output too.
- For WebSocket/CDP frame inspection: `DEBUG=pw:protocol node script.js 2>&1 | grep -i websocket`
- Handy alias to add to shell profile: `alias pwdebug='DEBUG=pw:protocol,pw:network PWDEBUG=1'`
- For filtering just SOCKS-related traffic: `DEBUG=pw:protocol,pw:network node script.js 2>&1 | grep -iE 'socks|auth|407'`
- Node version in use: 20 LTS (nvm alias default 20); some CDP internals differ on Node 18 — keep this in mind when testing.
- Chromium version in use: check with `npx playwright --version`; CDP differences between Chromium releases can affect proxy auth behavior.
- When a test fails only in CI, check if the runner has a different proxy env (HTTP_PROXY/NO_PROXY vars) leaking into the process.
- For inspecting raw HAR output during network interception work: `npx playwright --save-har=trace.har` then open in browser devtools network panel.
- Reminder: `page.route()` intercepts at the browser level; `browserContext.route()` applies to all pages in the context — easy to mix these up.
- For replaying a saved HAR file in tests: `await context.routeFromHAR('trace.har', { update: false })` — useful for offline/reproducible proxy auth testing.
- For a quick local SOCKS5 proxy to test against: `ssh -D 1080 -N localhost` spins one up on port 1080 without needing extra tooling.
- To verify a SOCKS5 proxy is working: `curl --socks5 localhost:1080 https://example.com` — quick sanity check before running playwright tests against it.
- To test proxy auth failure scenarios locally: `ssh -D 1080 -N localhost` doesn't require auth, so use `dante` or `microsocks` with a config if you need 407 responses.
