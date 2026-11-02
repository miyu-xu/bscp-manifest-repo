# AOSP17 release preparation
This branch provides a candidate aosp17.xml, not a completed default release.
It pins the published preparation code commit and all1087 official AOSP project commits.
The official superproject element is deliberately omitted from this composite manifest: the upstream superproject does not contain the BSCP root project; explicit per-project pins remain unchanged.
main/default.xml and the r14 rollback branch remain unchanged until binary assets, installation/configuration binding and final source integration are complete.
Do not treat a source sync as building the tested BSCP Host ports: the current frozen controller/provenance and Guest patchset are in the root repository; canonical portable Host build-source integration remains to be completed.
The versioned A17 binaries have been archived and hash-verified locally but are not yet uploaded to a GitHub release.
