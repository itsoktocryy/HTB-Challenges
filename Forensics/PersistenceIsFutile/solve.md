# Hackthebox - Forensics - PersistenceIsFutile
### Hackers made it onto one of our production servers 😅. We've isolated it from the internet until we can clean the machine up. The IR team reported eight difference backdoors on the server, but didn't say what they were and we can't get in touch with them. We need to get this server back into prod ASAP - we're losing money every second it's down. Please find the eight backdoors (both remote access and privilege escalation) and remove them. Once you're done, run /root/solveme as root to check. You have SSH access and sudo rights to the box with the connections details attached below.

#### We're given an ip/port and creds to login into the victim's ssh service.

#### Running $ ps aux will show all running processes. /connectivity-check process looks shady. $sudo cat on it and we see a reverse shell. 
![1](https://user-images.githubusercontent.com/73375576/230368847-8631ab87-9570-4b93-be94-a7475968c5bb.png)

#### Let's kill the running process and $ rm -rf the file.
![2](https://user-images.githubusercontent.com/73375576/230373017-e9de75cf-4f26-445b-9841-a2a8e7ed7dd7.png)

#### Running $ ps aux as root revealed another sus process using port 4444.
![3](https://user-images.githubusercontent.com/73375576/230374316-cfa4eba0-4cb4-412f-a4b1-4d2159b66f8f.png)

#### Let's locate the binary.
![4](https://user-images.githubusercontent.com/73375576/230375597-94b2790c-f4a4-4969-a486-34711f9e2f98.png)

#### And see where is it getting called from, we know it should come from root.
![5](https://user-images.githubusercontent.com/73375576/230376603-c72eeb31-7921-4611-90df-400671c5f388.png)

#### Knowing that we have a bad .bashrc, we can actually just overwrite it with the skeleton .bashrc stored in /etc/skel.
![6](https://user-images.githubusercontent.com/73375576/230378771-042da8bb-44b8-4ae3-96a4-854aae609768.png)

#### Don't forget to kill all related processes.
![7](https://user-images.githubusercontent.com/73375576/230378785-6917dc8a-f587-4869-afad-6fbc62ce0057.png)


#### Let's check cron jobs running on the machine.
![8](https://user-images.githubusercontent.com/73375576/230389490-0239edac-6cfb-4d90-8963-3b84d6894407.png)
#### We see that this cronjob uses the $ dig command which is well known to be used for DNS query records. This is definately sus. Remove it by typing $ crontab -l and removing that entry.

#### Running crontab -l as root doesn't show any jobs but that doens't mean there are none. Navigating to /etc and looking throught various cron folders there were a few that didnt match being there on a linux distro
![10](https://user-images.githubusercontent.com/73375576/230393925-69a267a2-cc8a-4e28-a3eb-960b563d1044.png)

#### Let's dig into access-up and pyssh:
#### In access-up cron job we see a chmod 4755 which is a huge red flag because that means we’re giving this binary SetUID permission… aka root permission!. So if an attacker, was to execute the random binary we generated it would actually spawn an instance of /bin/bash owned by the user root. DELETE the file now !
![11](https://user-images.githubusercontent.com/73375576/230397614-c618cc79-e005-44ab-8193-405e1edbcd47.png)
#### This script is adding whatever KEY is to PATH. They’re base64 encoded so let’s just run the commands and see what’s inside, it's being used to add a public SSH key to the root user’s authorized_keys file. Meaning they’ll be able to SSH in as root whenever they want.
![12](https://user-images.githubusercontent.com/73375576/230503522-a5b81a2f-469a-4e24-8914-a72ba781331c.png)

#### Earlier we found a script that gives to files SetUID permissions, let's find those. Go ahead and remove all sus binaries.
![13](https://user-images.githubusercontent.com/73375576/230505473-b761c543-fa09-42f7-9df3-d0b8ebb9e4c8.png)
![14](https://user-images.githubusercontent.com/73375576/230506218-71915c01-bf84-469d-88dc-29c6b42cb0d8.png)

#### Let's check /etc/passwd for any user account with elevated privileges. 'gnats' user is looking suspicious by not owning the correct permissions everywhere. We can fix this as:
![15](https://user-images.githubusercontent.com/73375576/230506301-60112c20-df2f-4597-98f3-64687af231ce.png)

