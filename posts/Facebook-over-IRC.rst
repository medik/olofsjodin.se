.. title: Facebook over IRC
.. slug:  facebook-over-irc
.. date: 2017-08-04 01:10:00 UTC+02:00
.. tags: 
.. category: 
.. link: 
.. description: 
.. type: text

How to get bitlbee running with the facebook plugin
---------------------------------------------------

Back in the days, when I didn't have a badass computer to chat with my friends
on facebook, I spoke with my friends over XMPP i.e. jabber. This was possible
due to the fact that facebook ran on XMPP. Sadly, this is not true
today. Facebook decided to shut down their XMPP support. However, some people
decided that it was a good idea to reverse engineer the Facebook Messenger to
make their own implementation of the facebook messenger protocol. And now their
is a plugin to Bitlbee, which is an IRC server which its purpose is to bridge
the instant messenger world like jabber, ICQ, Yahoo and so on, so now it is
possible to speak with people over IRC. Pretty cool huh? I will guide you into
the big "how".

I am doing this setup on an Ubuntu Linux 16.04 LTS Server, but the way should be
similar in other Debian like operating systems.

Begin by adding this to your /etc/apt/sources.list::
	
  deb http://download.opensuse.org/repositories/home:/jgeboski/xUbuntu_16.04 ./
  deb http://code.bitlbee.org/debian/master/xenial/amd64/ ./

Then, run these commands to retrieve their keys::
	  
  wget -O- https://jgeboski.github.io/obs.key | sudo apt-key add -
  wget -O- https://code.bitlbee.org/debian/release.key | sudo apt-key add -

Update your APT packages by running::
  
  apt update	

Install bitlbee and the facebook add-on with apt::

  apt install bitlbee bitlbee-facebook

Make sure that the bitlbee service is on::
	  
  sudo service bitlbee start

Now it should be up and running. Start your irc client and connect to localhost.
Go to the &bitlbee channel, make sure that the facebook plugin is loaded by
writing "plugins".

To add your account, make sure that you have generated an app password. You can
do that here:
https://www.facebook.com/settings?tab=security&section=per_app_passwords&view

Use the generated password by running the following command in the `&bitlbee` channel::
	  
  account add facebook <username/email> <password>

Finally, run "account on" to connect. Congratulations, you should be able to
talk to your facebook friends over IRC.

