# Asset Vault: Who Can See Each Page

## Access table

| # | Page | Admin | Editor | Viewer | Auditor |
|---|---|---|---|---|---|
| 1 | Sign in | Yes | Yes | Yes | Yes |
| 2 | Documents | Yes | Yes | Yes | Yes |
| 3 | Document detail | Yes | Yes | Yes | Yes |
| 4 | Upload new version | Yes | Yes | No | No |
| 5 | Create folder or document | Yes | Yes | No | No |
| 6 | Users and roles | Yes | No | No | No |
| 7 | Audit log | Yes | No | No | Yes |
| 8 | Messages | Shown when triggered | Shown when triggered | Shown when triggered | Shown when triggered |

## Notes

**Page 1 (Sign in)** is the only page visible before signing in. Once signed in, users land on Documents.

**Pages 2 and 3** are visible to every signed-in role. What changes is the set of buttons:

- Download: Admin, Editor, Viewer. Not Auditor.
- Upload new version, Rename, Move, Restore: Admin and Editor only.
- Delete: Admin, and Editors for documents they created.
- Version history: all roles.

**Pages 4 and 5** are hidden from Viewers and Auditors. They are reached through buttons on pages 2 and 3.

**Page 6** is visible only to Admins. It appears in the side menu only for Admins.

**Page 7** is visible to Admins and Auditors. It appears in the side menu for both.

**Page 8** is not a page you navigate to. It is a message that appears in place when:

- A role tries an action it cannot use.
- An upload is rejected (for example, the file is too large or the wrong type).
- A document was changed by someone else while it was being edited.
- An action succeeds.

## Side menu by role

| Role | Menu items |
|---|---|
| Admin | Documents, Audit log, Users |
| Editor | Documents |
| Viewer | Documents |
| Auditor | Documents, Audit log |
