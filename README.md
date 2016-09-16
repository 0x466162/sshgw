Proof of Concept (one day maybe more) of a SSH 'proxy' gateway to
channel and control all ssh connections at a single point.

The concept I am currently working on is to based on the following corner stones:

  *  access to backends is granted only via signed keys
  *  access is granted only if channeled through the gateway
  *  the user himself has no control over the keys that grant access to the backend
  *  unprivilegded user access is granted with a long living cert (e.g. 8h to cover a working day)
  *  privilegded user access is only granted via a short lived cert (e.g. 5m) so the window of opportunity is kept small for an attacker / mischief
  *  the user requests access to a system via a frontend (probably web based service) which will generate a signed key for a fixed amount of time
  *  the gateway grants access to specific systems only for any given cert

The current working state is using remote commands instead of proxy commands, I am undecided which of the two I want to have for the final product as both have different pros and cons.

This is a heavily moving target and not production ready by any means!

### ssh setup client side
    Host sshgw
    	User        _sshgw
        RequestTTY  force

### sshd setup on gateway
    AuthorizedKeysFile      none   # be careful there shouldn't be more than one global entry
    PasswordAuthentication  no
    TrustedUsersCAFile      /etc/ssh/ca.pub
    Match User _sshgw
            X11Forwarding       no
            AllowTcpForwarding  yes
            TrustedUserCAKeys   none
            AuthorizedKeysFile  /etc/ssh/sshgw.authorizedkeys

    Match User !root
            AuthorizedPrincipalsFile        /etc/ssh/principals

### setting up sshgw
This step heavily depends on your OS but be sure that your _sshgw user
can only login via public keys, i.e. in OpenBSD set an encrypted password
of 13 asterisks via  usermod(8) e.g.:

    adduser                             # create user, just use the system defaults
    usermod -p '*************' _sshgw   # make sure the user can never login via password
    $(umask 0027 ; >/etc/ssh/sshgw.authorizedkeys)
    $(umask 0022 ; >/etc/ssh/principals)
    echo unpriv > /etc/ssh/principals

#### filling /etc/ssh/sshgw.authorizedkeys

    printf "command=\"/usr/local/libexec/sshgw user\" %s" "$(cat /path/to/pubkey)" >> /etc/ssh/sshgw.authorizedkeys

#### create and populate the database

    $( umask 0027 ; >/var/db/sshgw.db )
    chgrp _sshgw /var/db/sshgw.db

For database setup see SCHEMAS.md

### setting up CA
Create a dedicated user and key for the CA. It's advisable to do so on a different machine.

    adduser
    usermod -p '*************' _sshca
    su -l _sshca
    umask 0077
    ssh-keycen -C CA -f .ssh/ca -t ed25519
    exit

Then distribut the ca.pub to the fleet under /etc/ssh/ca.pub with 0444 permissions.

#### sign keys

  * unprivileged access
    ssh-keygen -s .ssh/ca -I <keyid> -V+8h -n unpriv -z <serial> /path/to/key.pub
  * privileged access
    ssh-keygen -s .ssh/ca -I <keyid> -V+8h -n root -z <serial> /path/to/key.pub

### login

    ssh sshgw user@target
