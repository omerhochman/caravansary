# Brand assets

The canonical logo files live in [`assets/brand`](./assets/brand).

## App/repo surfaces

Update these when the logo changes:

- `assets/brand/logo.svg` - primary source file.
- `assets/brand/icon.svg` - icon-only source file.
- `assets/brand/logo.png` - PNG export for services that reject SVG.
- `assets/brand/icon.png` - PNG icon export for services that reject SVG.
- `README.md` - uses `assets/brand/logo.svg`.

If an app is added later, point it at `assets/brand/logo.svg` or copy from that directory during its build/deploy step. Do not keep a separate hand-edited logo copy in the app.

## External surfaces

These must be changed manually:

- Google OAuth consent screen branding.
- GitHub OAuth app logos.
- GitHub organization avatar.
- npm organization avatar.
- Stripe branding.
- Resend sender branding.
- Sentry project icon, if used.

Use the SVG when the service accepts it. Otherwise upload `assets/brand/logo.png` or `assets/brand/icon.png`.
