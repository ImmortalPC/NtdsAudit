NtdsAudit is an application to assist in auditing Active Directory databases.

It provides some useful statistics relating to accounts and passwords, as shown in the following example. It can also be used to dump password hashes for later cracking. 

```
Account stats for: domain.local
  Disabled users _____________________________________________________   418 of  5164 (8%)
  Expired users ______________________________________________________    67 of  5164 (1%)
  Active users unused in 1 year ______________________________________   787 of  4679 (17%)
  Active users unused in 90 days _____________________________________  1240 of  4679 (27%)
  Active users which do not require a password _______________________   156 of  4679 (3%)
  Active users with non-expiring passwords ___________________________  3907 of  4679 (84%)
  Active users with password unchanged in 1 year _____________________  1006 of  4679 (22%)
  Active users with password unchanged in 90 days ____________________  1400 of  4679 (30%)
  Active users with Administrator rights _____________________________    63 of  4679 (1%)
  Active users with Domain Admin rights ______________________________    54 of  4679 (1%)
  Active users with Enterprise Admin rights __________________________     0 of  4679 (0%)

  Disabled computer accounts _________________________________________    86 of  1414 (6%)

Password stats for: domain.local
  Active users using LM hashing ______________________________________    40 of  4679 (1%)
  Active users with duplicate passwords ______________________________  2312 of  4679 (49%)
  Active users with password stored using reversible encryption ______  4666 of  4679 (100%)
```

### Usage
NtdsAudit requires version 4.6 or newer of the .NET framework.

```
Usage:  [arguments] [options]

Arguments:
  NTDS file  The path of the NTDS.dit database to be audited, required.

Options:
  -v | --version            Show version information
  -h | --help               Show help information
  -s | --system <file>      The path of the associated SYSTEM hive, required when using the pwdump option.
  -p | --pwdump <file>      The path to output hashes in pwdump format.
  -u | --users-csv <file>   The path to output user details in CSV format.
  --history-hashes          Include history hashes in the pdwump output.
  --dump-reversible <file>  The path to output clear text passwords, if reversible encryption is enabled.
  --wordlist                The path to a wordlist of weak passwords for basic hash cracking. Warning, using this option is slow, the use of a dedicated password cracker, such as 'john', is recommended instead.
  --base-date <yyyyMMdd>    Specifies a custom date to be used as the base date in statistics. The last modified date of the NTDS file is used by default.
  --debug                   Show debug output.
  --potfile <hashcat.pot>   Pot file of hashcat

WARNING: Use of the --pwdump option will result in decryption of password hashes using the System Key.
Sensitive information will be stored in memory and on disk. Ensure the pwdump file is handled appropriately
```

For example, the following command will display statistics, output a file `pwdump.txt` containing password hashes, and output a file `users.csv` containing details for each user account.

```
ntdsaudit ntds.dit -s SYSTEM -p pwdump.txt -u users.csv
```

### Obtaining the required files
NtdsAudit requires the `ntds.dit` Active Directory database, and optionally the `SYSTEM` registry hive if dumping password hashes. These files are locked by a domain controller and as such cannot be simply copy and pasted. The recommended method of obtaining these files from a domain controller is using the builtin `ntdsutil` utility. 

* Open a command prompt (cmd.exe) as an administrator. To open a command prompt as an administrator, click Start. In Start Search, type Command Prompt. At the top of the Start menu, right-click Command Prompt, and then click Run as administrator. If the User Account Control dialog box appears, enter the appropriate credentials (if requested) and confirm that the action it displays is what you want, and then click Continue.

* At the command prompt, type the following command, and then press ENTER:

```
C:\> ntdsutil "activate instance ntds" "ifm" "create full <Drive>:\<Folder>" quit quit

```

Where `<Drive>:\<Folder>` is the path to the folder where you want the files to be created. 

