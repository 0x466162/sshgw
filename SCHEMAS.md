DROP TABLE IF EXISTS groups;
DROP TABLE IF EXISTS hosts;
DROP TABLE IF EXISTS access;
CREATE TABLE groups (name TEXT PRIMARY KEY NOT NULL);
CREATE TABLE hosts (
  fqdn TEXT NOT NULL,
  memberof TEXT,
  FOREIGN KEY(memberof) REFERENCES groups(name),
  UNIQUE(fqdn,memberof)
);
CREATE TABLE access (
  user TEXT NOT NULL,
  groupname TEXT,
  hostname TEXT,
  FOREIGN KEY(groupname) REFERENCES groups(name),
  FOREIGN KEY(hostname) REFERENCES hosts(fqdn),
  UNIQUE(user,groupname,hostname)
);
