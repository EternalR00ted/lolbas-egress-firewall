# Deploying via Intune — Endpoint Security → Firewall

This is the **recommended, native** way to ship these rules: as a managed **Windows Firewall Rules** profile under Endpoint Security. The rules show up as a policy object you can audit, report on, and version — not buried in each device's local store.

> A PowerShell/local-store alternative exists for labs and quick tests — see the bottom of this page — but the Endpoint Security profile is what you want for a managed fleet.

> 📸 *Screenshots below are placeholders — drop your own tenant captures into [`../assets/`](../assets/) using the filenames shown. A full shot list is in [`../assets/README.md`](../assets/README.md).*

---

## Before you click: two profiles, one operational gotcha

Under **Endpoint security → Firewall** there are two different Windows profiles. Don't confuse them:

| Profile | What it's for |
|---------|---------------|
| **Windows Firewall** | Global settings — turn the firewall on, default inbound/outbound action, logging, merge behavior. |
| **Windows Firewall Rules** | Granular per-application / per-port rules. **This is the one we use.** |

**The gotcha that changes how you structure this** (straight from Microsoft's docs):

> - Each **Windows Firewall Rules** profile supports up to **150 rules**.
> - **If a single rule in a profile fails to apply, every rule in that profile fails and none are applied to the device.**
> - When that happens, Intune reports the whole profile as failed and **cannot tell you which individual rule broke.**

Two consequences this guide is built around:

1. **Path accuracy is non-negotiable.** One typo'd or non-existent file path can silently take down the entire block set on a device. Validate paths before you publish (the `-WhatIf`/`Test-Path` logic in the script, or just confirm the file exists on a reference image).
2. **Split tiers into separate profiles.** Keep Tier 1 and Tier 2 in their own profiles so a bad path in Tier 2 can't nuke your Tier 1 blocks. It also lets you assign them to different device populations cleanly.

---

## Step-by-step: create the Tier 1 profile

### Step 1 — Create the policy

**Microsoft Intune admin center → Endpoint security → Firewall → Create Policy.**
Set **Platform** to `Windows 10, Windows 11, and Windows Server` and **Profile** to `Windows Firewall Rules`, then select **Create**.

> **📸 Screenshot 2 — Pick the right profile.** Capture the **Profile** dropdown so readers can see **Windows Firewall Rules** (the per-app rules profile) versus the global **Windows Firewall** profile.

![Create policy — Windows Firewall vs Windows Firewall Rules profile dropdown](../assets/02-create-policy-profile-dropdown.png)

> **📸 Screenshot 3 — Platform + profile selected.** The *Create a profile* blade with **Platform: Windows 10, Windows 11, and Windows Server** and **Profile: Windows Firewall Rules**.

![Create a profile — platform and profile selection](../assets/03-create-profile-platform.png)

### Step 2 — Name it (Basics)

On **Basics**, name the policy `LOLBAS Egress Block – Tier 1` and add a short description (e.g. *outbound-block rules for network-capable LOLBins, Tier 1*). Select **Next**.

> **📸 Screenshot 4 — Basics tab.** The named policy on the Basics tab.

![Basics tab — policy name and description](../assets/04-basics-name.png)

### Step 3 — Add the block rules

On **Configuration settings**, select **Add** to open the **Create Rule** pane. For **each** path in the Tier 1 list below, create one rule with these settings:

| Field | Value |
|-------|-------|
| **Name** | e.g. `Block mshta (System32)` |
| **Direction** | **Out** *(if left "Not configured" it defaults to Outbound — but set it explicitly)* |
| **Action** | **Block** *(⚠ "Not configured" defaults to **Allow** — you must explicitly choose Block)* |
| **File path** | the absolute path, e.g. `C:\Windows\System32\mshta.exe` |
| Network type / Protocol / Local & remote ports | leave **Not configured** → blocks all egress for that binary |

**Save** the rule; it appears in the list. Repeat for every path.

> **📸 Screenshot 5 — A completed rule.** The **Create Rule** pane filled in for `mshta` — Direction **Out**, Action **Block**, File path `C:\Windows\System32\mshta.exe`. This is the key instructional shot; make it clear and readable.

![Create Rule pane — outbound block for mshta by file path](../assets/05-create-rule-pane.png)

> **📸 Screenshot 6 — Rules populated.** The Configuration settings page after several rules are added, showing the System32 + SysWOW64 pairs building up.

![Configuration settings — populated rule list](../assets/06-rule-list-populated.png)

#### Tier 1 rule list (one rule per path)

Every 64-bit binary needs **both** the System32 and SysWOW64 path — a System32-only rule leaves the 32-bit copy usable.

| Binary | File path |
|--------|-----------|
| mshta | `C:\Windows\System32\mshta.exe` |
| mshta | `C:\Windows\SysWOW64\mshta.exe` |
| regsvr32 | `C:\Windows\System32\regsvr32.exe` |
| regsvr32 | `C:\Windows\SysWOW64\regsvr32.exe` |
| certutil | `C:\Windows\System32\certutil.exe` |
| certutil | `C:\Windows\SysWOW64\certutil.exe` |
| certoc | `C:\Windows\System32\certoc.exe` |
| certoc | `C:\Windows\SysWOW64\certoc.exe` |
| certreq | `C:\Windows\System32\certreq.exe` |
| certreq | `C:\Windows\SysWOW64\certreq.exe` |
| bitsadmin | `C:\Windows\System32\bitsadmin.exe` |
| bitsadmin | `C:\Windows\SysWOW64\bitsadmin.exe` |
| cmstp | `C:\Windows\System32\cmstp.exe` |
| cmstp | `C:\Windows\SysWOW64\cmstp.exe` |
| hh | `C:\Windows\System32\hh.exe` |
| hh | `C:\Windows\SysWOW64\hh.exe` |
| presentationhost | `C:\Windows\System32\PresentationHost.exe` |
| presentationhost | `C:\Windows\SysWOW64\PresentationHost.exe` |
| desktopimgdownldr | `C:\Windows\System32\desktopimgdownldr.exe` |
| makecab | `C:\Windows\System32\makecab.exe` |
| makecab | `C:\Windows\SysWOW64\makecab.exe` |
| diantz | `C:\Windows\System32\diantz.exe` |
| diantz | `C:\Windows\SysWOW64\diantz.exe` |
| expand | `C:\Windows\System32\expand.exe` |
| expand | `C:\Windows\SysWOW64\expand.exe` |
| extrac32 | `C:\Windows\System32\extrac32.exe` |
| extrac32 | `C:\Windows\SysWOW64\extrac32.exe` |
| printbrm | `C:\Windows\System32\spool\tools\PrintBrm.exe` |
| imewdbld | `C:\Windows\System32\IME\SHARED\IMEWDBLD.exe` |
| ieexec | `C:\Windows\Microsoft.NET\Framework\v4.0.30319\ieexec.exe` |
| ieexec | `C:\Windows\Microsoft.NET\Framework64\v4.0.30319\ieexec.exe` |

*`ieexec` also ships under `...\Framework\v2.0.50727\` on some builds — add those paths only if the file exists there (a path to a non-existent file can fail the whole profile).*

### Step 4 — Assign (with exclusions)

On **Assignments**, add your **standard endpoints** device group, then **exclude** the groups that have legitimate use: PKI/CA/cert-enrollment (`certutil`/`certoc`/`certreq`), and — for Tier 2 later — developer/build machines. Select **Next**.

> **📸 Screenshot 7 — Assignments with exclusions.** Show the included standard-endpoints group and the excluded PKI/dev/admin groups side by side — the exclusion pattern is the part people get wrong.

![Assignments — include standard endpoints, exclude PKI/dev/admin groups](../assets/07-assignments-include-exclude.png)

### Step 5 — Review + create

Review the summary and select **Create**. The profile deploys on the next device sync.

> **📸 Screenshot 8 — Review + create.** The summary page before creation.

![Review and create summary](../assets/08-review-create.png)

After devices sync, confirm it applied cleanly (remember: all-or-nothing — a green result means every rule landed):

> **📸 Screenshot 9 — Apply status.** Endpoint security → Firewall → your policy → device / report showing **Succeeded**.

![Firewall policy device report — Succeeded](../assets/09-device-apply-report.png)

---

## Tier 2 profile (audit first)

Create a **second** `Windows Firewall Rules` profile named `LOLBAS Egress Block – Tier 2`, same steps. Assign it to **non-developer** standard endpoints, and only after you've confirmed the population doesn't rely on VBScript/JScript automation. **Exclude** developer/build device groups (`msbuild`).

| Binary | File path |
|--------|-----------|
| cscript | `C:\Windows\System32\cscript.exe` |
| cscript | `C:\Windows\SysWOW64\cscript.exe` |
| wscript | `C:\Windows\System32\wscript.exe` |
| wscript | `C:\Windows\SysWOW64\wscript.exe` |
| esentutl | `C:\Windows\System32\esentutl.exe` |
| esentutl | `C:\Windows\SysWOW64\esentutl.exe` |
| findstr | `C:\Windows\System32\findstr.exe` |
| findstr | `C:\Windows\SysWOW64\findstr.exe` |
| msbuild | `C:\Windows\Microsoft.NET\Framework\v4.0.30319\MSBuild.exe` |
| msbuild | `C:\Windows\Microsoft.NET\Framework64\v4.0.30319\MSBuild.exe` |

*`msbuild` also exists under `v3.5\` and `v2.0.50727\` — add only the versions present on your image.*

---

## Gotchas checklist

- ✅ **Action = Block, explicitly.** "Not configured" silently means *Allow*.
- ✅ **Both System32 and SysWOW64** for every 64-bit binary.
- ✅ **Literal absolute paths** in the File path field — `C:\Windows\...`. Don't rely on `%SystemRoot%`/env-var expansion.
- ✅ **Validate every path exists** on a reference image first — one bad path fails the entire profile with no indication which rule broke.
- ✅ **Separate profile per tier** — contains blast radius of a failed rule and simplifies assignment.
- ✅ **Stay under 150 rules per profile** (Tier 1 is ~30, so you're fine in one).
- ✅ **Exclusions on assignment**, not deletion — carve PKI/dev/IT-admin device groups out of the assignment rather than dropping the rules.

## Verify

On a device:

```powershell
Get-NetFirewallRule -PolicyStore ActiveStore |
    Where-Object DisplayName -like '*mshta*' |
    Format-Table DisplayName, Direction, Action, Enabled
```

Then watch [`../detections/lolbas-egress-hunt.kql`](../detections/lolbas-egress-hunt.kql) in audit mode for a week before broadening. See [`../docs/rollout.md`](../docs/rollout.md).

> **📸 Screenshot 10 — The payoff.** Advanced Hunting results from the egress query — a LOLBin reaching public IP space is the signal that makes this worth doing. (Great shot for a post.)

![Advanced Hunting — LOLBin outbound egress results](../assets/10-hunting-query-results.png)

---

## Alternative: bulk deploy via PowerShell (labs / local store)

For a lab, a standalone box, or when you want every rule created in seconds rather than clicked, [`Deploy-LOLBASFirewall.ps1`](Deploy-LOLBASFirewall.ps1) creates the whole tiered set locally (handles System32/SysWOW64, skips paths not on the build):

```powershell
.\Deploy-LOLBASFirewall.ps1 -Tier 1            # or -Tier All
.\Deploy-LOLBASFirewall.ps1 -Tier All -Remove  # clean rollback by group
```

You *can* push this through Intune as a Platform script or Remediation, but note the trade-off: rules created this way live in the device's **local store**, so they won't appear as a managed firewall policy object in the Endpoint Security blade. For a managed fleet, prefer the profile method above.
