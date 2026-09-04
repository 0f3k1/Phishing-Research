A family member recently got caught up in one of the more interesting phishing campaigns I’ve come across.

What made this one so convincing was that the email did not come from some random or obviously suspicious address.

The family member had recently requested a school transcript and received an email about it from the legitimate school email address of someone they knew in the guidance department. It even included that person’s normal email signature.

Right person. Right email address. Right context. And most importantly, it was something they were actually expecting to receive.

So why would they be suspicious?

The email contained a link labeled:

“Download Transcript Record PDF”
<img width="1071" height="389" alt="image" src="https://github.com/user-attachments/assets/59c0889e-df92-4896-8374-dfa6a9efc987" />

When the family member initially tried opening the link on an iPhone, the page simply returned “Access Denied.”

They contacted the school because they thought something was wrong. The school later responded that their IT department had identified the email as part of a phishing campaign and that other people were experiencing the same thing.

So, being curious, I went to check it out.

Even though the sender was legitimate, there were still warning signs in the email itself.

The wording and grammar felt slightly off. More importantly, there was a complete lack of personalization. The family member was never addressed by name, the student was never mentioned by name, and there was nothing identifying who the supposed transcript was actually for.

There was also no prior mention of needing to install or use any new software, which became much more important later.

For something as specific and personal as a student transcript, that stood out.

Before clicking the link on a Windows machine, I wanted to understand where it was actually taking the user. Looking at the URL, I noticed it was going through a redirect, so instead of following it normally in a browser, I moved over to my Kali Linux VM and started tracing the redirect chain manually using curl.

For research purposes, as I go through the redirect chain I’ll use example1, example2, and so on in place of the actual domains.

The original link was a Google redirect URL, but decoding it showed that the first real destination was:

example1[.]com

When I requested that page without rendering it in a browser, the response was only 88 bytes of HTML containing:

window.location.href = "[https://example2[.]com/top](https://example2[.]com/top)"

So rather than using a normal HTTP redirect, the page was using JavaScript to send the browser to another domain.

When I then attempted to inspect that second destination, Bitdefender blocked the request and classified the site as suspicious.

This was also an interesting reminder about URL reputation tools. Some scanners initially showed little or no concern with parts of the link chain, while another security product flagged the next-stage domain.

A URL being marked “clean” by several scanners doesn’t automatically mean the entire redirect chain is safe.

Eventually, I decided to see what the link would actually deliver on my Windows machine.

Instead of the “PDF” the email advertised, the link resulted in a download named:

ScreenConnect.ClientSetup.exe

You'd think that if you request a transcript, it should just be a simple PDF, right...

This is where I went a little deeper than I originally planned 😅

With the help of ScreenConnect documentation and ChatGPT, I started pulling apart the executable.

What surprised me was that the executable itself appeared to be legitimate ScreenConnect remote-access software. The file contained ScreenConnect/ConnectWise metadata and had a valid digital signature.

Inside the EXE was an embedded MSI installer. Digging through its configuration revealed preconfigured ScreenConnect connection parameters, including:

e=Access
y=Guest
h=[redacted ScreenConnect relay]
p=443
k=[redacted]

The exact instance and configuration values aren't important here. What caught my attention was that the installer already contained the information needed to associate the client with a specific ScreenConnect environment.

And this is where something clicked for me.

There didn’t necessarily need to be some hidden piece of custom malware inside the executable.

The ScreenConnect software itself could be legitimate.

The URL chain could still be malicious.

The phishing email could still be malicious.

And the configuration suggested that the victim was being directed to install legitimate remote-access software that had already been configured for a particular ScreenConnect environment.

That also helps explain why a security product could flag the website or delivery infrastructure even though the executable itself was legitimately signed software.

Those are two different things.

The biggest lesson from the whole experience, though, had nothing to do with reverse engineering.

A legitimate sender does not automatically make the contents of an email legitimate.

This email came from a legitimate school email address.

It appeared to come from someone the recipient knew.

It was about something they had actually requested.

It arrived when they were expecting it.

Yet the email itself still had warning signs: no names, no personalization, unusual wording, and a link claiming to provide a PDF that ultimately delivered an executable.

It’s easy to say, “Don’t click links from random email addresses.”

It’s much harder when the email comes from someone you trust and is about something you were already expecting.

So even when an email comes from somewhere familiar, take a few extra seconds to actually read it before clicking anything.

Sometimes the sender isn’t the red flag.

Sometimes the red flags are hiding in the details.

P.S. I don't even want to get into how a phishing email ended up being sent from a legitimate guidance counselor's email account... because that could lead into an entirely different rabbit hole.

I also came across an interesting analysis from Forcepoint X-Labs covering a different attack involving ScreenConnect. Although it’s not the same campaign, there are some similar characteristics in how legitimate ScreenConnect software was used for remote access.

Worth checking out if you’re interested:
https://www.forcepoint.com/blog/x-labs/screenconnect-attack
