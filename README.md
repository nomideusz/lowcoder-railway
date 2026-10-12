# Deploy and Host Lowcoder on Railway

[![Deploy on Railway](https://railway.com/button.svg)](https://railway.com/new/template/lowcoder?referralCode=sLjDXb&utm_medium=integration&utm_source=button&utm_campaign=lowcoder)

[Lowcoder](https://lowcoder.cloud/) is the open-source Retool alternative: build internal tools, admin panels, dashboards and customer-facing apps with a drag-and-drop editor, connect them to your databases and APIs, script them with JavaScript, and share or embed the results. This template runs the official multi-image Lowcoder build (web frontend, API service and Node service) with MongoDB and Redis, and creates your super-admin account on first boot with a generated password.

## About Hosting Lowcoder

The stack is five services: Lowcoder (the web frontend), lowcoder-api, lowcoder-node, MongoDB and Redis.

- **Upstream's own images, pinned.** All three Lowcoder services run the official `lowcoderorg/lowcoder-ce-*` images at a fixed release (2.7.6), not `latest`, so a redeploy never surprises you with an untested version. Database migrations run automatically when the API service boots, so upgrading is bumping the tag on all three.
- **Only the frontend is public.** The Lowcoder service serves the web app and proxies `/api` to the API service over Railway's private network. The API service, Node service, MongoDB and Redis have no public domain.
- **Proxy that follows redeploys.** The frontend's nginx resolves the API and Node services per request (instead of once at startup), so redeploying the API service never leaves the frontend pointing at a dead address.
- **No default credentials.** The super-admin is created from `LOWCODER_SUPERUSER_USERNAME` (your email, asked at deploy time) and a generated `LOWCODER_SUPERUSER_PASSWORD`. Every other secret upstream ships a public default for is generated per deployment: the database encryption password and salt, the API key signing secret, the API-to-Node service secret, and the MongoDB and Redis passwords.
- **Everything in MongoDB.** Apps, queries, data source credentials (encrypted) and uploaded files (GridFS) live in MongoDB 7 with authentication on, on its own volume. Redis keeps sessions in an append-only file on its own volume. The Lowcoder services themselves are stateless.

## Common Use Cases

- Admin panels and back-office tools on top of Postgres, MySQL, MongoDB, REST or GraphQL APIs
- Internal dashboards and reporting for ops, support and sales teams
- CRUD apps, approval flows and forms without building a frontend from scratch
- Customer-facing portals and embeddable apps (Lowcoder apps can be embedded in other sites)

## Dependencies for Lowcoder Hosting

- MongoDB 7 (included, private network only)
- Redis 8 (included, private network only)

### Deployment Dependencies

- [Lowcoder documentation](https://docs.lowcoder.cloud/)
- [Lowcoder on GitHub](https://github.com/lowcoder-org/lowcoder)
- [Template source on GitHub](https://github.com/nomideusz/lowcoder-railway)

### Implementation Details

**Signing in.** When the deploy is green, open the Lowcoder service's Railway domain and sign in with the email you entered as `LOWCODER_SUPERUSER_USERNAME` and the password from `LOWCODER_SUPERUSER_PASSWORD` (lowcoder-api service, Variables tab). The super-admin is an admin of every workspace.

**The super-admin password is managed by the variable.** The API service re-applies `LOWCODER_SUPERUSER_PASSWORD` on every boot, so a password changed in the UI reverts on the next redeploy. To change it, edit the variable.

**Sign-ups.** Email sign-up is on, as upstream ships it, because invite links need it. In the default SAAS workspace mode every person who signs up gets their own isolated workspace (they cannot see yours), but anyone who finds the URL can create one. To close the instance, set `LOWCODER_EMAIL_SIGNUP_ENABLED=false` on lowcoder-api; the super-admin can still sign in.

**Custom domain.** Add it to the Lowcoder service, then set `LOWCODER_PUBLIC_URL` on lowcoder-api to `https://your-domain` so invite links and OAuth callbacks use it.

**Email (optional).** Invitations and password resets by email need SMTP: fill in the `LOWCODER_ADMIN_SMTP_*` variables on lowcoder-api. Railway only allows outbound SMTP on the Pro plan.

**Back up MongoDB and the encryption secrets.** `LOWCODER_DB_ENCRYPTION_PASSWORD` and `LOWCODER_DB_ENCRYPTION_SALT` encrypt the data source credentials stored in MongoDB. Changing or losing them makes every saved credential unreadable.

**Resources.** Out of the box the Java API service and the Node service each size their memory off the host and idle at 800-900 MB. This template caps them so every service fits in 512 MB: `JAVA_OPTS` (256 MB heap, C1-only JIT) on lowcoder-api and `NODE_OPTIONS` (`--max-old-space-size=384`) on lowcoder-node, which also runs node directly instead of through `yarn start`. In testing with a 512 MB limit per service the API service idled at about 370 MB, the Node service at 200-250 MB, MongoDB at about 85 MB, and the whole stack at about 700 MB. If your apps pull large query results, raise `-Xmx` on lowcoder-api (for example to `768m`) and drop `-XX:TieredStopAtLevel=1` on the Hobby plan.

## Why Deploy Lowcoder on Railway?

Railway is a singular platform to deploy your infrastructure stack. Railway will host your infrastructure so you don't have to deal with configuration, while allowing you to vertically and horizontally scale it.

By deploying Lowcoder on Railway, you are one step closer to supporting a complete full-stack application with minimal burden. Host your servers, databases, AI agents, and more on Railway.
