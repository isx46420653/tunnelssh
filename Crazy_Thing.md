# The Crazy Thing!

[Teoria](Teoria.md)

# Índex

- [Exercici 15 - Ldap-remot i phpldapadmin-local](#Exemple-15-Ldap-remot-i-phpldapadmin-local)
  - [Ldap-remot a Amazon](#Desplegar-el-servei-ldap)
  - [Phpldapadmin local a l'aula o a casa](#Desplegar-el-servei-phpldapadmin)
- [Ejercicio16: Ldap-local i phpldapadmin-remot](#ejercicio16-ldap-local-i-phpldapadmin-remot)
  - [Engegar ldap i phpldapadmin i que tinguin connectivitat](#engegar-ldap-i-phpldapadmin-i-que-tinguin-connectivitat)

## Exemple-15 Ldap-remot i phpldapadmin-local

Desplegem dins d’un container Docker (host-remot) en una AMI (host-destí) el servei ldap amb el firewall de la AMI només obrint el port 22. Localment al host de l’aula (host-local) desplegem un container amb phpldapadmin. Aquest container ha de poder accedir a les dades ldap. des del host de l’aula volem poder visualitzar el phpldapadmin.

### Desplegar el servei ldap

- en el host-remot AMI AWS EC2 engegar un container ldap sense fer map dels ports.

```bash
# Ens connectem
[Pau@portatil]$ ssh -i new_key.pem fedora@54.152.97.180

# Arranquem un contenidor
[root@ip-172-31-84-39 ~]$ docker run --rm --name ldapserver.edt.org -h ldapserver.edt.org --net mynet -d isx46420653/k19:ldapserver
```

-  en la ami cal obrir únicament el port 22

```bash
# Comprovem els ports oberts a la AMI, també podem comprovar els Security Groups de l'instància.
[root@ip-172-31-84-39 ~]$ nmap localhost
Not shown: 999 closed ports
PORT   STATE SERVICE
22/tcp open  ssh

Nmap done: 1 IP address (1 host up) scanned in 1.68 seconds
```

-  També cal configurar el /etc/hosts de la AMI per poder accedir al container ldap per nom de host (preferentment).

```bash
# Assignem la direcció del contenidor per poder accedir per nom quan creem el túnel
[root@ip-172-31-84-39 ~]$ echo "172.18.0.2 ldapserver.edt.org" >> /etc/hosts
[root@ip-172-31-84-39 ~]$ cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

172.18.0.2 ldapserver.edt.org
```

-  verificar que des del host de l’aula (host-local) podem fer consultes ldap.

```bash
# Creem el túnel directe al port 50000 que connecta amb el host destí d'Amazon,
# i aquest redirigeix el túnel a ldapserver.edt.org al port 389.
[Pau@portatil]$ ssh -i new_key.pem -L 50000:ldapserver.edt.org:389 fedora@54.152.97.180

# Comprovem que podem fer consultes

[Pau@portatil]$ ldapsearch -x -LLL -h localhost -p 50000 dn

dn: dc=edt,dc=org

dn: ou=maquines,dc=edt,dc=org

dn: ou=clients,dc=edt,dc=org

. . .
```

### Desplegar el servei phpldapadmin

-  engegar en el host de l’aula (host-local) un container docker amb el servei phpldapadmin fent map del seu port 8080 al host-local (o no).

```bash
# Hem de canviar la configuració, per això el creem interactivament
# No cal mapejar el port però és més còmode i no costa res
[Pau@portatil]$ docker run --rm --name phpldapadmin -h phpldapadmin -p 8080:80 --net mynet -it isx46420653/phpldapadmin /bin/bash
```

- crear el túnel directe ssh des del host de l’aula (host-local) al servei ldap (host-remot) connectant via SSH al host AMI (host-destí).

```bash
# Obrim un túnel directe desde el host local cap a la xarxa mynet on està el contenidor LDAP,
# al port 50000 que redireccionarà al port 389 del host ldapserver.edt.org.
[Pau@portatil]$ ssh -i new_key.pem -L 172.18.0.1:50000:ldapserver.edt.org:389 fedora@54.152.97.180
```

-  configurar el phpldapadmin per que trobi la base de dades ldap accedint al host de l’aula al port acabat de crear amb el túnel directe ssh.

```bash
# El servidor passa a ser l'entrada del túnel que redirecciona al contenidor LDAP d'Amazon
[root@phpldapadmin phpldapadmin]$ vi config.php

    $servers->setValue('server','host','172.18.0.1');

    $servers->setValue('server','port',50000);

    $servers->setValue('server','base',array('dc=edt,dc=org'));

# Arranquem
[root@phpldapadmin phpldapadmin]$ bash startup.sh
```

-  Ara ja podem visualitzar des del host de l’aula el servei phpldapadmin, accedint al
  port 8080 del container phpldapadmin o al port que hem fet map del host de l’aula (si
  és que ho hem fet).

```bash
# Accedim amb firefox al port mapejat
[Pau@portatil]$ firefox localhost:8080/phpldapadmin

# En cas de no haver mapejat el port
# 172.19.0.2 és la direcció del contenidor php
[Pau@portatil]$ firefox 172.19.0.2/phpldapadmin/

# Fem login:
user:'cn=Manager,dc=edt,dc=org'
password:'secret'
```

### Ja tenim accés

![Imatge](./aux/phpldapadmin.png)
