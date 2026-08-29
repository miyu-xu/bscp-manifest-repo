# BSCP Workspace License Policy

[简体中文](LICENSE_POLICY.zh-CN.md) | English

Original manifest configuration and documentation tracked by this repository are licensed under
`PolyForm-Noncommercial-1.0.0` unless a file states otherwise. See [LICENSE](LICENSE) for the
controlling terms. Commercial use requires prior written authorization as described in
[Commercial Licensing](COMMERCIAL_LICENSING.md).

The commercial-use restriction makes the BSCP-original material source-available, not
OSI-approved open-source software.

## Independent project licenses

This manifest selects multiple independent Git repositories. Each project retains its own
copyright, license, NOTICE files, file-level SPDX headers, and third-party obligations. In
particular:

- AOSP-derived projects remain under their applicable Apache-2.0, GPL, NOTICE, and file-level
  terms;
- crosvm remains under its BSD and bundled third-party licenses;
- gfxstream and aemu remain under their Apache-2.0 and bundled third-party licenses;
- the optional HD repository remains governed by its repository license and notices;
- firmware, generated Android images, APEX files, host tools, and dependencies retain their source
  project licenses.

The BSCP license does not relicense those projects. Combining independently licensed components in
one workspace, build, image, package, or commercial product requires compliance with every
applicable license.

## Commercial scope

A commercial BSCP license covers only rights the applicable BSCP copyright holder can grant. It
does not grant rights to third-party code, patents, trademarks, firmware, codecs, build tools, or
generated artifacts. Repository access or silence is not commercial permission; authorization
must be explicit and written and must identify the licensee, versions, scope, and effective date.

## Contributions and release governance

Copyright does not transfer when a contribution is submitted. Do not merge external contributions
into commercially licensable BSCP-original material unless the project has documented rights to
offer those contributions under both the repository license and a separate commercial license.

Before publishing a manifest release:

- pin every project revision and capture its license/NOTICE inventory;
- include the root and manifest licenses plus all component notices in the release package;
- generate an SBOM for source and binary artifacts;
- preserve historical permissions and do not replace upstream headers;
- obtain qualified legal review when ownership or compatibility is uncertain.