### Example of output for --users-csv
```
C:\myDump\Active Directory> NtdsAudit.exe --system ..\registry\SYSTEM --users-csv Users.csv --potfile hashcat.potfile ntds.dit

Sid,Domain,Username,Administrator,Domain Admin,Enterprise Admin,Disabled,Expired,Password Never Expires,Password Not Required,Password Last Changed,Last Logon,Hash,Password,PasswordLen,MemberOf,Count in group
S-1-5-21-2111111111-1111111111-311111111-500,FAKEDOM,Administrateur,True,True,True,False,False,True,False,18/07/2014 09:06:51,19/06/2018 08:02:57,AAD3B435B51404EEAAD3B435B51404EE:BB3EFEE1438C71BD2425AB010B8B568E,"<NOT FOUND>",-1,"Propriétaires créateurs de la stratégie de groupe / Opérateurs de sauvegarde / Admins du domaine / Utilisateurs du Bureau à distance / Administrateurs de l'entreprise / Administrateurs / Administrateurs du schéma", 21
S-1-5-21-2111111111-1111111111-311111111-502,FAKEDOM,krbtgt,False,False,False,True,False,False,False,12/12/2012 21:37:45,01/01/1601 00:00:00,AAD3B435B51404EEAAD3B435B51404EE:051C4E7EFD2446DDE6757D6236973864,"<NOT FOUND>",-1,"",0
S-1-5-21-2111111111-1111111111-311111111-501,FAKEDOM,Invité,False,False,False,True,False,True,True,01/01/1601 00:00:00,01/01/1601 00:00:00,AAD3B435B51404EEAAD3B435B51404EE:31D6CFE0D16AE931B73C59D7E0C089C0,"<EMPTY>",0,"Invités",1
S-1-5-21-2111111111-1111111111-311111111-1148,FAKEDOM,aze,False,False,False,False,False,False,False,31/05/2018 07:23:51,14/06/2018 16:16:19,AAD3B435B51404EEAAD3B435B51404EE:C9F2BA0BA6D8590F6787697FA0222098,"<NOT FOUND>",-1,"Accès compatible pré-Windows 2000",1
S-1-5-21-2111111111-1111111111-311111111-2172,FAKEDOM,rty,False,False,False,False,False,False,False,25/04/2018 08:57:24,15/06/2018 06:50:10,AAD3B435B51404EEAAD3B435B51404EE:4AABCD9051E4A0012AA388BFB7CE09BB,"<NOT FOUND>",-1,"Accès compatible pré-Windows 2000",1
S-1-5-21-2111111111-1111111111-311111111-2263,FAKEDOM,uio,False,False,False,False,False,False,False,25/04/2018 10:04:17,20/06/2018 08:00:55,AAD3B435B51404EEAAD3B435B51404EE:4737AF73FB0BB5CA76CCB90071011466,"<NOT FOUND>",-1,"Accès compatible pré-Windows 2000",1
S-1-5-21-2111111111-1111111111-311111111-2315,FAKEDOM,qsd,False,False,False,False,False,False,False,16/04/2018 07:57:14,12/06/2018 18:33:00,AAD3B435B51404EEAAD3B435B51404EE:1E17FD53A0B1B6D3D8B7AC5C091175C2,"<NOT FOUND>",-1,"Accès compatible pré-Windows 2000",1
S-1-5-21-2111111111-1111111111-311111111-2367,FAKEDOM,fgh,False,False,False,False,False,False,False,18/04/2018 14:43:21,20/06/2018 10:31:34,AAD3B435B51404EEAAD3B435B51404EE:AAD56EB25D13370ED9D002D5C4AC6548,"jay112002*#",11,"Accès compatible pré-Windows 2000",1
S-1-5-21-2111111111-1111111111-311111111-2416,FAKEDOM,jkl,False,False,False,False,False,False,False,09/05/2018 08:39:36,18/06/2018 15:54:53,AAD3B435B51404EEAAD3B435B51404EE:DE55C4012FDEF8CC086CF2D712A9583F,"<NOT FOUND>",-1,"Accès compatible pré-Windows 2000",1
S-1-5-21-2111111111-1111111111-311111111-6244,FAKEDOM,wxc,False,False,False,False,False,False,False,16/04/2018 07:03:07,14/06/2018 10:04:09,AAD3B435B51404EEAAD3B435B51404EE:828B8CE8212D38ACA528CEE4E79244B3,"Theo0910",8,"Accès compatible pré-Windows 2000",1
S-1-5-21-2111111111-1111111111-311111111-5342,FAKEDOM,vbn,False,False,False,False,False,False,False,26/04/2018 09:15:20,19/06/2018 09:50:17,AAD3B435B51404EEAAD3B435B51404EE:869BBDA1846DCA3A4D10578611D83EEF,"<NOT FOUND>",-1,"Accès compatible pré-Windows 2000",1
S-1-5-21-2111111111-1111111111-311111111-8390,FAKEDOM,svc-aze,False,False,False,False,False,True,False,22/03/2013 12:07:19,01/06/2015 06:46:41,AAD3B435B51404EEAAD3B435B51404EE:44E2CE30BDBC3193869CDB9BEE346079,"<NOT FOUND>",-1,"Propriétaires créateurs de la stratégie de groupe",1
S-1-5-21-2111111111-1111111111-311111111-8420,FAKEDOM,svc-qsd,False,False,False,True,False,True,False,04/07/2012 13:11:03,20/10/2012 02:01:01,9FF187C18A533C9D1BBCD60EF1A63E08:B4C176BBD9FC79B8177F520016987DDD,"<NOT FOUND>",-1,"Opérateurs de serveur",1
```

