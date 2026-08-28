# Introduction


This report is the conclusion of **PMAT (Practical Malware Analysis & Triage)** course, which requires to analyze a real malware sample. I have chosen one of the most famous ransomware: WannaCry, which caused a lot of damage around the world in 2017.

Este informe es la conclusión del curso **PMAT (Practical Malware Analysis & Triage)**, en el que se pide analizar una muestra de malware real. Para ello he elegido uno de los ransomware más famosos: WannaCry, que causó estragos allá por 2017.

Among the wide variety of ransomware that exists, the two main categories would be: 
- Crypto ransomware: attacks encrypting user's valuable files and makes them unusable.
- Locker ransomware: blocks the access to the computer so it cannot be used.

De entre la gran variedad de ransomware que hay, las dos categorías principales diría que son: 
- Ransomware de cifrado: ataca cifrando archivos valiosos para que no se pueda acceder a ellos.
- Ransomware de bloqueo: bloquea el acceso al ordenador, impidiendo su uso.

**WannaCry** is a crypto ransomware with worm capabilities identified for the first time in May 2017. It propagates automatically in Windows systems using the protocol SMB thanks to the vulnerability known as EternalBlue (CVE-2017-0144). It also uses the backdoor DoublePulsar. When it is executed successfully on a vulnerable computer, it encrypts victim's files and shows a ransom note with the intention of extorting the users and oblige them to pay money in Bitcoin in order to restore the access to their files.

**WannaCry** es un ransomware de cifrado con capacidades de gusano identificado por primera vez en mayo de 2017. Se propaga de forma automática en sistemas Windows mediante el protocolo SMB aprovechando la vulnerabilidad conocida como EternalBlue (CVE-2017-0144). También emplea el backdoor DoublePulsar. Una vez ejecutado con éxito en un equipo vulnerable, cifra los archivos de la víctima y muestra una nota de rescate con la idea de extorsionar a los usuarios y que paguen dinero en Bitcoin con la promesa de que se les devuelva el acceso a sus archivos.



# Basic Static Analysis


## 1 - Sample Hashes

- *MD5*: `db349b97c37d22f5ea1d1841e3c89eb4`
    
- *SHA1*: `e889544aff85ffaf8b0d0da705105dee7c97fe26`
    
- *SHA256*: `24d004a104d4d54034dbcffc2a4b19a11f39008a575aa614ea04703480b1022c`
    
Searching for these hashes on VirusTotal reveals that the sample has been identified as WannaCry. The platform also provides additional information, including detection names, community analysis, etc:

Al buscar estos hashes en VirusTotal se observa que la muestra es identificada como WannaCry. Además, la plataforma proporciona información adicional, como los nombres de detección, el análisis de la comunidad, etc:

<img width="639" height="476" alt="imagen" src="https://github.com/user-attachments/assets/f098861f-f83a-4472-a841-03efce541449" />

  

## 2 - Strings

Several strings related to cryptography can be found in the sample:

Pueden encontrarse muchos strings relacionados con la criptografía:

```
CryptAcquireContextA
CryptGenRandom
Microsoft Base Cryptographic Provider v1.0
CryptReleaseContext
Microsoft Enhanced RSA and AES Cryptographic Provider
CryptGenKey
CryptDecrypt
CryptEncrypt
CryptDestroyKey
CryptImportKey
CryptAcquireContextA
WanaCrypt0r
```

These strings indicate that the sample makes use the Windows cryptographic API and that it is gonna use related operations, like key generation, encryption, decryption, and key management. This is coherent with the encryption functionalities of WannaCry, but the strings alone are not enough to figure out how the sample uses them.

Estos strings indican que la muestra usa la API criptográfica de Windows y que va a realizar operaciones relacionadas, como la generación de claves, el cifrado, el descifrado y la gestión de claves. Esto encaja con las funcionalidades de cifrado de WannaCry, aunque sin proporcionar información sobre su uso exacto.

Also, a suspicious URL can be found:

También se puede encontrar una URL sospechosa:  

```
hxxp[://]www[.]iuqerfsodp9ifjaposdfjhgosurijfaewrwergwea[.]com
```

It is known that this URL is related with the kill switch of WannaCry. WannaCry will try to connect to this domain at the beginning of its execution. If the connection is successful, the execution ends.

Es conocido que esta URL está asociada con el kill switch de WannaCry. WannaCry intenta conectarse a este dominio durante su ejecución. Si la conexión se establece correctamente, el malware finaliza su ejecución.


Also can be seen a wide list of the file extensions that the malware will target for encryption:

Aparece también una lista de las extensiones de archivo que el malware intentará cifrar:

<details>
<summary>List of targeted file extensions / Lista de extensiones objetivo</summary>

```text
.der
.pfx
.key
.crt
.csr
.p12
.pem
.odt
.ott
.sxw
.stw
.uot
.3ds
.max
.3dm
.ods
.ots
.sxc
.stc
.dif
.slk
.wb2
.odp
.otp
.sxd
.std
.uop
.odg
.otg
.sxm
.mml
.lay
.lay6
.asc
.sqlite3
.sqlitedb
.sql
.accdb
.mdb
.dbf
.odb
.frm
.myd
.myi
.ibd
.mdf
.ldf
.sln
.suo
.cpp
.pas
.asm
.cmd
.bat
.ps1
.vbs
.dip
.dch
.sch
.brd
.jsp
.php
.asp
.java
.jar
.class
.mp3
.wav
.swf
.fla
.wmv
.mpg
.vob
.mpeg
.asf
.avi
.mov
.mp4
.3gp
.mkv
.3g2
.flv
.wma
.mid
.m3u
.m4u
.djvu
.svg
.psd
.nef
.tiff
.tif
.cgm
.raw
.gif
.png
.bmp
.jpg
.jpeg
.vcd
.iso
.backup
.zip
.rar
.tgz
.tar
.bak
.tbk
.bz2
.PAQ
.ARC
.aes
.gpg
.vmx
.vmdk
.vdi
.sldm
.sldx
.sti
.sxi
.602
.hwp
.snt
.onetoc2
.dwg
.pdf
.wk1
.wks
.123
.rtf
.csv
.txt
.vsdx
.vsd
.edb
.eml
.msg
.ost
.pst
.potm
.potx
.ppam
.ppsx
.ppsm
.pps
.pot
.pptm
.pptx
.ppt
.xltm
.xltx
.xlc
.xlm
.xlt
.xlw
.xlsb
.xlsm
.xlsx
.xls
.dotx
.dotm
.dot
.docm
.docb
.docx
.doc
```
</details>


Strings related to the SMB protocol, and, probably, with the exploitation of EternalBlue:

Aparecen strings relacionados con el protocolo SMB, y, presumiblemente, con la explotación de EternalBlue:

```
\%s\IPC$
\172.16.99.5\IPC$
\192.168.56.20\IPC$
\192.168.56.20\IPC$
```


## 3 - PEStudio

### Imports

Analyzing the sample in PEStudio, we can first see that it was compiled with Microsoft Visual C++ v6.0 and is a 32-bit PE executable:

Al analizar la muestra en PEStudio se puede ver en primer lugar que fue compilado con Microsoft Visual C++ v6.0 y que se trata de un ejecutable PE de 32 bits:

<img width="371" height="151" alt="imagen" src="https://github.com/user-attachments/assets/8459f981-2a64-4db6-be7e-085aa35bbc7f" />

