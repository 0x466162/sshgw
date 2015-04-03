Proof of Concept (one day maybe more) of a SSH 'proxy' gateway to
channel and control all ssh connections at a single point.

#### ssh setup
    Host target
    	ProxyCommand	ssh -l _sshgw gateway %r@%h

#### sshd setup on gateway
    Match User _sshgw
            X11Forwarding no
            AllowTcpForwarding yes
            ForceCommand /home/_sshgw/sshgw
            PermitTTY no
            AuthorizedKeysFile /etc/sshgw.authorized_keys

#### create _sshgw user
This step heavily depends on your OS but be sure that your _sshgw user
can only login via public keys, i.e. in OpenBSD set a encrypted password
of 13 asterisks via vipw(8)

#### create AuthorizedKeysFile
    touch /etc/sshgw.authorized_keys
    chown root:_sshgw /etc/sshgw.authorized_keys
    chmod 640 /etc/sshgw.authorized_keys

#### create and populate the database
See SCHEMAS.md

#### login
    ssh -luser target
