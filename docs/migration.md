# Migration

Suggested order:
1. Build new VPS.
2. Deploy PostgreSQL, coturn, and ejabberd.
3. Validate login, MUC, upload, push, and calls.
4. Migrate accounts and any required archives.
5. Switch DNS.
6. Keep old server available until smoke tests pass.