There are 91 imports listed, of which 30 are flagged as potentially dangerous or suspicious:

Figuran 91 imports, de los cuales 30 están marcados como potencialmente peligrosos o sospechosos:

<img width="356" height="249" alt="imagen" src="https://github.com/user-attachments/assets/f9c415be-38c2-4753-8556-5eed538d7aa9" />

Among them are several that use ordinal import, a technique in which a PE imports functions from a DLL using the function's ordinal number instead of their name. This is also suspicious because it can be a form of light obfuscation when certain APIs are called.

De entre ellos hay varios que presentan una importación ordinal, la cual es una técnica mediante la cual un PE importa funciones de una DLL usando el número ordinal de la función en lugar de su nombre. Esto también es sospechoso porque puede ser una forma de ofuscación ligera al llamar a ciertas APIs.

However, this kind of technique is not inherently malicious and could be used by legit software, because it reduces binary's size and the loading speed improves. But it is not common in modern software.

De todas maneras, este tipo de técnica no es necesariamente maliciosa y también puede darse en software legítimo ya que hace que los binarios puedan ser más pequeños y que la velocidad de carga mejore. Aunque no es común en software moderno.

<img width="817" height="234" alt="imagen" src="https://github.com/user-attachments/assets/024f9c65-4261-4ed8-beb8-f1342d731917" />

The names of these imports suggest that are related to socket operations. It is revealing that the called DLL is WS2_32.dll, Windows Socket Library, what it means, maybe, that those imports are related to the worm behaviour of wannacry.

Puede verse que por el nombre hacen referencia al uso de un socket, pero lo más revelador es que la DLL a la que llaman es WS2_32.dll, la Windows Socket Library, por lo que estos imports puede que sirvan al comportamiento de gusano que tiene el wannacry.

MalAPI is a useful website that classifies certain APIs often used by malware, and points what use can be given. These are the ones in the sample and identified as suspicious:

MalAPI es una página web muy útil que cuenta con una clasificación de determinadas apis a menudo usadas por malware, y señala qué uso se les suele dar. Estas son las presentes en la muestra y señaladas como sospechosas: 

<details>
<summary> Flagged imports classified in MalAPI / Clasificación en MalAPI de imports marcados </summary>
	
<img width="2848" height="4683" alt="imagen" src="https://github.com/user-attachments/assets/964e2e2f-e8d4-471d-a302-68708bf6634a" />

</details>

On one hand, APIs related to network communication:
- **GetAdaptersInfo**: commonly used to obtain data about network adapters in the system.
- **InternetOpenA, InternetOpenUrlA, InternetCloseHandle**: used to initialize Internet access, open a URL/resource, and release the associated handles.

On the other hand, related with encryption:
- **CryptAcquireContextA, CryptGenRandom**: cryptographic functions.

Finally, related with persistence:
- **CreateServiceA**
- **StartServiceA**
- **StartServiceCtrlDispatcherA**
- **OpenSCManagerA**


Por un lado, tenemos las relacionadas con la comunicación de red:
- **GetAdaptersInfo**: comúnmente usada para obtener información acerca de los adaptadores de red presentes en el sistema.
- **InternetOpenA, InternetOpenUrlA, InternetCloseHandle**: pueden utilizarse para inicializar el acceso a Internet, abrir una URL o recurso y liberar los handles asociados.

Por otra parte, relacionadas con la encriptación:
- **CryptAcquireContextA, CryptGenRandom**: funciones de carácter criptográfico.

Finalmente, relacionadas con la persistencia:
- **CreateServiceA**
- **StartServiceA**
- **StartServiceCtrlDispatcherA**
- **OpenSCManagerA**

### Second-stage payload

A 32-bit executable can be seen inside the sample, named as resource R:

Se aprecia que hay un ejecutable de 32 bits dentro de la muestra:

<img width="724" height="75" alt="imagen" src="https://github.com/user-attachments/assets/8f1fd095-87d7-4215-8bec-725822e4913c" />

This could indicate that Wannacry's first stage would act as a dropper, in other words, it contains an executable inside, which would be its second phase or second stage. Sample's second stage will be analyzed later, in its own section.

Esto podría indicar que la primera fase de Wannacry actuaría como un dropper, es decir, que contiene un ejecutable en su interior, el cual constituiría su segunda fase o segunda etapa. Se analizará esta segunda fase del malware más adelante, en un apartado propio.



# Basic dynamic analysis

## 1 - Network-based indicators

Initially, the sample tries to connect with the following URL, as a kill switch:

A modo de kill switch, intenta conectar al principio de la ejecución con la URL:

`hxxp[://]www[.]iuqerfsodp9ifjaposdfjhgosurijfaewrwergwea[.]com` 

<img width="805" height="154" alt="imagen" src="https://github.com/user-attachments/assets/8f0e555c-e989-4cf3-aea4-b9bee0550773" />

If the connection is successful, the malware stops its execution. That is what allowed to stop the attack back in May 2017, thanks to the researcher Marcus Hutchins, who registered this domain, helping to stop the global propagation of the ransomware.

Si la conexión es exitosa, el programa deja de actuar y no realiza ningún proceso más. Esto es lo que permitió parar el ataque, ya que el investigador Marcus Hutchins registró este dominio, ayudando a detener la propagación global del ransomware en mayo de 2017.

Then, if the connection is unsuccessful and the payload starts, a lot of network activity is detected, due to the worm functionality that wannacry has, expanding itself across the network. In the images can be seen, both in Wireshark and at the system process level, how the malware tries to connect with any possible system in the net, scanning through the different IPs on the network. Moreover, the port is always **445**. That is because the SMB protocol uses that port, **445**, and therefore that port should be used to the successful exploitation of **EternalBlue**.

Luego, si se empieza a ejecutar el payload, se observa que empieza a haber mucha actividad de red, debido a la funcionalidad de worm que tiene wannacry, expandiéndose por la red. Aquí puede verse, tanto en wireshark como a nivel de procesos del sistema, cómo intenta conectarse con el resto de posibles sistemas en la red, haciendo un barrido por las diferentes IPs de la red. Por otra parte, tenemos que el puerto al que apunta siempre es el **445**. Esto se debe a que el protocolo SMB opera en ese puerto, el **445**, y por lo tanto es al que se debe apuntar para la explotación de **EternalBlue**.

<img width="428" height="272" alt="imagen" src="https://github.com/user-attachments/assets/dc0fc126-9876-4481-a4ad-a79ecd31ad00" />

<img width="910" height="197" alt="imagen" src="https://github.com/user-attachments/assets/576e3455-c496-49dd-8feb-f06be9755146" />

Different connections to localhost can also be observed, involving the processes `taskhsvc.exe` and `@WanaDecryptor@.exe`.

Se inicia otra conexión con un proceso nuevo llamado `taskhsvc.exe` y otra con `@WanaDecryptor@.exe`, dirigidas al localhost.

<img width="892" height="78" alt="imagen" src="https://github.com/user-attachments/assets/3d851202-2e68-42f0-bd98-ec2e7e33e159" />

That process establish the port 9050 in listen mode for all interfaces:

Puede verse además que este proceso deja el puerto 9050 a la escucha para todas las interfaces:

<img width="894" height="66" alt="imagen" src="https://github.com/user-attachments/assets/92bb02bb-2f66-4516-9b83-75ef27875302" />

I tried to connect to that port using netcat, with no success.

He intentado conectarme a dicho puerto usando netcat, sin éxito.

