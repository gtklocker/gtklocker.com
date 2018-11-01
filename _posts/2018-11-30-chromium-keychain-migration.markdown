---
layout: post
title:  "Migrating Chromium from KWallet to GNOME Keyring"
date:   2018-11-01 23:30:00 +0300
---

I recently found myself wanting to make the switch from KDE to GNOME. I
remembered though that Chromium extensively uses the keyring; I've had it
seemingly forget all my cookies in the past because of the keyring not being
available. These cookies were irrecoverable so I had to log in to every website
on earth once again. I really dreaded having to do this ever again so I
starting cracking at the problem.

# Migrating the whole keychain

At first I thought: how about migrating the whole keychain? Since the keychain
was not only used by Chromium but also by NetworkManager to store WiFi passwords
this only made sense. However a little search on the internet revealed that
[the two keyrings are pretty
incompatible](https://unix.stackexchange.com/questions/5005/gnome-keyring-kwallet-integration)
and while there's interest to fix this no progress has been made.

# Popping the hood on Chromium's keyring usage

Then I thought something else: maybe I can use Chromium on GNOME but have it
utilize KWallet. Thankfully [Chromium provides us with the option of which
keyring to
choose](https://chromium.googlesource.com/chromium/src/+/master/docs/linux_password_storage.md)
which made my life a little easier. After logging in to GNOME, opening KWallet
via KWalletManager and starting Chromium with the `--password-store=kwallet` I
still had all my cookies. This was nice but I didn't like having to keep two
keyrings so I kept digging.

I opened KWalletManager to see exactly how Chromium was using my keyring,
figuring out that I could maybe transfer over the important stuff manually
somehow.

![KWalletManager showing Chromium Safe Storage]({{ site.url
}}/assets/chromium-keychain-migration/kwalletmanager.png)

*Chromium Safe Storage* seems to be the key that `~/.config/chromium` is encrypted with.

We also see lots of form data, but let's ignore these for now.

After backing up the browser's data with `cp -r ~/.config/chromium{,.bak}` I
decided I should open the browser making it use the GNOME keyring, so I used
`--password-store=gnome` this time. I then promptly shut down the browser and
proceeded to check with Seahorse what Chromium had created.

Sure enough, the same *Chromium Safe Storage* entry was there, but the value
was different. Sure, Chromium derived a new encryption key. I then copied over
the key from KWalletManager replacing the one on Seahorse. 

Then I moved the previous Chromium data back with `rm -rf ~/.config/chromium &&
cp -r ~/.config/chromium{.bak,}` and reopened Chromium. No arguments needed since
it'll pick up the GNOME keyring automatically if it runs on GNOME.

And sure enough, my cookies were all there and I was still logged in everywhere. **Phew!**

Remember the form data we ignored earlier? The form data will sync
automatically from the Google account pretty quickly, no need to copy those
over.