Second file
```
"Number of account",1427
"Active account (Enabled and password not expired)",1027
"Disabled users","  398 of  1427 (27,9%)"
"Expired users","    2 of  1427 (0,1%)"
"Active users unused in 1 year","  145 of  1027 (14,1%)"
"Active users unused in 90 days","  210 of  1027 (20,4%)"
"Active users which do not require a password","   19 of  1027 (1,9%)"
"Active users with non-expiring passwords","  685 of  1027 (66,7%)"
"Active users with password unchanged in 1 year","  240 of  1027 (23,4%)"
"Active users with password unchanged in 90 days","  351 of  1027 (34,2%)"
"Active users with Administrator rights (domain=xxx)","   18 of  1027 (1,8%)"
"Active users with Domain Admin rights (domain=xxx)","   17 of  1027 (1,7%)"
"Active users with Enterprise Admin rights (domain=xxx)","   16 of  1027 (1,6%)"
"Number of compromised accounts","870 (60%)"
"Number of compromised accounts with enabled flag","533 (37%)"
"Number of compromised accounts with admin priviledge","5 (0%)"
"Number of compromised accounts, enabled and admin","4 (0%)"

"Passwword with a length of 0",1
"Passwword with a length of 1",0
"Passwword with a length of 2",0
"Passwword with a length of 3",0
"Passwword with a length of 4",0
"Passwword with a length of 5",0
"Passwword with a length of 6",0
"Passwword with a length of 7",0
"Passwword with a length of 8",85
"Passwword with a length of 9",107
"Passwword with a length of 10",108
"Passwword with a length of 11",51
"Passwword with a length of 12",38
"Passwword with a length of 13",28
"Passwword with a length of 14",6
"Passwword with a length of 15",1
"Passwword with a length of 16",0
"Passwword with a length of 17",0
"Passwword with a length of 18",0
"Passwword with a length of 19",0
"Passwword with a length of 20",0
"Passwword with a length of 21",0
"Passwword with a length of 22",0
"Passwword with a length of 23",0
"Passwword with a length of 24",0
"Passwword with a length of 25",0
"Passwword with a length of 26",0
"Passwword with a length of 27",0
"Passwword with a length of 28",0
"Passwword with a length of 29",0
"Passwword with a length of 30",0
```

Third file
```
Hash,Password,Count
"31d6cfe0d16ae931b73c59d7e0c089c0","<EMPTY>",223
"e19ccf75ee54e06b06a5907af13cef42","P@ssw0rd",49
"7100a909c7ff05b266af3c42ec058c33","Password01",34
"48c2bcb542e7cca4aac292e74f76bc3a","password2018$",28
"632b06c75304ab6d074b8ed8b73cf98b","Password02",24
"0d244685030cbfacfc47a1cd4119194f","P@ssword.2018",17
"dc92d04be474bbed6eb2b896acd7ee35","Password03",12
"ac339e0625d04fc85f36dbc7ccdc137f","Juillet2018",11
"a394b1e1a66ef17e506d6466773d2a1f","Hotline01",9
"da41b016caab2338d5a164bb6c705878","P@ssword.2017",8
"7cf9529950c6cab673a1f1fd2f17f6e2","Hotline123",5
"0c90dd3cfc7d2b2743d166cb80e8d339","Password05",4
"645d3cfe689183f6a1fde2525aff92df","Septembre2018",4
"4424147a7dcd3c47c4ec3921443023bd","P@ssword01",4
"192b3172fd78a6fc93fb0635a92f0650","Aout2018",3
"f69c364a819981ed1e3ff4808a769c7a","Juillet18",3
"1ace506c10a06249cd98858946036f9b","P@ssword.2015",3
"c0c02a9c8c0edb1101cbfab74ab2e4e2","Password2018",3
"a65e89a846c4f2d9d4618f8cbe882185","Password0123",3
"6f658e373f9137895e2866bfe61e2de7","<NOT FOUND>",2
"d4ae376a9a90b36d3358d4fb9098a8fe","<NOT FOUND>",2
"46629b79aceb86b0422b967eef98aea2","<NOT FOUND>",2
"39573455576d86d5d749d8008b4d9d99","Shauny201809",2
"71b9b05a0e41a7dd2cf8c13a23cc7de1","<NOT FOUND>",2
"f1408bb1e97d29b0b5c87d4ce7d7cebd","Manchester_11",2
"76ece2ca222c55e6499280e59184a141","<NOT FOUND>",2
"cabf4b457486465187c76e7c24461050","Soleil07",2
"06a6066abf57fd751781aae33cd99be5","Password08",2
"98e4efd691345539852c6c3a7161ce2c","Sept2018",2
"12f73b3b10b9171adcc0ea79f8b26d8c","Password10",2
"15214b8d8394af359fc1dcce7ab991f1","Aout082018",2
"3b9712cd7f15ce7dbed29d11095cab37","<NOT FOUND>",2
...
```