After further research, I discovered that 9050 is the default port of the proxy SOCKS of Tor. As my personal assumption, this could be some kind of backdoor that allows the attacker to connect with the system using Tor network.

Tras investigar un poco, descubro que el puerto 9050 es usado por defecto por el proxy SOCKS de Tor. Como hipótesis personal, creo que esto podría ser un backdoor de algún tipo, que permitiese al atacante conectarse al sistema mediante el uso de la red Tor.

In order to make a try to capture the worm behaviour of WannaCry, I added to the virtual network a Windows 7 vulnerable virtual machine vulnerable to EternalBlue:

Para intentar captar el comportamiento de gusano de WannaCry, puse en la red virtual una máquina Windows 7 vulnerable a EternalBlue:

<img width="646" height="245" alt="imagen" src="https://github.com/user-attachments/assets/b5b64c0c-98f8-4b48-876b-11a321da8350" />

However, after several attempts, I did not detect the network propagation of the malware. That, probably, due to the low rate of success of EternalBlue. As a curiosity, I detected that one of the exploitation tries does not use as path the IP of the vulnerable virtual machine, but 192.168.56.20, which also was observed as an IPC$ path in the identified strings in static analysis:

Sin embargo, tras varios intentos, no se consiguió captar la propagación por la red del malware. Esto probablemente sea debido a que la vulnerabilidad EternalBlue no tiene una tasa de éxito demasiado elevada. Como curiosidad, detecté que uno de los intentos de explotación no usa como path la IP de la VM vulnerable, sino 192.168.56.20, lo cual aparece como una ruta IPC$ en los strings identificados en el análisis estático:

<img width="563" height="217" alt="imagen" src="https://github.com/user-attachments/assets/636fa03d-b363-40c5-9cec-2f96f92019c0" />

This could suggest that the malware was developed in virtualbox, using his host-only mode, because it is the range of IPs that uses by default: 192.168.56.0/24. But this is just a personal hypothesis.

Esto podría indicarnos que el malware fue desarrollado en virtualbox, usando su modo host-only, ya que es el rango de IPs que usa por defecto: 192.168.56.0/24. Aunque esto es sólo una hipótesis personal.


## 2 - Host-based indicators


Our greatest tool in this section is procmon. First of all, the sample is executed with administrator privileges and the name of process that the sample has, is established as a filter. As we saw in the basic static analysis, this malware's first phase acts as a dropper, so the procmon's filter *Operation is CreateFile* is a must in order to see the name of the file that the second stage will have and where it will be created. A lot of files will be seen with this filter, but that is because the API *CreateFile* is used both for to create new files and for to access to files in general

Nuestra mejor herramienta en esta sección es procmon. Para empezar, se ejecuta la muestra como administrador y se establece como filtro el nombre de proceso que tendrá el ejecutable. Como se ha visto, la primera fase de este malware actúa un dropper, por lo que se establece como filtro *Operation is CreateFile* en procmon y se podrá ver el nombre que se le da a la segunda fase del malware y dónde se creará. Al establecer este filtro se ven muchos archivos como objetivo de wannacry, pero eso es sólo porque la API *CreateFile* sirve tanto como para crear archivos nuevos como para acceder a archivos en general:

<img width="438" height="184" alt="imagen" src="https://github.com/user-attachments/assets/6c70220e-fb2f-482b-8407-b381448b7288" />

According to the capabilities of the API *CreateFile*, it is reasonable to think that the malware first of all verifies that exists a file called *tasksche.exe*, presumably his second stage, in the path *C:\Windows*. If it is not found, it will create it, as suggested by the two consecutive highlighted operations and their respective results.

Teniendo en cuenta las capacidades de la API *CreateFile*, es de suponer que el malware verifica primero la existencia de un archivo llamado *tasksche.exe*, el cual presumiblemente es la segunda fase, en la ruta *C:\Windows*, y si no lo encuentra, lo crea, como puede inferirse de las dos operaciones sucesivas remarcadas y su resultado.

<img width="305" height="222" alt="imagen" src="https://github.com/user-attachments/assets/63503066-ff92-4120-a062-20afb4e90c46" />

Having as a new clue the name of the second phase, it will be the next filter and the two previous filters are deleted. An operation involving a strange alphanumeric string can be seen:

Sabiendo ahora cómo se llama la segunda fase, se establece como filtro para seguir investigando. Puede verse que hay una operación con un string alfanumérico extraño:

<img width="426" height="106" alt="imagen" src="https://github.com/user-attachments/assets/f74375df-037c-4bcd-a868-3ba3157467b4" />

Which is a folder created by the payload:

El cual es una carpeta que creada por el payload:

<img width="401" height="178" alt="imagen" src="https://github.com/user-attachments/assets/1080e705-17a5-4a4d-a25c-fd9fb55a17b5" />

This directory contains many files, and among them the file of the second stage itself (`tasksche.exe`):

En dicha carpeta puede verse multitud de archivos, así como el archivo que conforma la segunda fase (`tasksche.exe`):

<img width="325" height="476" alt="imagen" src="https://github.com/user-attachments/assets/bbdbc383-59f3-4457-81c9-c134a4d7e3c6" />

Also 3 files with name 00000000. Analyzing the file with extension .pky it is shown that:

También aparecen 3 archivos de nombre 00000000. Al analizar el de extensión .pky, puede verse lo siguiente:

<img width="638" height="232" alt="imagen" src="https://github.com/user-attachments/assets/a52815c9-b827-4657-a006-0c2a4a3acbb0" />

The mention of the RSA cryptographic system and the similarity of the names suggest that these three files are related to the data encryption process.

La mención al sistema criptográfico RSA y la similitud de los nombres, hace pensar que estos tres archivos están involucrados en el proceso de cifrado de los datos.

An interesting outcome is shown with the name of the directory as a filter in procmon:

Al poner como filtro en procmon el nombre de la carpeta, se obtienen un resultado interesante:

<img width="486" height="52" alt="imagen" src="https://github.com/user-attachments/assets/795b135c-d004-4b9e-ac8b-4f07928460bf" />

The creation of a new register named as the directory:

La creación de un nuevo registro con el nombre de la carpeta creada y su posterior inicio. Yendo al registro:

<img width="602" height="206" alt="imagen" src="https://github.com/user-attachments/assets/e810bdf1-9433-40db-886e-8848b14e811d" />

Which enables a service with the same name that executes the second stage. This a persistence mechanism that will execute it every time that the system starts, encrypting the new files created after the initial infection, trying to spread the malware again, etc.

El cual sirve a un servicio con el mismo nombre y que se encarga de ejecutar el payload. Esto es un mecanismo de persistencia que se encargará de ejecutarlo cada vez que se inicie el equipo, cifrando todo nuevo archivo que el usuario haya creado tras la infección inicial, intentando esparcir de nuevo el malware, etcétera.

Within services can be seen the persistence service created, which is stopped in the beginning and has automatic startup. 

En efecto, en servicios puede verse el servicio de persistencia creado, el cual se encuentra detenido al principio y cuenta con inicio automático:

<img width="383" height="371" alt="imagen" src="https://github.com/user-attachments/assets/67fdd899-c8d8-4b47-bc9c-c2797c6a5bf6" />

Finally, the most obvious and evident host-indicators: almost all the are encrypted and unavailable. Moreover, the wallpaper was changed for another one with instructions to pay and that red window pop up with more detailed instructions for the payment:

Por último, los indicadores de host más claros y evidentes: casi todos los archivos quedan cifrados y no se puede acceder a ellos. Además, el fondo de pantalla cambia a uno con instrucciones para pagar y aparece esta ventana con instrucciones más detalladas para el pago:

<img width="1541" height="669" alt="imagen" src="https://github.com/user-attachments/assets/7e546c70-4d88-4908-b2bb-442329d195cd" />

While I was checking events in procmon in order to make this analysis, I realised that the window with instructions, named *Wana Decrypt0r 2.0*, pop up again and again every time when it was closed, being so annoying to the user. I found out that *tasksche.exe* always reopens the process. However, if *tasksche.exe* is killed, the windows will not open again so neither if the file *@WanaDecryptor@.exe* is deleted from the created directory by the malware.

Mientras comprobaba eventos en procmon para la realización de este análisis, me di cuenta de que por mucho de que cerrara la ventana con instrucciones, de nombre *Wana Decrypt0r 2.0*, siempre volvía a aparecer, resultando muy molesta al usuario. Comprobé que *tasksche.exe* vuelve a abrir siempre el proceso, apareciendo la ventana cada vez. Sin embargo, si se termina *tasksche.exe* no volverá a aparecer la ventana, y tampoco si se elimina el archivo *@WanaDecryptor@.exe* de la carpeta creada por el malware.

Researching in procmon about this executable, I found that executes this command when starts:

Investigando en procmon sobre este ejecutable, encontré que al iniciarse, ejecuta el siguiente comando:

<img width="838" height="368" alt="imagen" src="https://github.com/user-attachments/assets/092eeee3-3b90-411a-8cc2-e1df7b3a6eb5" />

```
cmd.exe /c  
vssadmin delete shadows /all /quiet  
& wmic shadowcopy delete  
& bcdedit /set {default} bootstatuspolicy ignoreallfailures  
& bcdedit /set {default} recoveryenabled no  
& wbadmin delete catalog -quiet
```

Looking at the command step by step:

Examinando el comando paso a paso:

- **vssadmin delete shadows /all /quiet**: deletes all the Volume Shadow Copies (backups of files, folders, or entire volumes) without confirmation
- **wmic shadowcopy delete**: deletes shadow copies through WMI. This provides another mechanism for removing Volume Shadow Copies.
- **bcdedit /set {default} bootstatuspolicy ignoreallfailures**: modifies Boot Configuration Data so that windows ignores boot errors and does not show options of automatic recovery
- **bcdedit /set {default} recoveryenabled no**: unables Window's recovery environment (WinRE), which unallows automatic repair and the restoration from the recovery environment
- **wbadmin delete catalog -quiet**: deletes backup's catalog of Windows Backup

- **vssadmin delete shadows /all /quiet**: elimina todas las Volume Shadow Copies (puntos de restauración del sistema) sin confirmación
- **wmic shadowcopy delete**: elimina las shadow copies mediante WMI. Esto proporciona otro mecanismo para eliminar las Volume Shadow Copies.
- **bcdedit /set {default} bootstatuspolicy ignoreallfailures**: modifica el Boot Configuration Data para que windows ignore errores de arranque y no muestre opciones de recuperación automática
- **bcdedit /set {default} recoveryenabled no**: deshabilita el entorno de recuperación de Windows (WinRE), lo cual impide la reparación automática y la restauración desde el entorno de recuperación
- **wbadmin delete catalog -quiet**: borra el catálogo de backups de Windows Backup

It is obvious that this executable takes care of the anti-recovery phase of the malware, thanks to this command. The malware not only encrypts the data, but also makes it difficult to recover.

Es evidente que este ejecutable se encarga de, entre otras cosas, la fase antirecuperación del malware, mediante la ejecución de este comando. El malware no sólo cifra los datos, sino que además dificulta su recuperación.


# Advanced static analysis


Thanks to advanced analysis, certain internal aspects of the sample can be showed. For example, the killswitch, seen  in the section of network-based indicators:

Mediante el análisis avanzado se pueden ver ciertos aspectos de la muestra a nivel interno. Por ejemplo, el killswitch comentado en la sección de indicadores basados en red:

<img width="637" height="632" alt="imagen" src="https://github.com/user-attachments/assets/1de95c47-3ee0-43b7-967e-c5582f30de49" />

In the red highlighted *1*, the string that contains the URL is provided to the ESI register so it can be used as an argument. The calls for functions in *2* that will connect to the killswitch URL. After that, if there are no response from the website, the program will continue running the instructions in *3* normally. But if the request is answered, the instructions in *4* will be executed an the malware will stop and exit without any encryption of data.

Puede verse en *1* que se pasa el string que contiene la url al registro ESI para que pueda ser usada luego como argumento. En *2* se ven las llamadas a las funciones que realizarán la comunicación web a dicha url. Luego, si no hay respuesta por parte de la web, irá por *3* y continuará ejecutándose de forma normal. Si hay respuesta, irá por *4* y el programa saldrá sin haber cifrado ninguno de los archivos.

It is also clear when it saves to disk his second stage, thanks to the different calls to APIs that performs in order to do that. First of all, it loads into register the strings of the API's calls that will do with the goal of to create a file and write into him.

Por otra parte, puede verse también cuando guarda en disco el ejecutable que lleva en su interior, por diferentes llamadas a APIs que realiza para este fin. Primero, se ve cómo carga en registro los strings de las llamadas que va a hacer a las APIs para crear un archivo y escribir en él:

<img width="612" height="368" alt="imagen" src="https://github.com/user-attachments/assets/e7c8c192-1186-4167-abd6-a87aeb974a52" />

Then, it uses these APIs to look for the resource, load it and obtain his pointer and size:

Luego, hace uso de estas APIs para buscar el recurso, cargarlo, obtener su puntero y su tamaño:

<img width="482" height="632" alt="imagen" src="https://github.com/user-attachments/assets/a5154c42-ff2b-42ed-9744-99b5163743a2" />

Finally, it is shown the future name of the file, the path where it will be saved, etc

Para finalizar, se ve el nombre que se le va a poner al archivo, además de, por ejemplo, la ruta en que se va a guardar, etc:

<img width="305" height="419" alt="imagen" src="https://github.com/user-attachments/assets/8d894136-c1cf-41a0-bd71-19d379e4f1b7" />


# Advanced dynamic analysis 


I tried to capture the moment of exploitation of EternalBlue and propagation of the malware, manipulating the worm behaviour of wannacry. Unfortunately, in my tests, the exploit always fails and does not infect my vulnerable VM. That's why I will try to research and dynamically find where EternalBlue fails and change the execution flow in order to obligue the malware to send the payload, which I will capture using wireshark.

He intentado manipular el comportamiento de worm de wannacry para intentar capturar el momento en que se intenta llevar a cabo la explotación de EternalBlue y la propagación de WannaCry. Ya que el exploit siempre falla en mis pruebas y no consigue infectar a la VM vulnerable, intentaré llegar dinámicamente a la parte en que falla EternalBlue y cambiar el rumbo de la ejecución para que mande el payload, el cual capturaré gracias a wireshark.

As starting point of the research, I will use the attempts of EternalBlue exploitation captured during the basic dynamic analysis section:

Como punto de partida para empezar a investigar, tengo los intentos de explotación de EternalBlue que he podido ver en la sección de análisis dinámico básico:

<img width="563" height="217" alt="imagen" src="https://github.com/user-attachments/assets/49900272-73df-4360-8867-35137943f419" />

