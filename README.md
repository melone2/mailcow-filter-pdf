# mailcow-filter-pdf

This repo lists the steps neccecary in order to activate a sieve filter in mailcow, which filters all emails based on the attachment file extension.
A usecase could be forwarding all E-Mails with a .pdf attachment to a separate mailbox for processing by a document management system (e.g. paperless-ngx).

# Dovecot config

Go into your mailcow docker folder (where the `update.sh` script is).
Open `./data/conf/dovecot/dovecot.conf` with a text-editor of your choice (vim, nano).
Look for the `sieve_extensions` parameter.
Add the following options (if not already present): `+mime +foreverypart`

This is an example how this line could look like:
```
sieve_extensions = +notify +imapflags +vacation-seconds +editheader
```

Save and close the file.
Go back to your mailcow docker folder (where the `update.sh` script is).
Execute the following to restart the dovecot container:
```
sudo docker compose restart dovecot-mailcow
```

# Sieve parser

From your mailcow docker folder change to the following directory:
`./data/web/inc/lib/sieve`

Insert the files `mime.xml` and `foreverypart.xml` from this Github repository into that directory.
This will extend the mailcow sieve parser (which checks your sieve script before saving) to allow the inclusion of *mime* and *foreverypart* extensions in your sieve script.

No restarts are neccecary after this step.

# Sieve script

Now open the mailcow dashboard in your browser.
Go to *E-Mail > configuration > filter* and add or open a sieve-filter script for the desired mailbox.

Add the following content and adjust it to suit your needs (replace copycat@example.com):
```
require ["mime", "foreverypart", "fileinto", "imap4flags", "mailbox", "copy"];

foreverypart
{
  if header :mime :anychild :param "filename" :matches "Content-Disposition" ["*.pdf"]
  {
    redirect :copy "copycat@example.com";
  }
}
```

This script will then tell sieve to send a copy of every mail which contains at least one attached .pdf file to the specified E-Mail-address.

## Why shouldn't i check the mime type instead?

The provided script will check the actual filename of the attachement for the presence of a .pdf string at the end.
You could also change this script to check the mime type of attachements instead:
```
if header :mime :anychild :contenttype "Content-Type" "application/pdf"
```
However, it seems there are a few companies out there (Ebay) which will send attached .pdf files as mime-type `application/octet-stream` instead.
Checking the filename circumvents this issue.
