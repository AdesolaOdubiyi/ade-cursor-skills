# Infrastructure and Config Checklist

Apply this checklist in a single dedicated pass to all Secondary surfaces.
This pass is lighter than the application code pass — flag clear violations,
do not perform deep adversarial analysis on config files.

## Environment and Secrets
- [ ] No .env files with real values are committed to the repository
- [ ] .env.example contains only placeholder values
- [ ] Secret references in config point to environment variables, not hardcoded values
- [ ] No credentials appear in docker-compose or CI/CD configuration files

## Container and Infrastructure Config
- [ ] Containers do not run as root unless explicitly required and documented
- [ ] No unnecessarily broad port exposure in docker-compose or network config
- [ ] Base images are pinned to specific versions, not latest
- [ ] No debug modes or development flags are active in production config

## HTTP and Network
- [ ] Security headers are configured: Content-Security-Policy, X-Frame-Options,
      X-Content-Type-Options, Strict-Transport-Security
- [ ] CORS configuration is restrictive — not wildcard unless explicitly required
- [ ] HTTPS is enforced — no plaintext fallback in production config
- [ ] Admin endpoints are not publicly exposed without auth

## CI/CD and Deploy Scripts
- [ ] No secrets are hardcoded in GitHub Actions, build scripts, or deploy scripts
- [ ] CI/CD secrets are injected via the platform's secret management, not env files
- [ ] Build scripts do not pull from unversioned or unverified external sources
- [ ] Deploy scripts do not have unnecessarily broad permissions