Strings that contain `IPC` have already appeared in the basic static analysis section. I look for them using Cutter to know the memory address where those strings are loaded:

Strings que contienen la cadena `IPC` ya han aparecido en la sección de análisis estático básico, por lo que las busco mediante Cutter para saber la dirección de memoria en que se cargan esos strings:

<img width="369" height="73" alt="imagen" src="https://github.com/user-attachments/assets/7f9b2023-9bb4-4024-a7f9-037379cba7c5" />

Cross-references (X-Refs) are useful to show the instruction that loads the string into memory:

Usando las referencias cruzadas (X-Refs), se puede saber en qué instrucción se carga ese string en memoria:

<img width="767" height="409" alt="imagen" src="https://github.com/user-attachments/assets/770f5020-d8c5-4f82-a70e-6b5e1799f1c1" />

A good point of Cutter is that allow to change the name of the functions at will during the analysis. So, I moved up to the upper function and renamed it as `EternalBlue`.

Lo bueno de Cutter es que durante el análisis permite cambiar el nombre a las funciones a voluntad, por lo que subo en la jerarquía y renombro a la función como `EternalBlue`. 

Another strings that I discovered in the static analysis and could be useful are long alphanumeric string. I had not previously observed these strings being used during the execution flow:

Por otra parte, algo que había descubierto en el análisis estático eran largas cadenas alfanuméricas. Hasta ese momento, no había observado que estos strings se utilizasen durante el flujo de ejecución:

<img width="724" height="473" alt="imagen" src="https://github.com/user-attachments/assets/23a9ba11-3414-482c-8ab6-08e8dd3972e7" />

Under the suspicion of those strings might be related with the spreading of the malware, something that had not happened before, I redid the same steps in Cutter in order to determine which function would use them. I renamed that function as `Payload`. Moving up to the upper function it can be seen that both functions are very close.

Bajo la sospecha de que estos strings podían tener que ver con la propagación del malware, algo que no había pasado hasta ahora, realicé los mismos pasos en Cutter que para determinar cuál era la función que tomaría este papel. Renombré dicha funcióncomo `Payload`. Luego, subiendo en la jerarquía, veo que están muy cerca una de otra:

<img width="559" height="447" alt="imagen" src="https://github.com/user-attachments/assets/43f66340-bf1d-47f7-9acb-774116a8d7fb" />

The function `EternalBlue` (*1*) is very close and before the function `Payload` (*2*), which makes sense, because first of all the malware checks if the exploit works and then sends the payload. So, the conditional jumps in (*3*) and (*4*) are the ones that I need to manipulate. I set a breakpoint in `0x00407582` to control step by step the execution.

La función `EternalBlue` (*1*) se halla muy cerca y antes de la función `Payload` (*2*), lo cual tendría sentido, porque primero se comprueba que funciona el exploit y luego se manda el payload. Por lo tanto, los saltos condicionales en (*3*) y (*4*) son los que hay manipular. Establezco un breakpoint en `0x00407582` para controlar paso a paso la ejecución del programa.

However, the breakpoint never activates when the malware runs and automatically the debugger shows that the debugging ended, but all the other process were made: the second stage was released and the encryption started successfully. Having that in mind, and inspecting more carefully, I realised that after the successful execution of the killswitch (*1*), the PID of the process changes (*2*):

Sin embargo, el breakpoint nunca se activa, pues al correr el programa aparece en el debugger que la ejecución ha terminado, pero el resto de procesos se ha llevado a cabo, ya que el proceso de cifrado de archivos sí que se realiza. Con esto en mente y mirando más cuidadosamente, me he dado cuenta de que tras ejecutar el killswitch con éxito (*1*), el PID del proceso cambia (*2*):

<img width="651" height="196" alt="imagen" src="https://github.com/user-attachments/assets/287dcae5-821b-48d4-a61a-66ead44b553d" />

<img width="427" height="172" alt="imagen" src="https://github.com/user-attachments/assets/230c7a88-1581-4555-9932-d4a5259e3bd9" />

The details of the event show that the first stage of the malware is executed again with the flag `-m security`. Also the PID of the new process is the same seen in the last picture. However, I lose the control of the debugging process after the killswitch. The solution I found was to set a breakpoint in the call to Create Thread, stop the execution in any new thread created and check procmon every time. But the new process will run uncontrolled when starts, so the attachment should be quick and the process must be paused. The problem with this solution is that I have not any control over the initial moments of the execution of this new process. Nevertheless, I was fast enough to continue the investigation. Shortly after the new process starts, the old one ends:

En los detalles del evento puede verse que se ejecuta de nuevo la primera fase del malware con el flag `-m security`. Se puede ver también que el PID del nuevo proceso coincide con el mostrado en la imagen anterior. El problema que me surge es que pierdo el control de la ejecución en el debugger tras la comprobación del killswitch. La solución que he encontrado ha sido poner un breakpoint en la función que crea nuevos hilos para que se pare la ejecución en ese punto, comprobar procmon cada vez y vincular rápidamente el debugger x32dbg al nuevo proceso, y pausarlo una vez vinculado. El problema de esto es que no tengo control sobre las primeros momentos de la ejecución de este nuevo proceso. No obstante, fui lo suficientemente rápido como para poder continuar la investigación. Al poco de iniciarse el nuevo proceso, se termina el anterior:

<img width="260" height="134" alt="imagen" src="https://github.com/user-attachments/assets/8e0c8188-17e8-4294-a84a-a20f6ce13582" />

This time, after attaching to the new process, the malware does stop in the set breakpoint on `EternalBlue`. Doing Step Over over that function appears network traffic showing the exploitation try:

Ahora sí, tras asociar el debugger al nuevo proceso, se para la ejecución en el breakpoint fijado en la función `EternalBlue`. Al hacer Step Over sobre esa función, se ve el tráfico del intento de explotación:

<img width="825" height="215" alt="imagen" src="https://github.com/user-attachments/assets/61e30ccb-7651-41a7-a550-c3345de4285d" />

Keeping on with normal execution, the malware tries to take the next jump (*3*). I modify the value of Zero Flag (ZF) from 1 to 0 to prevent the jump:

Al seguir con la normal ejecución, veo que intenta tomar el salto siguiente (*3*). Modifico el valor de la Zero Flag (ZF) de 1 a 0 para que no tome el salto: 

<img width="559" height="447" alt="imagen" src="https://github.com/user-attachments/assets/2a982f09-9259-4caa-a1f0-7ebcdfa54c22" /><br/>

<img width="199" height="82" alt="imagen" src="https://github.com/user-attachments/assets/9a869c4e-81c1-4dc9-90e3-fac68c2b99ec" />

The next jump (*4*) is not taken. After reaching the function `Payload`, and to do Step Over over it, it can be seen SMB traffic that I did not detect before to my vulnerable VM:

El siguiente salto (*4*) no intenta tomarlo. Tras llegar a la función `Payload` y hacer Step Over sobre ella, puede verse tráfico SMB que no había detectado antes hacia mi máquina vulnerable:

<img width="718" height="228" alt="imagen" src="https://github.com/user-attachments/assets/93b34aa2-eeef-4e82-9edf-62e86659661b" />

Alphanumeric strings are sent in multiple SMB packets. The size of the sent data is higher than expected, which is 4096, in every packet. The following picture shows the symbols `==` at the end of the string in the last sent packet:

Se mandan las diferentes cadenas alfanuméricas en varios paquetes SMB. En cada paquete la cantidad de datos enviados es muy superior a la esperada, de 4096. Como puede verse en la siguiente imagen, que muestra el último paquete enviado, al final del string están los símbolos `==`:

<img width="488" height="440" alt="imagen" src="https://github.com/user-attachments/assets/bcb618d3-3add-42ba-b737-2eb8937f19be" />

The presence of the symbols `==` makes reasonable to think that the data is base64-encrypted. However, I tried to reconstruct the entire string according to the order of the sent packets, unsuccessfully.

La presencia de los símbolos `==` hace pensar que se trate de información cifrada en base64. Sin embargo, he intentado poner todas las cadenas una tras otra, siguiendo el orden en que son enviadas en un intento para desencriptar el payload enviado, sin éxito. 

After those strings, the malware sends data again:

Por otro lado, tras el envío de estas cadenas, el malware vuelve a mandar datos:

<img width="444" height="349" alt="imagen" src="https://github.com/user-attachments/assets/d35124a2-4abb-4ee5-b7fb-fbac922825d2" />

These packets does not contain any string that can make sense, it seems only hexadecimal data, and for that, it may be shellcode:

Estos datos no contienen ninguna cadena que pueda tener sentido, más bien parece que sólo manda información en hexadecimal, por lo que podría ser un shellcode:

<img width="541" height="33" alt="imagen" src="https://github.com/user-attachments/assets/dbab7b05-e5f8-4dfd-ba3a-6dd0629d22c2" />

<img width="490" height="305" alt="imagen" src="https://github.com/user-attachments/assets/62a533bc-26f1-434b-b459-a9bbd800ed1a" />

These data must combine in some way with the base64-string as the payload to infect other systems. Making some research I found that this kind of network activity is related with the use of DoublePulsar, a backdoor, capable of injecting shellcode or running DLLs into memory. My guess is that the apparently base64 string is used to implant DoublePulsar, while other packets are the malware itself. Therefore, WannaCry uses EternalBlue to open the door to the vulnerable system and after that DoublePulsar implants itself in order to inject the malware and to run it.

Estos datos han de combinarse de alguna manera con la cadena en base64 para la infección de otros equipos. Investigando, descubrí que este tipo de actividad de red es el que se ve con el uso de DoublePulsar, un backdoor, capaz de inyectar shellcode o ejecutar DLLs en memoria. Mi suposición es que la aparente cadena en base64 sirva para la implantación de DoublePulsar, mientras que los otros paquetes sean el malware en sí mismo. Por lo tanto, el malware ha de usar EternalBlue para abrir la puerta al sistema vulnerable y a continuación DoublePulsar se implanta para inyectar luego el malware y ejecutarlo.

DoublePulsar uses a not too complex XOR obfuscation, so knowing the used formula, which is public, it should be easy to obtain the sent payload. I manage to decrypt it using a PCAP file with the captured network traffic and the following script:
```
https://github.com/WithSecureLabs/doublepulsar-c2-traffic-decryptor/blob/master/decrypt_doublepulsar_traffic.py
```
I easily adapted the script to python 3 using ChatGPT.

DoublePulsar usa una ofuscación XOR no demasiado compleja, por lo que conociendo la fórmula que se usa para ello, la cual es pública, debería ser sencillo obtener el payload enviado. Consigo desencriptarlo usando para ello un archivo PCAP con el tráfico capturado y el siguiente script:
```
https://github.com/WithSecureLabs/doublepulsar-c2-traffic-decryptor/blob/master/decrypt_doublepulsar_traffic.py
```
Adapté sin problema el script a python 3 usando ChatGPT.

The output file from the script is a PE executable:

El archivo que da el script a la salida es un ejecutable PE:

<img width="460" height="234" alt="imagen" src="https://github.com/user-attachments/assets/2a5ebd5e-899c-45dc-9074-23f93414818a" />

Within this executable, the resource W can be observed. Due to its high entropy, it is highly probable that it could be another executable: 

Dentro de este ejecutable se halla el recurso W. Debido a su alta entropía, es muy posible que sea otro ejecutable:

<img width="933" height="307" alt="imagen" src="https://github.com/user-attachments/assets/7ee68248-68b4-4642-8655-9777fd819607" />

Inside the resource W I found the resource R, and the resource XIA within R. Whereby, resource W is the first stage of the malware.

Dentro del recurso W encuentro a su vez al recurso R, y al recurso XIA dentro de éste último. Por lo cual, el recurso W es la primera fase del malware.

<img width="507" height="466" alt="imagen" src="https://github.com/user-attachments/assets/f137c840-8464-4729-b9d4-bab4b637f0b2" />

Mixing static and dynamic analysis I had determine which functions are used by the malware to exploit EternalBlue and to spread the malware across the network, as well as to capture all that traffic with wireshark and to reconstruct the sent payload.

Combinando análisis estático y dinámico he logrado determinar las funciones que llevaban a cabo tanto la explotación de EternalBlue como el traspaso del malware a la nueva máquina infectada, así como captar todo este tráfico en wireshark y reconstruir el payload enviado.

# Second-stage payload

Using PEStudio it can be seen a suspicious resource, called R, which is an executable. It is the second phase of the malware and as we know in this point of the analysis, will have *tasksche.exe* as his name. I saved it for analysis.

Analizando la muestra con PEStudio puede verse que el recurso R es un ejecutable, el cual es la segunda fase y que tendrá por nombre *tasksche.exe*. Gracias a PEStudio puedo guardarlo para analizarlo:

<img width="724" height="75" alt="imagen" src="https://github.com/user-attachments/assets/1e1bd88c-6b93-434b-b34d-949451e3b9ab" />

That executable can be saved thanks to PEStudio. Inspecting him, there are a new resource inside, named XIA, which is a PKZIP compressed file:

Usando PEStudio para guardar este ejecutable y a su vez inspeccionándolo se puede ver que hay un archivo comprimido PKZIP dentro de él:

<img width="666" height="102" alt="imagen" src="https://github.com/user-attachments/assets/a92b9ebc-cf09-4978-8e06-d147342c4601" />

I extract XIA and change his extension to .7z to unzip it and check what is inside. However, it is password protected. Given that this compressed file was inside *tasksche.exe*, it is highly probable that the password to unzip it is contained within. The password is found while inspecting with Cutter:

Extraigo el archivo XIA y le cambio la extensión a .7z para descomprimirlo y comprobar su contenido. Sin embargo, está protegido con contraseña. Teniendo en cuenta que este archivo comprimido estaba dentro de *tasksche.exe*, es altamente probable que la contraseña para descomprimirlo esté dentro de él. Inspeccionando con Cutter figura la contraseña:

<img width="487" height="155" alt="imagen" src="https://github.com/user-attachments/assets/87b6ba0f-02e3-4a62-b29a-cf02cf2a87c2" />

The password is: **WNcry@2ol7**

La contraseña es: **WNcry@2ol7**

Something interesting can be seen in the same picture:

Algo interesante puede verse también en la imagen:

<img width="477" height="112" alt="imagen" src="https://github.com/user-attachments/assets/5c724bc1-6ace-4f20-856c-bbf834807ea6" />

These commands are posterior actions ran by the malware when the zip is extracted. Both are legit Windows commands, used here with evil purposes:

 1. `attrib +h .`

	- `attrib` it is a command from windows usable to change attributes of files or folders.
	- `+h` means "to add the hidden attribute"
	- `.` current directory

