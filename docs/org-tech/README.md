# Organizational Technology

Third-party tech supporting general club operations — digital identity/access, member communications, and community engagement — as distinct from the apps FSRC built itself ([applications/](../applications/)) and the race-day tech in [race-services/](../race-services/). Like race-services, this covers hardware/software FSRC didn't build but configures and depends on.

## Google Workspace for Nonprofits

Provides `@steeplechasers.org` email addresses to select volunteers and serves as the club's file/document system (Google Drive). Governed by the [Digital Access and Identity Policy](https://docs.google.com/document/d/1dxsT7itCPQ_iHNLq_rNajqF9Ry72GxwGo9QmAflob4U/preview) (approved 2026-04-13):

- **Named accounts** (e.g. `jane.doe@steeplechasers.org`) — for individuals who regularly represent the club in an official capacity, need personal accountability for their actions (e.g. Board Members), or need to own/manage Drive content.
- **Role-based groups** (e.g. `president@`, `treasurer@`) — mandatory for President, VP, Secretary, Treasurer, and standing Committee Chairs; handed off to successors at transitions rather than tied to a person.
- **Collaborative inboxes** (e.g. `info@`, `volunteer@`) — Google Groups shared by multiple members, avoiding a shared password.

Drive access follows least privilege via groups: an Executive Board group (view/edit on board folders plus all of the FSRC Library), per-committee groups scoped to their own folders, an all-staff group (view-only, excluding restricted folders), and a restricted group for sensitive material (passwords, personnel issues). The same policy also governs FSRC Community forum group placement (board/committee categories, moderators, member-only and training-registrant-only access, synced from the membership/training databases) — see [membertility](../applications/membertility.md)'s `community` module, which implements this sync.

Admin responsibility is split by asset: the **Technology Chair** is primary admin for the FSRC server, Google Workspace, and domain/DNS; the **Communications Chair** is primary admin for MailChimp/Canva and Facebook/social (each is the other's backup). All admin accounts require 2FA. Onboarding/offboarding tracks the club's election cycle — the Secretary notifies the Technology Chair within 7 days of an election or appointment, an annual audit (each February) suspends orphaned accounts, and external-app access is revoked immediately when someone leaves a role.

## MailChimp

Used for member email communications (list synced from [membertility](../applications/membertility.md)'s membership module — see its `membership` CLI command group) and to offer a paid "premium promotion" service to local race directors, distinct from FSRC's contracted race-timing services ([contractility](../applications/contractility.md), [race-services/](../race-services/)).

## Facebook and the FSRC Community migration

FSRC currently uses Facebook Groups/Pages (e.g. "FSRC Group Runs") for general-community and member communication, alongside an email newsletter. An effort is underway to migrate some of this to [FSRC Community](../GLOSSARY.md) (see [membertility](../applications/membertility.md)'s `community` module), driven largely by Facebook's algorithmic feed making it unreliable for time-sensitive event info and inaccessible to members who don't use Facebook.

A June 2026 [member communications survey](https://docs.google.com/presentation/d/1d6UbXKCqowknwfKrcoBn2PuCpGTy01scSH2YXnBV2VY/preview) (196 responses, ~22% of memberships — self-selected/opt-in, so directional rather than representative) found broad support for the forum across every Facebook-usage segment: net forum willingness +55% vs. net Facebook approval -8%, even though 80% of respondents hadn't tried the forum yet. The ~17% of members who don't use Facebook had the highest forum willingness of any group and currently rely on an ad hoc 27-person opt-in email list as their only workaround for club information. Roughly a quarter of respondents said they'd missed a club event because of Facebook's feed, with another third unsure. Rollout has been intentionally gradual so far — introduced to the Coed 5K Training group and the non-Facebook pub-run distribution list, not yet promoted club-wide.

## References

- [Digital Access and Identity Policy](https://docs.google.com/document/d/1dxsT7itCPQ_iHNLq_rNajqF9Ry72GxwGo9QmAflob4U/preview) (Google Doc, approved 2026-04-13)
- [FSRC Community Forum Member Communications Survey](https://docs.google.com/presentation/d/1d6UbXKCqowknwfKrcoBn2PuCpGTy01scSH2YXnBV2VY/preview) (Google Slides, June 2026, N=196)
