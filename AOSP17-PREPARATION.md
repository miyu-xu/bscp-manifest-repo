# Published AOSP 17 default

`main/default.xml` includes `aosp17.xml`: 1087 official projects retain their exact fixed SHA revisions, plus the published BSCP root commit d8af389d8ed635c3ef5d94dbc10aa4c982aae8b2. The custom Host source/build recipes and Guest adaptations are in the root release tree; syncing official projects alone does not build these ports.

Release: https://github.com/miyu-xu/bscp-root/releases/tag/bscp-aosp17-20261102

This is the default AOSP 17 source baseline plus limited-deployment reference runtime archives, not a fresh-machine installer. Read the exact hardware, account, protected-path, driver and dynamic-library requirements before use: https://github.com/miyu-xu/bscp-root/blob/d8af389d8ed635c3ef5d94dbc10aa4c982aae8b2/doc/aosp17/SUPPORTED-DEPLOYMENT-CONDITIONS-v1830.md

Windows independent CLI reconnect remains disabled. Mac ext4 historical 70 pass / 1 fail is retained. Fault tasks remain UNKNOWN even when resources are REAPED. Real UI/input and A/H3 are outside the acceptance scope.

Fresh checkout: `repo init -u https://github.com/miyu-xu/bscp-manifest-repo.git -b main` then `repo sync`. Do not force-sync/reset existing user changes. Rollback uses the preserved `bscp-android-15.0.0_r14` branch, manifest commit `89c0365a4dff5c0e52a5386877592a2f6fd45548`, root commit `43e4786a48c142b4c8b6f4bfb0db38cd32b20e42`. `r14-default.xml` preserves the previous default manifest without deleting its history. Never overwrite live/failed instance state or replay UNKNOWN requests.

## Current documentation and private configuration

The current AOSP17 deployment guide and verification scripts require an explicit private binding directory. Personal account names, home directories and LAN addresses are not configuration examples. Redacted placeholders do not make the reference binaries relocatable. Use the new public-docs-v1838 attachment for current instructions. Legacy attachments and Git history can still contain original build paths; they have not been silently deleted or rewritten.
