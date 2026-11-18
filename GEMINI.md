# JAS: guide for AI coding assistants (Gemini and others)

JAS is a planning workspace by Jancarl Angelo H. Santiago: sticky notes, shapes, arrows, tables, charts and drawing on one board, with live collaboration. It is hosted on GitHub Pages at https://jancarlsantiago2-blip.github.io/jas/ and every push to `main` goes live within a minute or two.

## Ground rules

1. **Never put secret keys in this repo.** The repo is public. Only these may appear in code: the Supabase project URL, the Supabase publishable (anon) key, and the Google OAuth Client ID. Never add the Supabase `service_role`/secret key, a Google Client secret, passwords, or anyone's email address.
2. **Security lives in the database, not the page.** Row Level Security (RLS) in Supabase decides who can read or change what. Hiding a button is not security. If a change needs new permissions, write SQL (see "Database changes").
3. **Don't break saved boards.** Board data is JSON in `boards.data`. Add new fields with safe defaults; never rename or drop existing item fields (`id, type, x, y, w, h, z, text, color, fs, label, shape, lock, g, rows, kind, src, pts, vw, vh, sw, hl`).
4. **Keep it one file, no build step.** `index.html` is plain HTML, CSS and JavaScript with no framework and no bundler. Keep it that way so GitHub Pages can serve it directly.
5. Test on a phone width (390 px) as well as desktop before pushing.

## Files

| File | What it is |
|---|---|
| `index.html` | The whole app: sign in, About page, Boards, Feedback, College Recon, and the board workspace. |
| `admin.html` | Admin panel (users, sign-ups, subscribers, feedback, admins). Only works for accounts listed as admins. |
| `classroom-test.html` | Stand-alone test page for the Google Classroom connection. |
| `jas-mcp.txt` | Source of the Supabase Edge Function `jas-mcp` (the AI connector / MCP server). Deployed separately to Supabase with "Verify JWT" off. |
| `setup.txt`, `admin-setup.txt`, `finish-setup.txt`, `roles-setup.txt`, `profile-feedback-setup.txt` | SQL that has been (or must be) run in the Supabase SQL editor, in that order. |
| `logo/` | Logo files. |

## How index.html is organised

Search for these comment headings or names to find each part:

- **CSS tokens** at the top in `:root`: the palette is lime `#CCFF00`/`#E3EF26` on forest greens `#06231D`, `#0C342C`, `#076653`, cream text `#FFFDEE`. Gradients are `--grad` and `--grad-deep`. The board canvas itself stays light by default so notes keep their colours.
- **Screens**: `#auth` (sign in/up), `#lobby` (Page 1, About), `#app` with views `#v-home` (Page 2, Boards), `#v-feedback` (Page 3), `#v-recon` (College Recon), and `#ws` (the board workspace). The left sidebar is `#side`. `showScreen()`, `showView()` and `sideActive()` switch between them.
- **Cloud**: `CLOUD_CONFIG`, `initCloud()`, `DB`, `cloudSave()`, `mergeRemote()`, `startSync()`. Saves use optimistic concurrency on `data._rev` and a 3-way merge, so two people can edit at once.
- **Board state**: the global `W` (`W.board`, `W.sel`, `W.tool`, ...). Change board data only through `mutate(fn)` so undo, saving and view-only mode all work.
- **Rendering**: `renderItems()` / `paintItem()` per item type, `renderLinks()` for arrows, `renderCtx()` for the floating toolbar above a selection.
- **Item types**: `sticky`, `text`, `shape` (`SHAPES`), `image`, `table` (`rows`), `chart` (`kind`, `src` = a table id; `chartSVG()`), `draw` (`pts`, drawn with `strokePath()`).
- **Live collaboration**: the `Live` module sends short broadcast messages over the board's Supabase Realtime channel (selection, moving, typing, cursors, follow view, table cells). Nothing in `Live` is saved; the normal save stores the result.
- **Roles and locks**: `setReadOnly()` / `loadMyRole()` for viewers, `lockSel()` / `canUnlock()` for locked items.
- **Other modules**: `Profile` (name, bio, picture, nickname per board), `Feedback`, `Recon` (Google Classroom, read-only scopes, browser-only token).

## Database (Supabase project `qxejaxieoqqjljxrneau`)

Tables: `profiles`, `boards` (owner, title, `data` jsonb), `board_members` (role `owner`/`editor`/`viewer`, `nickname`), `feedback`, `api_tokens`, `subscriptions`, `admins`. RLS is on for all of them. Helper functions include `is_member`, `can_edit`, `is_admin`, `add_board_member`, `set_member_role`, `set_board_nickname`, `create_api_token`, and the `admin_*` functions.

### Database changes

Write new SQL in a new `*-setup.txt` file, make it safe to run twice (`if not exists`, `create or replace`, `drop policy if exists`), and have the owner run it in the Supabase SQL editor. Make the page keep working before the SQL runs (catch the "column/function does not exist" error and show a friendly message).

## Testing tips

- Open `index.html` through any static server (for example `npx serve .`). It talks to the real Supabase project, so use a test account.
- Check a shared board with two browser windows signed in as two different accounts: moving, typing, locking, follow view, and view-only mode.
- Press Ctrl+Shift+R after a push to bypass the browser cache.
