# MariaDB Event Management

An individual database-design assignment by Johan Snis, exploring an event system
with attendees, venues, tickets, employees, and sponsorships.

Read the [original coursework report](Databas-VG-event.md), written in Swedish.
It includes a Mermaid ER diagram, schema and sample-data SQL, a ticket-revenue
view, a revenue stored procedure, EXPLAIN examples, roles, and backup/restore notes.

## Scope and status

This repository showcases the report and design reasoning. It is not an application
or a verified one-command database installation. SQL blocks include intentionally
incorrect normalization examples as well as implementation examples; do not execute
the whole Markdown document as SQL.

The original report is retained as coursework and needs a technical review before
being used as a guide. Known issues include a malformed quotation in the event seed
data, an index example targeting an ordinary view, incomplete procedure delimiter
setup, and normalization explanations that need refinement. The global administrator
grant and database deletion examples are historical lab examples, not setup steps.

Embedded passwords were replaced with placeholders, backup commands prompt for a
password, and sample email domains were changed to example.com. No database dumps,
server account files, or full Git history are included. No live database validation
was performed during publication.

See [ATTRIBUTION.md](ATTRIBUTION.md) for provenance.
