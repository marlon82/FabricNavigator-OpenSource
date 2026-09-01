# Public release checklist

Use this checklist for the public source snapshot and each subsequent release.

## Project licensing

- [x] Add GNU GPL v3.0 as the top-level license for FabricNavigator's original code.
- [x] Add the license identifier to the README and repository metadata.
- [x] Confirm that the repository owner is authorized to license the original contributions under that license.

FabricNavigator's license does not replace third-party licenses. ACLI remains
under GNU GPL v3.0, SNMP4J remains under Apache License 2.0, and every other
bundled dependency remains under its own license.

## Third-party compliance

- [x] Credit ACLI and its author, Ludovico Stevens.
- [x] Include the complete ACLI GNU GPL v3.0 license.
- [x] Make the corresponding ACLI source and FabricNavigator integration changes available at each release tag.
- [x] Include the complete SNMP4J Apache License 2.0 text.
- [x] Keep detailed acknowledgements in `THIRD_PARTY_NOTICES.md` and in the application Credits page.
- [x] Keep optional Extreme Networks product photographs outside the core distribution.
- [ ] Confirm redistribution permission before publishing any optional product-image package.

## Repository and release hygiene

- [x] Confirm that the current tracked tree contains no GitHub token, FabricNavigator secret path, or personal email match from the initial targeted scan.
- [x] Publish from a new single-commit snapshot so private development history is not transferred.
- [ ] Review GitHub secret scanning and dependency/security alerts regularly.
- [ ] Enable GitHub private vulnerability reporting.
- [ ] Verify every release archive against its SHA-256 manifest.
- [x] Confirm that source and binary packages contain the license texts for their bundled components.
- [x] Document the security-reporting process and supported release policy in `SECURITY.md`.
- [x] Remove the existing screenshots containing lab details and obsolete product views from the public snapshot.

## Resolved publication blockers

- [x] Remove all 76 tracked Extreme Networks product photographs from the clean
  public snapshot. The private repository retains its original history; the
  public repository is created from a new, history-free snapshot.
- [x] Remove the screenshots containing a lab IP/MAC address, `admin:admin`,
  and an obsolete EDM view from the clean public snapshot.

## Final publication

- [x] Publish the sanitized source snapshot without the private development history.
- [x] Keep optional product-image packages unpublished until redistribution permission is confirmed.
- [x] Verify the public README, Credits page, source tree, and GPL-3.0 license detection after publication.
