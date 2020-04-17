#### Apache2_4::AuthCookieMultiDBI 
- An AuthCookie module backed by a DBI database for apache 2.4.

###### CPAN
https://metacpan.org/pod/Apache2_4::AuthCookieMultiDBI

###### VERSION

    This is version 0.4


###### SYNOPSIS

    # In httpd.conf or .htaccess

    # Optional: Initiate a persistent database connection using Apache::DBI.
    # See: http://search.cpan.org/dist/Apache-DBI/
    # If you choose to use Apache::DBI then the following directive must come
    # before all other modules using DBI - just uncomment the next line:
    #PerlModule Apache::DBI


    PerlModule Apache2_4::AuthCookieHandler
    PerlSetVar WhatEverPath /
    PerlSetVar WhatEverLoginScript /login.pl

    # Optional, to share tickets between servers.
    PerlSetVar WhatEverDomain .domain.com

    # These must be set
    PerlSetVar WhatEverDBI_DSN "DBI:mysql:database=test"
    PerlSetVar WhatEverDBI_SecretKey "489e5eaad8b3208f9ad8792ef4afca73598ae666b0206a9c92ac877e73ce835c"

    # These are optional, the module sets sensible defaults.
    PerlSetVar WhatEverDBI_User "nobody"
    PerlSetVar WhatEverDBI_Password "password"
    PerlSetVar WhatEverDBI_UsersTable "users"
    PerlSetVar WhatEverDBI_UserField "user"
    PerlSetVar WhatEverDBI_PasswordField "password"
    PerlSetVar WhatEverDBI_UserActiveField "" # Default is skip this feature
    PerlSetVar WhatEverDBI_CryptType "none"
    PerlSetVar WhatEverDBI_GroupsTable "groups"
    PerlSetVar WhatEverDBI_GroupField "grp"
    PerlSetVar WhatEverDBI_GroupUserField "user"
    PerlSetVar WhatEverDBI_EncryptionType "none"
    PerlSetVar WhatEverDBI_SessionLifetime 00-24-00-00
    perlSetVar WhatEverDBI_URIRegx "^/(.+)/(.+)$" # if have uri pattran like /client_id/file_name.pl
    perlSetVar WhatEverDBI_URIClientPos 0 # client_id position in uri
    perlSetVar WhatEverDBI_LoadClientDB 1 # do you have seperate database for each cleint

    # Protected by AuthCookieDBI.
    <Directory /www/domain.com/protected>
        AuthName WhatEver
        AuthType Apache2_4::AuthCookieMultiDBI
        PerlAuthenHandler Apache2_4::AuthCookieMultiDBI->authenticate
        require valid-user
    </Directory>

    # Login location.
    <Files LOGIN>
        AuthType Apache2_4::AuthCookieMultiDBI
        AuthName WhatEver
        SetHandler perl-script
        PerlResponseHandler Apache2_4::AuthCookieMultiDBI->login

        # If the directopry you are protecting is the DocumentRoot directory
        # then uncomment the following directive:
        #Satisfy any
    </Files>

###### DESCRIPTION

This module is an authentication handler that uses the basic mechanism provided
by Apache2_4::AuthCookie with a DBI database for ticket-based protection. Actually
it is modified version of L<Apache2::AuthCookieDBI> for apache 2.4. It
is based on two tokens being provided, a username and password, which can
be any strings (there are no illegal characters for either).  The username is
used to set the remote user as if Basic Authentication was used.

On an attempt to access a protected location without a valid cookie being
provided, the module prints an HTML login form (produced by a CGI or any
other handler; this can be a static file if you want to always send people
to the same entry page when they log in).  This login form has fields for
username and password.  On submitting it, the username and password are looked
up in the DBI database.  The supplied password is checked against the password
in the database; the password in the database can be plaintext, or a crypt()
or md5_hex() checksum of the password.  If this succeeds, the user is issued
a ticket.  This ticket contains the username, an issue time, an expire time,
and an MD5 checksum of those and a secret key for the server.  It can
optionally be encrypted before returning it to the client in the cookie;
encryption is only useful for preventing the client from seeing the expire
time.  If you wish to protect passwords in transport, use an SSL-encrypted
connection.  The ticket is given in a cookie that the browser stores.

After a login the user is redirected to the location they originally wished
to view (or to a fixed page if the login "script" was really a static file).

On this access and any subsequent attempt to access a protected document, the
browser returns the ticket to the server.  The server unencrypts it if
encrypted tickets are enabled, then extracts the username, issue time, expire
time and checksum.  A new checksum is calculated of the username, issue time,
expire time and the secret key again; if it agrees with the checksum that
the client supplied, we know that the data has not been tampered with.  We
next check that the expire time has not passed.  If not, the ticket is still
good, so we set the username.

Authorization checks then check that any "require valid-user" or "require
user jacob" settings are passed.  Finally, if a "require group foo" directive
was given, the module will look up the username in a groups database and
check that the user is a member of one of the groups listed.  If all these
checks pass, the document requested is displayed.

If a ticket has expired or is otherwise invalid it is cleared in the browser
and the login form is shown again.


###### EXPORTS

None.

###### REVISIONS

Please see the enclosed file CHANGES.

###### PROBLEMS?

If this doesn't work, let me know and I'll fix the code. Or by all means send a patch.
Please don't just post a bad review on CPAN.

###### SEE ALSO

L<Apache2::AuthCookieDBI>: L<Apache2_4::AuthCookie>.

###### AUTHOR

berlin3, details -at- cpan -dot- org.

###### COPYRIGHT

Copyright (C) details, 2018, ff. - All Rights Reserved.

This library is free software and may be used only under the same terms as Perl itself.
