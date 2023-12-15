# Break it down! 
***
Traditionally, the goal of CTFs are trying to find hidden pieces of information, or activate some piece of code in order to export information. This is a bit different. The goal here is to bring down your target as quickly as possible. Seems simple, right? It may not be as easy as you might think!

This CTF revolves around CVE-2021-22205, a Gitlab exploit involving payloads masquerading as images. Images that were passed to a file parser were not being handled correctly, which resulted in the ability to remotely execute code without any authentication.

***
## Setup

Contained in this repository are all of the files necessary to perform this exploit. The container itself can (and probably will) get pretty resource hungry, so just be aware. Once the repository is cloned, the only command you need to run is `docker compose up`. Wait about a minute for the environment to fully initialize, then you are good to move on to the next step.

***
## Exploiting

This is where the fun begins. Remember, our goal is to bring down the server. The file `ctf_poc.py` is designed to deliver payloads to the server via post request. What is contained in those payloads are commands that the server will execute! From here, you need to get cracking. Time is of the essence!

`ctf_poc.py` is ran as such

`python ctf_poc.py http://<your-ip>:8080 <payload>`

Observer what effects your commands have on the container by observing the resource statistics! Bear in mind that it may take a second to observe the effect that your payload has on the server. Tear it all to pieces!

***
## Solution

As you may have discovered, it was a bit trickier than you may have expected. The main constrain with this vulnerability is that this method of posting payloads results in said payloads being recognized as the Gitlab user. This user doesn't have near the permissions that regular users do, so you are pretty constrained in what you can do. While a CPU attack such as a fork bomb might be your first stop, evaluating what you can do is the first step. Simply creating a file is the simplest start.

`touch ~/example`

The issue here is that it doesn't work. You need to find a place that has permissions for you to be able to work with. This is where `/tmp` comes in. With a permission setting of 1777, you can do whatever you want in here save for renaming/removing. This is also helpful, because you can figure out just *who* you are running commands as. By piping the output of `whoami` into a text file, you are able to see that you aren't the normal user. You've got to your commands in a place where permissions are very open, and your commands have to be pretty low level.

This is where the actual exploits can be performed. In my solution, I chose to completely fill the disk. I did so by writing all available space on the disk with this command.

***Take caution when executing this command. Its goal is to write tons of null information to disk***
`python ctf_poc.py http://192.168.1.57:8080 "dd if=/dev/zero of=/tmp/largefile2 bs=1M count=68719476736"`

Once this command is executed, it will take some time to see the effects, but the login page that was once available will no longer be served, and you will have issues viewing files within docker desktop.
