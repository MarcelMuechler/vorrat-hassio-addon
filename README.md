# vorrat-hassio-addon

Home Assistant Add-on Repository für [Vorrat](https://github.com/MarcelMuechler/vorrat) —
a self-hosted grocery & stock management app.

## Installation

In Home Assistant, go to **Settings → Add-ons → Add-on Store → ⋮ → Repositories**, add:

```
https://github.com/MarcelMuechler/vorrat-hassio-addon
```

Then install "Vorrat" from the store.

This repo only contains the add-on packaging (`config.yaml`, `Dockerfile`). The app itself
is built from [MarcelMuechler/vorrat](https://github.com/MarcelMuechler/vorrat) at image build
time — see [`vorrat/DOCS.md`](vorrat/DOCS.md) for usage and mobile app setup.