This command hides the current directory where the malware dropped the second stage, to avoid that the user could view the files or that malware components could be easily detectable.

 2. `icacls . /grant Everyone:F /T /C /Q`

	- `icacls` manages permissions in Windows
	- `.`  current directory
	- `/grant Everyone:F`  grants full control permissions to the group “Everyone” (all users)
	- `/T`  applies it recursively to all files and subfolders
	- `/C`  continues even with errors
	- `/Q`  quiet mode

It changes permissions of the entire current directory and his content in order to allow that any user (and process) to have full control and can act without restrictions.

Estos comandos son acciones posteriores que el malware ejecuta en el sistema una vez ya ha extraído el contenido. Ambos son comandos legítimos de Windows, usados aquí con fines maliciosos:

 1. `attrib +h .`

	- `attrib` es un comando de Windows que sirve para cambiar atributos de archivos o carpetas
	- `+h` significa “añadir el atributo oculto” (hidden)
	- `.` se refiere al directorio actual

Este comando oculta la carpeta actual donde se ha descomprimido el contenido del malware, para evitar que el usuario pueda ver los archivos o que los componentes del malware sean fácilmente detectables.

 2. `icacls . /grant Everyone:F /T /C /Q`

	- `icacls` gestiona permisos en Windows
	- `.`  directorio actual
	- `/grant Everyone:F`  concede permisos de control total (“Full”) al grupo “Everyone” (todos los usuarios)
	- `/T`  aplica recursivamente a todos los archivos y subcarpetas
	- `/C`  continúa aunque haya errores
	- `/Q`  modo silencioso

Cambia los permisos de toda la carpeta y su contenido para que cualquier usuario (y proceso) tenga control total y pueda actuar sin restricciones.

Inside the compressed file there is the following files:

Dentro del archivo comprimido tenemos los siguientes archivos:

<img width="631" height="246" alt="imagen" src="https://github.com/user-attachments/assets/f37d241c-74fc-46ef-8b56-e9012ea6c9df" />

Each file can be checked using the tool detect-it-easy and changing the file extension:
- **Folder msg**: contains .rtf files with the .wnry extension, which are explanatory notes, in different languages, with all the steps to follow in order to make the payment. Those notes are used by *Wana Decrypt0r 2.0*: 

Se puede ver qué es cada archivo usando la herramienta detect-it-easy y luego cambiando la extensión:
- **Carpeta msg**: contiene archivos .rtf con la extensión .wnry, los cuales tienen, en diferentes idiomas, una nota explicativa de la situación y de los pasos a seguir para realizar el pago. Estas notas serán usadas por el programa *Wana Decrypt0r 2.0*:

<img width="337" height="236" alt="imagen" src="https://github.com/user-attachments/assets/40cf334a-502e-4299-9d7d-ff31d880e26e" />

<img width="569" height="243" alt="imagen" src="https://github.com/user-attachments/assets/d58f3a54-8f35-46e8-904e-40bcb90ce42c" />

As an example, the message in russian.

Puede verse el mensaje en ruso, por ejemplo.

- **b.wnry**: wallpaper with instructions to the user.

- **b.wnry**: fondo de pantalla que queda tras la ejecución con instrucciones para el usuario.

<img width="779" height="534" alt="imagen" src="https://github.com/user-attachments/assets/a50111fc-7ce5-44e6-a1af-25a75545bc35" />

- **c.wnry**: list of .onion sites, maybe related with the payment or with command and control functions. There is also a link to download Tor browser, possibly in case that it is not installed in the system:

- **c.wnry**: lista de direcciones .onion, puede que para realizar el pago o bien para funciones de command and control. También hay un link para descargar el navegador Tor, probablemente en caso de que no estuviera presente en el sistema:

<img width="555" height="404" alt="imagen" src="https://github.com/user-attachments/assets/ba22a8df-985b-43c6-b506-67a9c720d1b2" />

<img width="671" height="172" alt="imagen" src="https://github.com/user-attachments/assets/ddd29c74-4401-4f5c-bee2-6e8f04c563e9" />

- **r.wnry**: text file with an explanatory message to the user which says that the user is a victim of a ransomware attack and must pay. 
- **s.wnry**: compressed folder with some .dll files related with Tor.

- **r.wnry**: archivo de texto con mensaje para el usuario explicándole que ha sido víctima de un ransomware y debe pagar.
- **s.wnry**: carpeta comprimida en la que figuran diferentes archivos .dll relacionados con Tor.

<img width="201" height="272" alt="imagen" src="https://github.com/user-attachments/assets/331b181e-5cb1-4b7f-b490-aedc3bc29649" />

- **t.wnry**: it is complicated to know the use of this file, because it does not have a common magic number, but the magic number `WANACRY!`:

- **t.wnry**: se hace complicado saber para qué se usa este archivo, ya que el magic number de este archivo no es de los comunes, sino `WANACRY!`:

<img width="645" height="265" alt="imagen" src="https://github.com/user-attachments/assets/fedf4293-011e-4843-83eb-ec360204a9e1" />

Trying to change the magic number to `MZ`, I do not observe any new information. Further investigation would be needed.

Probando a cambiar el magic number por `MZ`, no se observa nueva información. No tengo claro para qué sirve este archivo, haría falta investigar más.

- **taskdl.exe**: this executable has the following suspicious imports:
	- **FindFirstFileW**
	- **FindNextFileW**
	- **DeleteFileW**
The first two are used to find in directory and the third to delete files, so it is reasonable to think that the use of this executable is the deletion of files, possibly the user's files after their encryption.

- **taskdl.exe**: este ejecutable cuenta con las siguientes imports sospechosas:
	- **FindFirstFileW**
	- **FindNextFileW**
	- **DeleteFileW**
Las dos primeras se usan para buscar en directorio y la tercera para el borrado de archivos, por lo cual es lícito pensar que el uso de este ejecutable es el de borrar archivos, posiblemente los archivos del usuario tras su encriptación.

- **taskse.exe**: analyzing this executable with PEStudio, or checking his strings, nothing suspicious can be seen. Thanks to dynamic analysis I do realised about his role regarding the other files. May having more functions, but it is sure that is related with `@WanaDecryptor@.exe`, which shows a window at the end of the encryption. If this window is closed, the active process `tasksche.exe` runs `taskse.exe`, which runs again `@WanaDecryptor@.exe`. This happens every 30 seconds approximately and turns to be something really annoying to the user, unless the main process `@WanaDecryptor@.exe` and the process `tasksche.exe`, are closed.

- **taskse.exe**: analizando este ejecutable en PEstudio o bien mirando sus strings, no se aprecia nada sospechoso. Mediante el análisis dinámico sí he podido figurarme cómo encaja en el gran esquema de las cosas, y pudiendo tener más funciones, se ve que está relacionado con el programa `@WanaDecryptor@.exe`. Este programa muestra una ventana al término de la ejecución del malware. Si se cierra esta ventana, el proceso activo `tasksche.exe` ejecuta  `taskse.exe` y este a su vez vuelve a ejecutar `@WanaDecryptor@.exe`. Esto ocurre aproximadamente cada 30 segundos, convirtiéndose en algo bastante molesto, a menos que se cierre el proceso principal `@WanaDecryptor@.exe` y el proceso  `tasksche.exe`, que es quien llama a `taskse.exe` cada vez. 

<img width="244" height="79" alt="imagen" src="https://github.com/user-attachments/assets/9bcba2c4-7fb3-46eb-b9bf-3367d0ac6ba9" /><br/>

