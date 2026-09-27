This is a repo containing libraries for Silverbullet (an extensible note taking app) written in space-lua.
- All libraries are single-file by default: logic, style and whatever else is needed lives in one library page.
- Additional pages or files (e.g. page templates) may only be shipped alongside a library via the official `files:` frontmatter mechanism; they must live in this repo at the same folder level as the library page.
- All libraries must be in the list in `REPO.md`
- Library URIs pointing at nested repo paths must use the full `https://github.com/owner/repo/blob/main/path` form; the `github:` short scheme only supports top-level files (its handler drops path segments)