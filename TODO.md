# TODO

- [ ] **Check `AGENTS.md` < 24,000 bytes in CI** (mikelward/stopdash#932):
      assert `AGENTS.md` stays under 24,000 bytes, beside the front-matter
      checks in `agents-front-matter.test.js` (an `AGENTS.md` edit already
      rides the code lane), so it can't silently grow past Antigravity's
      24,000-byte per-file rule cap, which truncates the tail.
