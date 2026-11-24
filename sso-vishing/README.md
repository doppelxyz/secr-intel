# SSO Vishing

An open, curated feed of internet infrastructure used in passkey and single-sign-on (SSO) enrollment voice-phishing (vishing) campaigns. Maintained by the Doppel SECR (Social Engineering Counter-intelligence and Research) team.

TLP:CLEAR. Free to use, including inside the security operations of commercial organizations. Attribution required. Not for resale or repackaging as a commercial product. See [LICENSE](../LICENSE).

## What this is

Attackers in this threat set call employees while posing as IT or help-desk staff and direct them to convincing fake pages to "re-enroll" a passkey, MFA method, or SSO login. The page captures the login or enrolls the attacker's own passkey or MFA device, which gives the attacker lasting access. The sites are built on generic, login-themed domains (for example `createssopasskey[.]com`) with the targeted organization's name usually placed in the subdomain.

Because the brand lives in the subdomain or in specific domain naming patterns and the sites often use wildcard DNS, conventional brand monitoring misses most of this infrastructure.

The feed covers the setup and launch stages of the attack, before the call is placed: the enrollment-themed root domains, and the subdomains staged under them for each target (published as patterns, never by the target's name).

These domains are registered fresh, used within days, and often suspended soon after, so a list of domains goes stale quickly. A naming pattern outlives every domain built from it, which is why the feed publishes domain patterns alongside the domains themselves.

## Why it exists

- **Block at the root.** One wildcard root can serve a lure for any company, such as `examplecorp.createssopasskey[.]com`. Blocking the registrable root stops all of them at once. These roots are tagged so you can prioritize them.
- **Share what works.** Much of this infrastructure is only visible to whoever first saw the call. Pooling sightings across the community makes everyone faster.

## What's in the feed

- Registrable domains, subdomains, and their naming patterns observed in these campaigns. Where a domain carries an organization's name, the feed lists its shape instead (for example `<company>.linkpasskey.com`), never the name.
- A confidence tier for every entry (T1 confirmed, T2 corroborated).
- First-seen date, newest entries first.
- A wildcard tag for roots that accept any subdomain, and a priority-block tag for roots worth blocking first.
- Source credits in the `source_names` column: who reported each entry (for example Sekoia, Palo Alto Networks Unit 42, or Doppel), without identifying any affected organization.

No victim, targeted, customer, or personal data appears in this feed. Entries that cannot be redacted and published safely are withheld.

## The file

| File | Format | Use it for |
|---|---|---|
| [doppel-sso-vishing.csv](doppel-sso-vishing.csv) | CSV, one row per domain or domain pattern | Spreadsheets, SIEM import, enrichment, blocklists (domain rows only) |

Raw URL for your tools: `https://raw.githubusercontent.com/doppelxyz/secr-intel/main/sso-vishing/doppel-sso-vishing.csv`

Monitoring and detection run multiple times a day. New findings are published as they are identified and verified. The file name never changes between releases, so a saved link keeps working. Older versions are in the repository history. [CHANGELOG.md](CHANGELOG.md) summarizes what changed in each release.

### Columns

| Column | Meaning |
|---|---|
| `indicator` | The domain or pattern, defanged (`[.]` for dots) so it is safe to paste. |
| `value` | The same entry, not defanged, for tools and blocklists (domain rows only). |
| `type` | `domain` (a registrable domain or subdomain) or `domain pattern` (a domain shape, see below). |
| `confidence_tier` | `T1` confirmed or `T2` corroborated (see Confidence tiers below). |
| `status_registration` | Registration status of the root domain at our last check (for example `registered, active` or `suspended by registrar (on hold)`). |
| `registered_on` | Registry creation date of the root domain. |
| `registrar` | Sponsoring registrar of the root domain. |
| `root_wildcard_dns` | `yes` if any random name under the root resolves, so hostname lists can never be complete. |
| `first_seen` | Earliest date any source reported the entry. For a pattern row: the most recent first-seen date among its domains. |
| `date_added` | Date the entry entered this feed's source list. For a pattern row: the most recent date among its domains. |
| `source_names` | Who reported the entry (source credit). |
| `attribution` | The actor or cluster associated with the entry, relayed as reported, with the reporting source in parentheses where known, or `Unattributed (pattern or context match)`. |
| `domain_template` | The entry's generic shape (tags explained below). |
| `example` | The shape filled in with the made-up company `examplecorp`. |
| `asn`, `asn_org` | Network the domain resolved to, where known. |
| `ip_addresses` | IP addresses the domain resolved to, where they are safe to publish. |
| `priority_block` | `yes` for roots worth blocking first. |
| `lifecycle_status` | Whether the infrastructure is currently `active`, `inactive`, `suspended`, and so on. |
| `pattern_domains` | For a `domain pattern` row: how many confirmed domains in this feed it stands for. `0` means the pattern was published by the source credited in `source_names`, and no domain in this feed matches it yet. |

Source credit and attribution are kept separate on purpose. `source_names` says who reported the infrastructure. `attribution` says which actor or cluster a reporter associated with it.

### Confidence tiers

- **T1 (confirmed):** confirmed by a Doppel analyst, published by a security vendor or ISAC, reported by an affected organization, or seen in two or more independent source types.
- **T2 (corroborated):** one source type plus a high-confidence naming match, a registrar suspension, or (for researcher posts) a registration date inside the campaign window.

### Domain patterns

A `domain pattern` row describes a shape instead of one domain. Tags mark the parts that vary:

- `<company>`: where a targeted organization's name sits.
- `<tld>`: a top-level domain code used as part of the name, for example `cn` in `examplecorp.cn.getpasskey.org`.
- `<N digits>` or `<N-M digits>`: a run of exactly N digits, or of N to M digits.

## How to use it

- **Block wildcard roots first.** Rows with `priority_block` or `root_wildcard_dns` set to `yes` are roots where one block covers every target-named lure beneath them. Load the `value` column of `domain` rows into DNS filtering, web gateways, or firewalls.
- **Search your logs.** Run the domains and patterns against the last 90 days of DNS and proxy logs, then against new hostnames as they appear. A historical match is a reason to look at who resolved it and when. A match on a hostname first seen in the last 30 days is a candidate for a root-level block.
- **Turn a pattern into a match rule** by replacing each tag: `<company>` with any name, `<N digits>` or `<N-M digits>` with that many digits, and `<tld>` with any top-level domain. For example, `<company>.linkpasskey.com` matches every subdomain of `linkpasskey.com`.
- **Match whole hostname labels, never substrings.** `sso` sits inside `picasso`, and `entra` inside `central`. Split hostnames on dots and hyphens and compare the labels, or anchor the pattern. Expect some noise from legitimate identity vendors, and tune by registrar and hosting rather than by widening the match.
- **Treat a match as a lead.** A match shows contact with infrastructure of this shape. It is a candidate for review and a block decision, and does not on its own establish compromise.
- **Identity and help-desk teams:** review passkey and MFA enrollments that follow a visit to a listed domain.

## How to contribute a sighting

Email new sightings to SECR@DOPPEL.COM, with the domain(s), subdomain(s), or IP(s) and when you saw them. Submissions stay private, and entries are added or updated only after manual analyst review. Any company, brand, or domain name that identifies your organization is redacted before publication.

## How to report a correction or problem

Think a listed entry is legitimate, wrong, or stale, or found a format problem? Email SECR@DOPPEL.COM with the exact entry and why. Entries are corrected or removed only after manual analyst review, and confirmed changes are noted in [CHANGELOG.md](CHANGELOG.md).

## License and responsible use

Use of this feed is governed by [LICENSE](../LICENSE). The feed is provided "as is," without warranty of any kind.

- Entries are provided for defensive blocking and detection.
- Treat every entry as intelligence, not proof of wrongdoing by any registrant; registrations change hands, and some entries may later be disputed. Verify before taking action with legal consequences.