<img width="245" height="97" alt="imagen" src="https://github.com/user-attachments/assets/d14c8157-bf6e-425b-91c0-a08149118f88" />

Considering that `taskse.exe` runs only when is needed to run `@WanaDecryptor@.exe` again and closes after that, I would say that `tasksche.exe` checks running processes and executes `@WanaDecryptor@.exe` if `taskse.exe` is not running. But this is just a guess.

Teniendo en cuenta que `taskse.exe` se abre sólo cuando hace falta invocar a `@WanaDecryptor@.exe` de nuevo y luego se cierra, diría que `tasksche.exe` monitoriza la lista de procesos activos, y si no figura `@WanaDecryptor@.exe`, es cuando ejecuta `taskse.exe`. Pero esto es sólo una suposición por mi parte.

- **u.wnry**: executable of **@WanaDecryptor@.exe**:

- **u.wnry**: ejecutable de **@WanaDecryptor@.exe**:

<img width="811" height="614" alt="imagen" src="https://github.com/user-attachments/assets/cd9a7c09-e309-4ab1-8881-1d62411cacf4" />


# Regla YARA


According to the gathered indicators in the whole analysis, a YARA rule for this malware can be written. However, I created this rule only with the first phase of the malware in mind, and that is why not all the indicators are valid, because some of them are into the compressed file. For example, .onion urls will never be triggered with this sample and will not be included.

Basándome en la mayoría de indicadores recopilados durante el análisis, se puede escribir una regla YARA para este malware. Sin embargo, esta regla la he creado sólo con la primera fase del malware en mente, y por ello no todos los indicadores valen, ya que algunos de ellos se encuentran en el archivo comprimido. Por ejemplo, las url .onion nunca van a saltar con esta muestra y por lo tanto no las incluyo. 

```
rule YaraCry {
    
    meta: 
        last_updated = "2026"
        author = "Me"
        description = "Rule YARA WannaCry"

    strings:
        // Fill out identifying strings and other criteria
        $string1 = ".wnry"                  ascii
        $string2 = "tasksche.exe"           ascii
        $string3 = "WNcry@2ol7"             ascii
        $string4 = "taskdl.exe"             ascii
        $string5 = "taskse.exe"             ascii

        $url = "iuqerfsodp9ifjaposdfjhgosurijfaewrwergwea.com" ascii

        $magic_number = "MZ"                ascii


    condition:
        // Fill out the conditions that must be met to identify the binary
        $magic_number at 0 and ($url or 1 of ($string*))
```



# Splunk

Since this is not a formal analysis but a course conclusion, I can allow myself to explore other related approaches. Splunk is one of the leading SIEM tools of the market, and since it allows to collect, analyze and correlate network and systems data in real time, it is interesting to observe the obtained results when the sample is run in the lab. I analyzed system telemetry using the app "Sysmon app for Splunk" and the logs generated by Sysmon.

Unfortunately, not all the known data, such as the creation of the encrypted files or other events, is collected. That might be due to data encryption, which can prevent logs from being sent, or to a lack of resources allocated to the Splunk server VM, which could cause problems with data ingestion, at least in my experience. However, it is a useful way to list relevant events in a preliminary way.

Ya que esto no es un análisis formal sino la conclusión de un curso, puedo permitirme explorar otros caminos relacionados. Splunk es una de las principales herramientas SIEM del mercado, y ya que permite recopilar, analizar y correlacionar datos de red y sistemas en tiempo real, resulta interesante ver qué resultados nos ofrece al ejecutar la muestra en el laboratorio. He analizado la telemetría del equipo mediante la app "Sysmon app for Splunk" y los logs generados por Sysmon. 

Desafortunadamente, no se recopilan datos de todos los eventos conocidos, como la creación de los archivos encriptados u otros. Esto puede deberse a la encriptación de los datos, lo cual impide que sean enviados, o a la falta de recursos asignados a la VM del servidor Splunk, lo que puede provocar problemas de ingesta de datos, al menos en mi experiencia. Sin embargo, sigue siendo una forma útil de listar eventos automáticamente de forma preliminar.


## Sysmon app for Splunk

To use this app, is needed to install sysmon in the target VM (I used the **SwiftOnSecurity** configuration file template) and send the generated log with the relevant events to Splunk through a lightweight agent, known as a forwarder. Once the data has been sent, it can be viewed on different dashboards.

Para usarla, hay que instalar sysmon en la VM objetivo (he usado el archivo de configuración de **SwiftOnSecurity**) y mandar el log que genera con los eventos de interés a Splunk mediante un agente ligero llamado forwarder. Una vez enviados los datos, se podrán visualizar en diferentes dashboards. 

<img width="1630" height="578" alt="imagen" src="https://github.com/user-attachments/assets/8453fa2a-83c5-4809-bef1-aa0eef98e518" />

Also, thanks to those dashboards, specific events can be viewed, like, for example, the DNS call that it works as the killswitch:

También en base a esos dashboard se puede acceder directamente a eventos concretos, como por ejemplo la llamada DNS que constituye el killswitch:

<img width="493" height="331" alt="imagen" src="https://github.com/user-attachments/assets/87e23f1e-6a74-44ed-bae3-0d86b1d836e4" />

Or even some other behaviours that I did not realized about can be seen, like the modification by the second stage of the malware of the creation date of the files `taskse.exe`, `tasksdl.exe` and  `tor.exe`. In the picture is shown the modification of `taskse.exe`:

O incluso se pueden ver otros comportamientos no percibidos hasta ahora en el análisis, como la alteración de la fecha de creación de los archivos  `taskse.exe`, `tasksdl.exe` y  `tor.exe` por parte de la segunda fase del malware,  `tasksche.exe`. En la foto, la modificación de `taskse.exe`:

<img width="494" height="466" alt="imagen" src="https://github.com/user-attachments/assets/964d4242-56ac-4e54-90fb-fce1deba6d6e" />

Another interesting section is about register-level operations. Here I found the register created to run *tasksche.exe* at boot, which I already talked about in the host-based indicators section, and a new service:

Otra sección que puede verse es la de operaciones a nivel de registro. Aquí encuentro el registro creado para ejecutar *tasksche.exe* al inicio, registro ya comentado en la parte de indicadores basados en host, y otro servicio nuevo:

<img width="489" height="140" alt="imagen" src="https://github.com/user-attachments/assets/cb2f7b9e-a253-4307-9457-2e5ec27ed920" />

I did not previously detected the service *mssecsvc2.0*, it runs the first stage of WannaCry at the boot of the system:

El servicio **mssecsvc2.0** no lo había detectado previamente, y se encarga de ejecutar la primera fase del malware en el inicio del sistema:

<img width="550" height="215" alt="imagen" src="https://github.com/user-attachments/assets/d2a6fa60-94f9-407a-95d5-bc71a9655949" />

Start means the kind of service startup, while the value 2 means that the service will initiate at Window's boot automatically.

Start indica el tipo de arranque del servicio, mientras que el valor 2 indica que el servicio se iniciará automáticamente al arrancar Windows.

<img width="583" height="305" alt="imagen" src="https://github.com/user-attachments/assets/0933b160-849f-4bd8-b357-1099a0f25250" />

As can be seen, it runs the original file of the malware at boot, which means that this is another persistence mechanism.

Como se ve, ejecuta al inicio el archivo original del malware, por lo que este servicio es otro mecanismo más de persistencia.
