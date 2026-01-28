# Architecture Decision Records & Runbooks

## Architecture Decision Records (ADRs)

### Full Template
```markdown
# ADR-001: Use PostgreSQL for Primary Database

## Status

Accepted

## Context

We need to select a database for our application. Requirements include:
- ACID compliance for transactional integrity
- Support for complex queries and joins
- JSON storage for flexible schemas
- Robust ecosystem and tooling
- Production-proven at scale

We evaluated: PostgreSQL, MySQL, MongoDB.

## Decision

We will use PostgreSQL as our primary database.

## Consequences

### Positive
- Strong ACID compliance ensures data integrity
- JSONB type allows flexible schema evolution
- Excellent query optimizer for complex joins
- Mature ecosystem (ORMs, monitoring, backups)
- Strong community support

### Negative
- Requires careful index management for performance
- More complex than NoSQL for simple key-value needs
- Vertical scaling limitations (horizontal requires Citus)

### Neutral
- Team needs to learn PostgreSQL-specific features
- Need to set up connection pooling (PgBouncer)

## Implementation Notes

- Use Prisma ORM for type-safe queries
- Configure connection pooling with PgBouncer
- Set up automated backups to S3
- Monitor slow queries with pg_stat_statements

## Related
- ADR-002: Use Prisma as ORM
- ADR-005: Database Backup Strategy

## Date

2024-01-15
```

### ADR Status Options
- **Proposed** - Under discussion
- **Accepted** - Approved and in effect
- **Deprecated** - No longer recommended
- **Superseded** - Replaced by another ADR (link to it)

---

## Runbooks

### Full Template
```markdown
# Runbook: Database Backup Restoration

## Overview
How to restore a PostgreSQL database backup in case of data loss or corruption.

## Prerequisites
- Access to backup S3 bucket
- Database credentials (in 1Password)
- SSH access to database server
- Estimated time: 15-30 minutes

## Impact
- **Severity**: High
- **Downtime**: 10-20 minutes
- **User impact**: Service unavailable during restoration

## Step-by-Step Procedure

### 1. Notify Team
```bash
# Post in #engineering Slack channel
"Starting database restoration. ETA: 20 minutes. Service will be unavailable."
```

### 2. Download Backup
```bash
# List available backups
aws s3 ls s3://backups/postgres/

# Download latest backup
aws s3 cp s3://backups/postgres/db-2024-01-15.sql.gz ./backup.sql.gz

# Verify integrity
sha256sum backup.sql.gz
# Compare with S3 metadata checksum
```

### 3. Stop Application Servers
```bash
# Prevent new connections
kubectl scale deployment api --replicas=0
```

### 4. Restore Database
```bash
# Decompress backup
gunzip backup.sql.gz

# Restore
psql -h localhost -U postgres -d mydb < backup.sql

# Verify row counts
psql -h localhost -U postgres -d mydb -c "SELECT COUNT(*) FROM users;"
```

### 5. Start Application Servers
```bash
kubectl scale deployment api --replicas=3
```

### 6. Verify Service
```bash
# Health check
curl https://api.example.com/health

# Test critical flows
# 1. User login
# 2. Data retrieval
# 3. Data creation
```

### 7. Post-Restoration
- Update incident log
- Notify team of completion
- Schedule post-mortem if needed

## Rollback
If restoration fails:
```bash
# Revert to previous backup
aws s3 cp s3://backups/postgres/db-2024-01-14.sql.gz ./backup.sql.gz
# Repeat restoration steps
```

## Troubleshooting

### "Permission denied" error
- Check database user permissions
- Ensure user has SUPERUSER role

### "Disk space full" error
- Check available disk space: `df -h`
- Clean up temp files: `rm -rf /tmp/pg_restore_*`
- Use different restore location

## Related Documents
- [Database Backup Procedure](runbooks/database-backup.md)
- [Incident Response Guide](playbooks/incident-response.md)
- [Database Architecture](docs/architecture/database.md)

## Contacts
- On-call Engineer: See PagerDuty
- Database Team: #database-team
- Escalation: CTO

## History
- 2024-01-15: Initial version
- 2024-02-01: Added verification steps
```

### Key Runbook Principles
1. **Write for 3 AM** - Clear enough to follow when tired
2. **Include all commands** - Copy-pasteable, no guessing
3. **Add verification steps** - Confirm each step worked
4. **Document rollback** - Always have an exit strategy
5. **Keep contacts current** - Who to call for help
