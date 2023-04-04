# Hackthebox - Forensics - TrueSecrets

### Our cybercrime unit has been investigating a well-known APT group for several months. The group has been responsible for several high-profile attacks on corporate organizations. However, what is interesting about that case, is that they have developed a custom command & control server of their own. Fortunately, our unit was able to raid the home of the leader of the APT group and take a memory capture of his computer while it was still powered on. Analyze the capture to try to find the source code of the server.

#### We're give a memory dump to analyse. I will use volatility2 to do so.

#### We need to determine what profile to use:
![Screenshot 2023-04-04 at 18 12 12](https://user-images.githubusercontent.com/73375576/229853520-2a211268-56f8-4149-bd56-813be22984f4.png)

#### Running process list we see TrueCrypt app running.
![Screenshot 2023-04-04 at 18 20 48](https://user-images.githubusercontent.com/73375576/229855179-3399044e-9f26-44bc-9e7d-e764f8f75993.png)

#### Volatility has special plugins to grab info from TrueCrypt services.

![Screenshot 2023-04-04 at 18 08 44](https://user-images.githubusercontent.com/73375576/229856185-92540170-856c-40f5-9ef7-203791610cc6.png)

#### $ python2 vol.py -f TrueSecrets.raw --profile=Win7SP1x86_23418 truecryptsummary
![Screenshot 2023-04-04 at 18 07 13](https://user-images.githubusercontent.com/73375576/229858041-d55fab9a-3ad7-43c2-a173-6a29bd964856.png)
#### And get a password for  virtual encrypted disk by TrueCrypt, the disk seems to be stored in development.tc file.

#### Running filescan plugin we see a backup_develpment zip file !
![Screenshot 2023-04-04 at 18 54 36](https://user-images.githubusercontent.com/73375576/229862982-9bb98e0c-8c70-438b-b204-5179b7a2a959.png)
#### Let's grab that with:
#### - $ python2 vol.py -f TrueSecrets.raw --profile=Win7SP1x86_23418 dumpfiles -Q 0x000000000bbf6158 --dump-dir dump
#### - cd dump && touch backup_development.zip
#### - mv *.dat backup_development.zip
#### - unzip it and we get the development.tc file

#### To inspect it we need to mount the disk file on a container, use the password we recovered earlier.
![Screenshot 2023-04-04 at 17 24 42](https://user-images.githubusercontent.com/73375576/229872448-76e5bc87-0def-40b5-9bf2-0e233fa2a38f.png)

#### We go to our container and see some interesting stuff.
![Screenshot 2023-04-04 at 17 25 01](https://user-images.githubusercontent.com/73375576/229873474-d1c45d66-90a3-4ed0-a238-d60e2805bb67.png)

#### Inspecting the files we see some base64 strings but they seem to be encoded with some extra hash.
![Screenshot 2023-04-04 at 17 36 11](https://user-images.githubusercontent.com/73375576/229873835-24646d8f-75cd-4f1f-97a6-9a37e37cc0d3.png)

#### Inspecting the source code there is an encoding function that first encrypts the string using DES encryption then encode it in base64.
![Screenshot 2023-04-04 at 17 36 48](https://user-images.githubusercontent.com/73375576/229873855-0215a94a-7c08-4ae8-9484-20efa0b1658c.png)

#### Let's bake the recipe with CyberChef.
![Screenshot 2023-04-04 at 17 39 59](https://user-images.githubusercontent.com/73375576/229875385-08f349e4-f0bb-4c34-9c09-93a5b94307a7.png)
####  And grab flag. 😈
