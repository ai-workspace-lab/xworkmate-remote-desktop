# Flutter client extraction

`legacy/` is a source-preserving copy from `xworkmate-app`. It deliberately
keeps the original imports during the first extraction so behavior and tests
can be compared while a standalone package API is designed.

Next steps are to replace `AppController` and gateway imports with a signaling
interface, move reusable rendering/input components into a Flutter package,
and leave the XWorkmate-specific panel in `xworkmate-app` as an adapter.
