# bower-send

A simple multi-account mail sending script designed for Bower.

This program is designed to be used as the 'sendmail' script for Bower. It is
capable of sending mail from multiple accounts, each with different settings
(sendmail, postmail hooks, etc.), based on the 'from:' field of the message. It
can also be used as a standalone multi-account mail sender with post-send hooks.

## INSTALLATION

Simple pip, easy_install, and setup.py installations are forthcoming. For now,
the easiest thing to do is probably to `git clone` this repository and `ln -s
bower-send/bower-send /usr/local/bin/`.

## OPTIONS

    -d - Debug. Use this to test your configuration without sending mail.

## QUICK START

To start using bower-send, do the following:

1.  Install bower-send. Simply download and place the program in your $PATH.

2.  Add 'accounts' to your ~/.config/bower/bower.conf file. They must have
    the following form:

        [account.<name>]
          from_address = <address-in-from:-field>
          sent_folder = <folder-to-place-sent-mail-in>
          sendmail = <command-to-run-to-send-message>
          post_sendmail = <command-to-run-after-send>
          post_post = <command-to-run-after-post_sendmail>

        [account.<name>]
            ...

    `bower-send` must be invoked with a message to send in stdin; Bower will do
    this for its sendmail hook. bower-send will look at the from: header in the
    message and match the email address there with one of the 'account.' blocks
    in your configuration. It will then send the message using 'sendmail',
    perform the 'post_sendmail' hook, and if it exists, perform the 'post_post'
    hook.

    Note that 'sendmail', 'post_sendmail', and 'post_post' are optional,
    while 'from_address' and 'sent_folder' are required. The defaults are:

        sendmail = msmtp -t --read-envelope-from
        post_sendmail = notmuch insert --folder=$sent_folder +sent -new
        post_post = <undefined: do nothing>

    Note the special $sent_folder syntax. This will be replaced with your
    `sent_folder` configuration parameter. It may be used in the following
    configuration parameters: "post_sendmail", "sendmail", and "post_post".

3. Set the 'sendmail' Bower configuration parameter to 'bower-send'.

4. Set the 'post_sendmail' Bower configuration parameter to empty.

5. Done!

## EXAMPLES

The following examples illustrate multiple configurations:

    [account.simple]
      from_address = sb@simple.com
      sent_folder = simple/Sent

    [account.home]
      from_address = sergio@myhome.net
      sent_folder = home/Sent
      post_sendmail = notmuch insert --folder=$sent_folder
      post_post = afew -t -n

    [account.work]
      from_address = sbenitez@work.org
      sent_folder = work/Sent Items
      sendmail: msmtp -t --account=work
      post_sendmail = notmuch insert --folder=$sent_folder
      post_post = notmuch tag +work -new -- tag:new

The 'simple' account illustrates the minimally required configuration. Any mail
that is sent from the 'sb@simple.com' address will be sent using the msmtp
account associated with that address. The default postmail hook is that same as
Bower's, albeit specific to the account: notmuch insert is called with the
'sent_folder', the 'sent' tag is added, and the 'new' tag is removed.

The 'home' account adds the 'post_sendmail' and 'post_post' configuration items.
In this case, the call to notmuch insert does not add the 'sent' tag or remove
the 'new' tag. The 'post_post' hook, which is run after the 'post_sendmail'
hook, calls 'afew', which will tag any 'new' mail via a set of rules. I
personally use a configuration much like this one.

The 'account.work' directly specifies the sendmail command. It's 'post_post'
hook directly calls notmuch to add the 'work' tag to the just-sent mail and
remove the 'new' tag.

# QUESTIONS, COMMENTS, FEEDBACK

Please open a Github issue for questions, comments, or feedback.

# CAVEATS

You'll need python3 to run this program.
