Audit trails for PostgreSQL tables.  Run this script in your
PostgreSQL database and use the output to create them.

To start auditing all of database db as user u:

```
psql -AtX -o actually_create_audit.sql -c 'SELECT create_audit();' -U u db
$EDITOR actually_create_audit.sql # Sanity check.
psql -X -1f actually_create_audit.sql -U u db
```

This here's a neat one-liner, but unless one-lining is an explicit
part of your job description, don't do this:

    psql -AtX -c 'SELECT create_audit()' -U u db | psql -X -1 -U u db

If you want to audit in smaller chunks than the entire database, read
on.

Auditing, as modeled here, consists of several components, some of
which have executable components.

- The table to be audited, hereinafter the audited table.  Changes to
  the structure of this table need to be done with some care as to how
  they will cascade through the auditing system.  In general,
  information-losing changes such as dropping a column or reducing a
  field's length are more dangerous and need to be thought through
  more carefully than information-gaining ones.

- The audit table, each row of which contains the old and new versions
  of the audited table's rows along with some information which may be
  helpful when doing an actual audit.

- An audit trigger function for each table being audited.  This
  captures the old and new versions of the rows, as applicable, along
  with some context data including the transaction time, marks
  the operation that caused the change, and writes all these do the
  audit table.

- The audit trigger itself.
