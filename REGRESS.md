## Create dummy values
### Groups
INSERT INTO groups VALUES('group_s');
### Hosts
INSERT INTO hosts VALUES('host_s','group_s');
INSERT INTO hosts VALUES('host_m',NULL);
### ACL
INSERT INTO access VALUES('foo','group_s',NULL);
INSERT INTO access VALUES('admin','*',NULL);
INSERT INTO access VALUES('moo',NULL,'host_m');

## Test cases
### foo@host_s
SELECT DISTINCT(a.user) FROM access a, hosts h WHERE
  (a.user="foo" AND a.groupname='*') OR
  (a.user="foo" AND a.groupname=h.memberof AND h.fqdn="host_s") OR
  (a.user="foo" AND a.hostname="host_s")
/
==> Granted (groupmembership)
### foo@host_m
SELECT DISTINCT(a.user) FROM access a, hosts h WHERE
  (a.user="foo" AND a.groupname='*') OR
  (a.user="foo" AND a.groupname=h.memberof AND h.fqdn="host_m") OR
  (a.user="foo" AND a.hostname="host_m")
/
==> Denied
### foo@host_x
SELECT DISTINCT(a.user) FROM access a, hosts h WHERE
  (a.user="foo" AND a.groupname='*') OR
  (a.user="foo" AND a.groupname=h.memberof AND h.fqdn="host_x") OR
  (a.user="foo" AND a.hostname="host_x")
/
==> Denied

### admin@host_s
SELECT DISTINCT(a.user) FROM access a, hosts h WHERE
  (a.user="admin" AND a.groupname='*') OR
  (a.user="admin" AND a.groupname=h.memberof AND h.fqdn="host_s") OR
  (a.user="admin" AND a.hostname="host_s")
/
==> Granted (access to all hosts)
### admin@host_m
SELECT DISTINCT(a.user) FROM access a, hosts h WHERE
  (a.user="admin" AND a.groupname='*') OR
  (a.user="admin" AND a.groupname=h.memberof AND h.fqdn="host_m") OR
  (a.user="admin" AND a.hostname="host_m")
/
==> Granted (access to all hosts)
### admin@host_x
SELECT DISTINCT(a.user) FROM access a, hosts h WHERE
  (a.user="admin" AND a.groupname='*') OR
  (a.user="admin" AND a.groupname=h.memberof AND h.fqdn="host_x") OR
  (a.user="admin" AND a.hostname="host_x")
/
==> Granted (access to all hosts)

### moo@host_s
SELECT DISTINCT(a.user) FROM access a, hosts h WHERE
  (a.user="moo" AND a.groupname='*') OR
  (a.user="moo" AND a.groupname=h.memberof AND h.fqdn="host_s") OR
  (a.user="moo" AND a.hostname="host_s")
/
==> Denied
### moo@host_m
SELECT DISTINCT(a.user) FROM access a, hosts h WHERE
  (a.user="moo" AND a.groupname='*') OR
  (a.user="moo" AND a.groupname=h.memberof AND h.fqdn="host_m") OR
  (a.user="moo" AND a.hostname="host_m")
/
==> Granted (specifically to host_m)
### moo@host_x
SELECT DISTINCT(a.user) FROM access a, hosts h WHERE
  (a.user="moo" AND a.groupname='*') OR
  (a.user="moo" AND a.groupname=h.memberof AND h.fqdn="host_x") OR
  (a.user="moo" AND a.hostname="host_x")
/
==> Denied
