'sq verify' output is for informational display only (shown in the
installation confirmation screen). Do not parse it. `sq` output is
not machine-readable.

Details: https://www.kicksecure.com/wiki/OpenPGP#sqop_vs_sq_for_automations

No --not-before/--not-after: 'sqop verify' above already enforces
signature freshness. Timestamps extracted from 'sqop' output only.

Verification decision logic:

- Only 'sqop verify' can set verification_success=true (above).
- Both 'sqop verify' and 'sq verify' can set verification_success=false.
- 'sq verify' failure therefore overrides a successful 'sqop verify'.
This is by design: both tools must agree for verification to pass,
providing defense in depth via two independent verification paths.

---

CURL_FORCE_SSL intentionally empty for onion mode:

When --onion is used, CURL_FORCE_SSL is never set. This is correct:
.onion transport is encrypted by Tor, and enforcing --proto =https
would break onion HTTP URLs. The variable is used unquoted on the
curl command line so it word-splits into two args (--tlsv1.3 and
--proto =https) for clearnet, or vanishes for onion. Do not "fix"
the missing quotes -- that is intentional.

---

version-parser output file permissions:

version-parser calls `output_path.touch(mode=0o644, exist_ok=True)`.
This is safe because the caller (version-validator) already creates
the output file via `mktemp`, which defaults to 0o600. The touch
with exist_ok=True on an existing file does not change its
permissions. Do not change 0o644 to 0o600 -- it has no effect.

---

Environment sanitization in the postinst path:

setpriv-tb-updater uses `setpriv --reset-env` which clears the
environment before running update-torbrowser as the tb-updater
account. This blocks env-based attacks (e.g. tb_skip_functions,
DEV_PASSTHROUGH, CURL_FORCE_SSL overrides) in the package-install
path. Manual user runs do NOT get this sanitization, but the user
is only affecting their own account.

---

eval of settings_echo:

`eval "$(/usr/libexec/helper-scripts/settings_echo)"` appears twice
(lines ~910, ~923 in update-torbrowser). This is acceptable because
settings_echo is a root-owned system binary in /usr/libexec.
Compromising it requires root, at which point the system is already
fully compromised. Do not refactor this to avoid eval -- the helper
script is designed to produce shell assignments.
