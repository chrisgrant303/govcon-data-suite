# Treeport: third-party source (ExtPay.js)

`ExtPay.js` here is **ExtPay (ExtensionPay) 3.1.2 by Glen Chiacchieri and ExtPay contributors**, licensed by its author under AGPL-3.0 (npm) / LGPL-3.0 (repo LICENSE).

- Upstream: https://github.com/Glench/ExtPay (npm `extpay@3.1.2`, `dist/ExtPay.js`, unmodified sha256 `ec38f20eeea04502e68e99b939fe0e188045bf68cabe605080cdad79c7fae15a`).
- **Modified for Treeport** by CCE Partners LLC on 2026-10-20: in ExtPay()'s internal get()/set() helpers the storage area was changed from `chrome.storage.sync` to `chrome.storage.local` (two lines, each marked in the file). Nothing else was changed. A notice at the top of the file describes the change.
- This is the exact file shipped in the Treeport extension (`vendor/ExtPay.js`, sha256 `94113431cbb0dc4e7bceb695678b2d466aad3ff31720849e824c9c319b3c1316`), published here as its public source copy.
- The npm package declares AGPL-3.0-or-later; the upstream repository's LICENSE file says LGPL-3.0. Treeport follows the stricter AGPL-3.0-or-later terms.
- The file also bundles Mozilla's **webextension-polyfill**, unmodified, licensed under **MPL-2.0** (source: https://github.com/mozilla/webextension-polyfill; licence text: [licenses/MPL-2.0.txt](licenses/MPL-2.0.txt)).

## License texts (official copies from gnu.org and mozilla.org)
- [licenses/AGPL-3.0.txt](licenses/AGPL-3.0.txt): GNU Affero General Public License v3 (https://www.gnu.org/licenses/agpl-3.0.txt)
- [licenses/LGPL-3.0.txt](licenses/LGPL-3.0.txt): GNU Lesser General Public License v3 (https://www.gnu.org/licenses/lgpl-3.0.txt)
- [licenses/GPL-3.0.txt](licenses/GPL-3.0.txt): GNU General Public License v3 (https://www.gnu.org/licenses/gpl-3.0.txt). It's included because the LGPL-3.0 is written as additional permissions on top of the GPL-3.0 and incorporates its terms, so the LGPL text alone is incomplete.
- [licenses/MPL-2.0.txt](licenses/MPL-2.0.txt): Mozilla Public License 2.0 (official text from https://www.mozilla.org/media/MPL/2.0/index.txt), for webextension-polyfill (MPL-2.0), which ExtPay.js bundles unmodified

Treeport itself: https://chrisgrant303.github.io/govcon-data-suite/treeport/privacy.html
