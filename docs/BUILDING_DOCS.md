<!--
Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

     https://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.

Copyright 2026-present Gabby Contributors.
-->

# Build and preview the documentation

The documentation site is built from the Markdown files in `docs/` with MkDocs and Material for
MkDocs. Its colors and typography follow the [Undreamt website](https://undreamt.in/). Read the Docs
uses the repository-level `.readthedocs.yaml` configuration and installs only the documentation
dependencies from `docs/requirements.txt`; it does not install Gabby or its runtime dependencies to
build the site.

## Preview locally

From the repository root, create a separate virtual environment for the documentation tools and
start MkDocs:

```sh
python -m venv .venv-docs
source .venv-docs/bin/activate
python -m pip install -r docs/requirements.txt
mkdocs serve
```

Open the local URL printed by MkDocs, usually `http://127.0.0.1:8000/`. The development server
reloads pages as Markdown or site configuration changes.

On Windows PowerShell, activate the environment with
`.\.venv-docs\Scripts\Activate.ps1` instead of `source .venv-docs/bin/activate`.

Build the static site for a production-style check with:

```sh
mkdocs build
```

The generated site is written to `site/`, which should remain a build artifact and not be committed.

## Host on Read the Docs

Create or import the Gabby project in Read the Docs and connect it to the GitHub repository
`undreamt-oss/gabby`. The checked-in `.readthedocs.yaml` tells Read the Docs to build `mkdocs.yml`
using the isolated documentation requirements. Enable builds for the branch or tag you want to
publish; the default version is generally the `latest` branch build.

The site uses Read the Docs' canonical URL when that environment value is available. Local builds
and other hosts fall back to `https://docs.undreamt.in/`.

To serve Gabby at this subdomain, add `docs.undreamt.in` under the project's **Admin → Domains**
settings in Read the Docs and mark it canonical. Then add the CNAME record Read the Docs provides
to the `undreamt.in` DNS zone. For Read the Docs Community, the target is `readthedocs.io`; in
Cloudflare, set the record to **DNS only** (proxy disabled). Use the target shown in the Read the
Docs dashboard if it differs. This configures the `docs` subdomain while leaving the apex domain
and `www` records in place. The zone export in `undreamt.in.txt` is archival guidance, not a file
to upload directly to production DNS.

## Host on GitHub Pages

The repository includes a GitHub Actions workflow that builds `mkdocs.yml` and deploys the
generated `site/` directory. In the repository's **Settings → Pages**, set **Build and deployment
→ Source** to **GitHub Actions**. The workflow runs for documentation and MkDocs configuration
changes on `main`, or can be started manually from the Actions tab.

To use `docs.undreamt.in` with GitHub Pages, set it as the custom domain in **Settings → Pages**
and create a DNS `CNAME` record with name `docs` pointing to `undreamt-oss.github.io`. If Cloudflare
hosts the DNS zone, set the record to **DNS only**. The `docs` subdomain can point to only one host;
use either GitHub Pages' target here or the Read the Docs target from its dashboard. Wait for
GitHub's DNS check and HTTPS certificate before enforcing HTTPS.

For each published Read the Docs version, review the build log and open the resulting site to check
navigation, search, code blocks, and internal links. Keep the generated `site/` directory out of Git.
