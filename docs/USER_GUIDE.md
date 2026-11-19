# DocFlow — User Guide

## 1. About DocFlow

DocFlow helps an organization manage documents, versions, reviews, approvals, and audit activity in one tenant-scoped workspace.

The screens and actions shown here describe the intended product behavior. Exact labels may change as the application is implemented.

## 2. Sign in

1. Open the DocFlow application URL provided by your organization.
2. Enter your registered email and password, or use the configured sign-in method.
3. Complete any additional verification required by your organization.
4. You will see the navigation and actions permitted by your assigned role.

If you were invited, use the invitation link before signing in for the first time. Invitation links are time-limited and intended for one-time use.

## 3. Roles at a glance

| Role | What you can generally do |
|---|---|
| Tenant Admin | Manage tenant settings, users, invitations, roles, and permissions |
| Document Manager | Manage folders and document lifecycle within assigned scope |
| Editor | Create documents, upload versions, and submit documents for review |
| Reviewer | Review submitted versions and approve or reject them |
| Viewer | Browse, preview, and download permitted documents |
| Auditor | Review permitted audit events and reports |

Your organization may grant additional permissions or restrict access to particular resources. Your role does not automatically provide access to every document.

## 4. Common navigation

Depending on your role, you may see:

- Overview or Dashboard
- Documents and Folders
- My Documents
- Uploads and Versions
- Pending Approvals
- Users and Invitations
- Roles and Permissions
- Audit Logs and Reports
- Notifications
- Settings
- Profile

If an item is not visible, you may not have the required permission.

## 5. Tenant Admin

Typical navigation:

- Overview
- Documents
- Folders
- Users
- Invitations
- Roles & Permissions
- Audit Logs
- Notifications
- Settings
- Profile

Typical tasks:

### Invite a user

1. Open **Users** or **Invitations**.
2. Choose **Invite User**.
3. Enter the user's details and select the appropriate role.
4. Send the invitation.
5. Track its status from the invitation list.

An invitation may expire. If it expires before acceptance, issue a new invitation.

### Manage users

Open the user record to review their status and assigned role. Deactivate access when a user should no longer use the tenant. Deactivation should prevent new sign-ins and refreshes.

### Manage roles and permissions

Review the permissions assigned to each role before making changes. Grant only the access needed for the person's responsibilities.

## 6. Document Manager

Typical navigation:

- Overview
- All Documents
- Folders
- Uploads
- Versions
- Approvals (if granted)
- Archive
- Notifications
- Profile

Typical tasks:

- Create and organize folders.
- Review document metadata.
- Upload document versions.
- Archive or restore documents within your permissions.
- Track document lifecycle status.

A Document Manager does not automatically receive approval rights. Approval depends on explicit permission.

## 7. Editor

Typical navigation:

- Overview
- My Documents
- All Documents
- Create Document
- Drafts
- Submitted
- Rejected
- Notifications
- Profile

### Create a document

1. Open **Create Document**.
2. Enter the required title and metadata.
3. Select a permitted folder.
4. Save the document as a draft.

### Upload a version

1. Open the document.
2. Choose **Upload Version**.
3. Select the file.
4. Confirm the version details.
5. Wait for the upload to complete.

Uploading a new version should preserve earlier versions rather than overwrite their file contents.

### Submit for approval

1. Open the draft.
2. Review the metadata and uploaded version.
3. Choose **Submit for Approval**.
4. Confirm the submission.

After submission, the document is no longer an editable draft until the workflow permits further changes.

### Respond to rejection

1. Open the rejected document.
2. Read the reviewer's reason or comments.
3. Make the required changes.
4. Upload a new version if needed.
5. Submit it again.

## 8. Reviewer

Typical navigation:

- Overview
- Pending Approvals
- Reviewed by Me
- Approved
- Rejected
- Notifications
- Profile

### Review a submission

1. Open **Pending Approvals**.
2. Select a submitted document.
3. Review its metadata and the specific submitted version.
4. Choose **Approve** or **Reject**.
5. If rejecting, provide a reason.
6. Confirm the decision.

A reviewer cannot approve their own submission. Approval applies to the version that was reviewed.

## 9. Viewer

Typical navigation:

- Overview
- Browse Documents
- Recent
- Shared with Me
- Downloads
- Notifications
- Profile

Viewers can browse and open documents within their permitted scope. Download availability depends on the document's permissions and policy.

## 10. Auditor

Typical navigation:

- Overview
- Audit Logs
- User Activity
- Document Activity
- Reports
- Notifications
- Profile

Auditors can inspect permitted audit records. Audit access does not automatically grant permission to open or download the underlying document.

## 11. Document statuses

| Status | Meaning |
|---|---|
| Draft | Being prepared; not submitted |
| Submitted | Waiting for review |
| Approved | The reviewed version was approved |
| Rejected | Reviewer returned it with a reason |
| Archived | Removed from active working lists but retained according to policy |

A rejected document can return to Draft for revision. An archived document may be restored if the user has permission.

## 12. Notifications

Notifications may inform you about:

- Invitations and account events.
- A document submitted for review.
- Approval or rejection decisions.
- Comments or workflow changes.
- Administrative updates.

Open the notification to navigate to the related item when you have access.

## 13. Safe usage

- Do not share your password or invitation link.
- Upload only files you are authorized to share.
- Check the document and version before submitting.
- Review rejection comments before resubmitting.
- Do not assume that hiding an action in the UI means an operation is allowed through another route.
- Sign out when using a shared device.

## 14. Troubleshooting

| Problem | What to do |
|---|---|
| Invitation link expired | Ask a Tenant Admin to send a new invitation |
| Cannot see a menu item | Ask an administrator to review your role and permissions |
| Cannot open a document | Confirm that the document is in your permitted scope |
| Upload fails | Check file type/size and retry; contact support if it continues |
| Cannot submit | Confirm required metadata and a valid uploaded version are present |
| Cannot approve | Confirm you are assigned the Reviewer permission and did not submit the item yourself |
| Sign-in or refresh fails | Contact your administrator; your account may be inactive or your session revoked |

## 15. Support

For access problems, contact your organization's DocFlow Tenant Admin. For application errors, provide the time of the issue, the action you attempted, and any displayed correlation/reference ID. Do not send passwords or tokens to support.
