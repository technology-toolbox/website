---
title: 'Is "hgsLZ7Rm" a good password? Apparently not (anymore).'
date: 2021-10-04T08:13:36-06:00
categories: ["Development", "Infrastructure", "My System"]
description: Your passwords are probably not as secure as you think they are.
images:
  [
    "https://assets.technologytoolbox.com/screenshots/66/45EA6D5A31ADAE93C914117C826D91FA20115F66.png",
  ]
tags: ["Development", "Infrastructure", "My System", "Security"]
---

This morning, I woke up to find the following "delightful" e-mail from Google in
my inbox:

{{< figure
  src="https://assets.technologytoolbox.com/screenshots/66/45EA6D5A31ADAE93C914117C826D91FA20115F66.png"
  class="screenshot" height="862" width="628"
  title="Figure 1: E-mail notification for password found in data breach" >}}

[See full-sized image.](https://assets.technologytoolbox.com/screenshots/66/45EA6D5A31ADAE93C914117C826D91FA20115F66.png)

Apparently a randomly generated password I was using (for the past two years on
a website I access somewhat regularly) was found in a recent data breach. The
password was something like `hgsLZ7Rm`.

> **Note**
>
> Out of an abundance of caution, I have changed one character in the password
> to avoid exposing the actual password in this blog post. Since I have already
> changed the password on the website, this probably isn't necessary, but it
> just didn't _feel_ right to divulge the original password.

A password like `hgsLZ7Rm` certainly _seems_ secure, doesn't it? After all, this
random password:

- is at least 8 characters in length
- contains a mix of uppercase and lowercase characters
- contains at least one number

Furthermore, according to [Use A Passphrase](https://www.useapassphrase.com/),
this password has a relatively high _entropy_ (i.e. approximate crack time is
**3 centuries**).

However, none of that really matters -- once the password is exposed in a data
breach. The mere fact that this password is out there on the Internet in one or
more lists means it is subject to brute force attack.

Sure, in my particular case, the website is an e-commerce site that doesn't have
my actual credit card number -- in other words, even if someone did manage to
hack my account, about the only thing he or she could access is my order
history. Yes, my name and home address would also be visible -- and the hacker
would already have to know my e-mail address in order to login to the website.
There are much easier ways of accessing those pieces of personal data rather
than hacking someone's password.

So, what do I recommend instead of generating random 8-character passwords
containing uppercase and lowercase letters and numbers? As I mentioned before,
this password was last updated roughly two years ago. At some point during that
time, I started using _passphrases_ instead.

I've been using various password managers for years. For accounts that I
consider highly sensitive, I used to create passwords via
[Use A Passphrase](https://www.useapassphrase.com/) -- typically editing the
passphrases slightly to shorten them a little and adding some random numbers.
Several years ago, I migrated most of my passwords from
[PasswordMinder](https://docs.microsoft.com/en-us/archive/msdn-magazine/2004/july/security-briefs-mind-those-passwords)
to [Bitwarden](https://www.bitwarden.com). At some point since I switched to
Bitwarden, the ability to create passphrases was added. In other words, instead
of passwords comprised of random _characters_ and _letters_, I recommend
(somewhat lengthy) passphrases comprised of random _words_.

[The word list used by Bitwarden](https://github.com/bitwarden/jslib/blob/master/common/src/misc/wordlist.ts)
contains nearly 8,000 items. By default, it generates passphrases with three
words separated by a dash (`-`). Here is an example:

> `doorman-blatancy-litter`

Copying and pasting that passphrase into
[Use A Passphrase](https://www.useapassphrase.com/) tells me the approximate
crack time is **206,632,886 centuries**. Wow!

Bitwarden also includes **Capitalize** and **Include Number** options when
generating passphrases, so -- if you are really paranoid -- you could instead
choose to create passphrases like:

> `Dodgy-Carnivore-Skype4`

The crux here is that the length of a password is, generally speaking, much more
important than the characters used within the password. So, all those websites
out there that make you create a password with, say, 8-12 characters and at
least one uppercase character, one lowercase, one number, and perhaps even a
"special character" like `!` or `@` are actually, well..._crap_.

Okay, perhaps _crap_ is a little strong, but if you build software that limits
me to creating a password to _only_ 12 characters, then I'll be the first to
tell you are probably not very knowledgeable when it comes to security. You
_owe_ it to your customers to allow them to use lengthy passphrases instead of
short randomly generated passwords that could end up being much easier to hack.

Personally, I typically generate passphrases using the default Bitwarden options
and then insert a two-digit random number (between 10 and 99) somewhere in the
passphrase. I feel this "gives me the best of both worlds" -- meaning, I am
confident the passphrase is sufficiently complex while at the same time is
relatively easy to type in manually, if necessary. Usually, I simply copy/paste
passwords from Bitwarden, but there are occasionally times when I have to look
up a password in Bitwarden and then enter it on some other device (for example,
when adding a device to a wireless network.)

Try typing something like `hgsLZ7Rm` and then try typing
`doorman-48-blatancy-litter`. I think you'll find the latter to be much easier
despite the fact that it is three times longer.

For websites that I do not consider to be highly sensitive, I will allow Google
Chrome to store the password. Another option would be to use the Bitwarden
browser extension. However, my risk tolerance precludes this. Admittedly, I'm
more paranoid than some but also less paranoid than others. You must find the
tradeoff between convenience and security that is right _for you_.

If you are worried that one or more of your passwords might have been
compromised in a data breach -- or you are worried about this happening in the
future -- then I recommend using one of the paid versions of Bitwarden which
includes the
[**Exposed Passwords Report**](https://bitwarden.com/help/article/reports/#exposed-passwords-report).
Note that this feature essentially iterates through your passwords stored in
Bitwarden and checks them against Troy Hunt's
["Have I Been Pwned?"](https://haveibeenpwned.com/) list of exposed passwords.

> **Note:**
>
> It's probably worth noting that your passwords stored in Bitwarden are never
> actually sent to Troy's website. Rather this is accomplished by attempting to
> partially match a hash of each password in Bitwarden against a list of hashes
> for exposed passwords.
