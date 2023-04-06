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

TO BE CONTINUED... (machine crashed)
