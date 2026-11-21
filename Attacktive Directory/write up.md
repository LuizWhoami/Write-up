

# Enumeration active directory:
1. What tool will allow us to enumerate port 139/445?
![[Pasted image 20261120190738.png]]
![[Pasted image 20261120191029.png]]

2. What is the NetBIOS-Domain Name of the machine?
![[Pasted image 20261120191341.png]]


3. What invalid TLD do people commonly use for their Active Directory Domain?
![[Pasted image 20261120191449.png]]

# Enumeration kerberos:
1. What command within Kerbrute will allow us to enumerate valid usernames?
![[Pasted image 20261120191909.png]]

2. What notable account is discovered? (These should jump out at you)
![[Pasted image 20261120195437.png]]

3. What is the other notable account is discovered? (These should jump out at you)
![[Pasted image 20261120195437.png]]

# Abusing kerbrute
1. We have two user accounts that we could potentially query a ticket from. Which user account can you query a ticket from with no password?
![[Pasted image 20261120201536.png]]

2. Looking at the Hashcat Examples Wiki page, what type of Kerberos hash did we retrieve from the KDC? (Specify the full name)
https://hashcat.net/wiki/doku.php?id=example_hashes

3. What mode is the hash?
![[Pasted image 20261120202644.png]]

4. Now crack the hash with the modified password list provided, what is the user accounts password?
![[Pasted image 20261120202717.png]]


# Enumeration SMB

1. What utility can we use to map remote SMB shares?
Search

2. Which option will list shares?
```
man {tool}
```

3. How many remote shares is the server listing?
tente listar com a ferramenta que você encontrou

4. There is one particular share that we have access to that contains a text file. Which share is it?
encontre o compartilhamento na listagem que você fez

5. What is the content of the file?
baixe o conteudo com o comando get

6. Decoding the contents of the file, what is the full contents?
decodifique o conteudo

# Domain Privilege Escalation
Verification of privilege
![[Pasted image 20261120205248.png]]
1. What method allowed us to dump NTDS.DIT?
![[Pasted image 20261120211921.png]]
2. What is the Administrators NTLM hash?
mesmo print

3. What method of attack could allow us to authenticate as the user without the password?
Pass The hash

4. Using a tool called Evil-WinRM what option will allow us to use a hash?
![[Pasted image 20261120212128.png]]

# Flag Submission Panel

todos no desktop de cada usuario