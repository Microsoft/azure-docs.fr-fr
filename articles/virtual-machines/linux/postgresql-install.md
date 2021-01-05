---
title: Configurer PostgreSQL sur une machine virtuelle Linux
description: Découvrez comment installer et configurer PostgreSQL sur une machine virtuelle Linux dans Azure.
author: cynthn
ms.service: virtual-machines-linux
ms.subservice: workloads
ms.topic: how-to
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 02/01/2016
ms.author: cynthn
ms.openlocfilehash: fdde7613627c9fec0694f3985f78cf10e52f59c2
ms.sourcegitcommit: 4313e0d13714559d67d51770b2b9b92e4b0cc629
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/27/2020
ms.locfileid: "91397094"
---
# <a name="install-and-configure-postgresql-on-azure"></a>Installer et configurer PostgreSQL sur Microsoft Azure
PostgreSQL est une base de données open source avancée similaire à Oracle et DB2. Il inclut des fonctionnalités destinées aux entreprises, comme la conformité complète à ACID, un traitement transactionnel fiable et un contrôle d’accès concurrentiel multiversion. Il prend également en charge des normes comme ANSI SQL et SQL/MED (y compris les wrappers de données externes pour Oracle, MySQL, MongoDB et beaucoup d’autres). Il est hautement extensible, avec la prise en charge de 12 langages procéduraux, les index GIN et GIST, la prise en charge des données spatiales et plusieurs fonctionnalités de type NoSQL pour les applications JSON ou basées sur les paires clé-valeur.

Dans cet article, vous allez apprendre à installer et à configurer PostgreSQL sur une machine virtuelle Azure exécutant Linux.


## <a name="install-postgresql"></a>Installation de PostgreSQL
> [!NOTE]
> Vous devez déjà disposer d’une machine virtuelle Microsoft Azure exécutant Linux pour effectuer ce didacticiel. Consultez le [didacticiel sur les machines virtuelles Linux Azure](quick-create-cli.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)pour créer et configurer une machine virtuelle Linux avant de continuer.
> 
> 

Dans ce cas, utilisez le port 1999 comme port PostgreSQL.  

Connectez-vous à la machine virtuelle Linux que vous avez créée via PuTTY. S’il s’agit de la première fois que vous utilisez une machine virtuelle Azure Linux, consultez la page [Utilisation de SSH avec Linux sur Azure](mac-create-ssh-keys.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) pour apprendre comment utiliser PuTTY pour vous connecter à une machine virtuelle Linux.

1. Exécutez la commande suivante pour passer à l’utilisateur root (admin) :

    ```console
    # sudo su -
    ```

2. Certaines distributions ont des dépendances à installer avant d’installer PostgreSQL. Recherchez votre distribution dans cette liste et exécutez la commande appropriée :
   
   * Red Hat Linux :

        ```console
        # yum install readline-devel gcc make zlib-devel openssl openssl-devel libxml2-devel pam-devel pam  libxslt-devel tcl-devel python-devel -y
        ```

   * Debian Linux :

        ```console
        # apt-get install readline-devel gcc make zlib-devel openssl openssl-devel libxml2-devel pam-devel pam libxslt-devel tcl-devel python-devel -y
        ```

   * SUSE Linux :

        ```console
        # zypper install readline-devel gcc make zlib-devel openssl openssl-devel libxml2-devel pam-devel pam  libxslt-devel tcl-devel python-devel -y
        ```

3. Téléchargez PostgreSQL dans le répertoire racine, puis décompressez le package :

    ```console
    # wget https://ftp.postgresql.org/pub/source/v9.3.5/postgresql-9.3.5.tar.bz2 -P /root/

    # tar jxvf  postgresql-9.3.5.tar.bz2
    ```

    Voici un exemple ci-dessus. Vous pouvez trouver l’adresse de téléchargement détaillée dans l’ [Index de /pub/source/](https://ftp.postgresql.org/pub/source/).

4. Pour démarrer la génération, exécutez ces commandes :

    ```console
    # cd postgresql-9.3.5

    # ./configure --prefix=/opt/postgresql-9.3.5
    ```

5. Si vous voulez créer tout ce qui peut l’être, notamment la documentation (HTML et les pages man) et les modules complémentaires (`contrib`), exécutez à la place la commande suivante :

    ```console
    # gmake install-world
    ```

    Vous recevez normalement le message de confirmation suivant :

    ```output
    PostgreSQL, contrib, and documentation successfully made. Ready to install.
    ```

## <a name="configure-postgresql"></a>Configurer PostgreSQL
1. (Facultatif) Créez un lien symbolique pour raccourcir la référence PostgreSQL en n’y incluant pas le numéro de version :

    ```console
    # ln -s /opt/postgresql-9.3.5 /opt/pgsql
    ```

2. Créez un répertoire pour la base de données :

    ```console
    # mkdir -p /opt/pgsql_data
    ```

3. Créez un utilisateur non-root et modifiez le profil de cet utilisateur. Passez ensuite à ce nouvel utilisateur (appelé *postgres* dans notre exemple) :

    ```console
    # useradd postgres
   
    # chown -R postgres.postgres /opt/pgsql_data
   
    # su - postgres
    ```
   
   > [!NOTE]
   > Pour des raisons de sécurité, PostgreSQL utilise un utilisateur non-root pour initialiser, démarrer ou arrêter la base de données.
   > 
   > 
4. Modifiez le fichier *bash_profile* en entrant les commandes ci-dessous. Ces lignes seront ajoutées à la fin du fichier *bash_profile* :

    ```config
    cat >> ~/.bash_profile <<EOF
    export PGPORT=1999
    export PGDATA=/opt/pgsql_data
    export LANG=en_US.utf8
    export PGHOME=/opt/pgsql
    export PATH=\$PATH:\$PGHOME/bin
    export MANPATH=\$MANPATH:\$PGHOME/share/man
    export DATA=`date +"%Y%m%d%H%M"`
    export PGUSER=postgres
    alias rm='rm -i'
    alias ll='ls -lh'
    EOF
    ```

5. Exécutez le fichier *bash_profile* :

    ```console
    $ source .bash_profile
    ```

6. Vérifiez votre installation avec la commande suivante :

    ```console
    $ which psql
    ```

    Si votre installation est réussie, vous voyez la réponse suivante :

    ```output
    /opt/pgsql/bin/psql
    ```

7. Vous pouvez également vérifier la version de PostgreSQL :

    ```sql
    $ psql -V
    ```

8. Initialisez la base de données :

    ```console
    $ initdb -D $PGDATA -E UTF8 --locale=C -U postgres -W
    ```

    Les résultats suivants doivent normalement s’afficher :

![Capture d’écran de la sortie après l’initialisation de la base de données](./media/postgresql-install/no1.png)

## <a name="set-up-postgresql"></a>Configurer PostgreSQL
<!--    [postgres@ test ~]$ exit -->

Exécutez les commandes suivantes :

```console
# cd /root/postgresql-9.3.5/contrib/start-scripts

# cp linux /etc/init.d/postgresql
```

Modifiez deux variables dans le fichier /etc/init.d/postgresql. Le préfixe est défini sur le chemin d’installation de PostgreSQL : **/opt/pgsql**. PGDATA est défini sur le chemin de stockage des données de PostgreSQL : **/opt/pgsql_data**.

```config
# sed -i '32s#usr/local#opt#' /etc/init.d/postgresql

# sed -i '35s#usr/local/pgsql/data#opt/pgsql_data#' /etc/init.d/postgresql
```

![Capture d’écran du préfixe d’installation et du répertoire de données](./media/postgresql-install/no2.png)

Changez le fichier pour le rendre exécutable :

```console
# chmod +x /etc/init.d/postgresql
```

Démarrez PostgreSQL :

```console
# /etc/init.d/postgresql start
```

Vérifiez si le point de terminaison de PostgreSQL est activé :

```console
# netstat -tunlp|grep 1999
```

Vous devez normalement voir la sortie suivante.

![Capture d’écran du point de terminaison de PostgreSQL activé](./media/postgresql-install/no3.png)

## <a name="connect-to-the-postgres-database"></a>Connectez-vous à la base de données Postgres
Repassez à l’utilisateur postgres :

```console
# su - postgres
```

Créez une base de données Postgres :

```console
$ createdb events
```

Connectez-vous à la base de données d’événements que vous venez de créer :

```console
$ psql -d events
```

## <a name="create-and-delete-a-postgres-table"></a>Créer et supprimer une table Postgres
Maintenant que vous vous êtes connectés à la base de données, vous pouvez y créer des tables.

Par exemple, créez un exemple de table Postgres avec la commande suivante :

```sql
CREATE TABLE potluck (name VARCHAR(20),    food VARCHAR(30),    confirmed CHAR(1), signup_date DATE);
```

Vous avez maintenant configuré une table de quatre colonnes avec ces noms de colonnes et ces restrictions :

1. La colonne « name » a été limitée par la commande VARCHAR à une longueur de 20 caractères.
2. La colonne « food » indique l’aliment que chaque personne emporte. VARCHAR limite ce texte à moins de 30 caractères.
3. La colonne « confirmed » indique si la personne a répondu à l’invitation au repas. Les valeurs acceptables sont « Y » et « N ».
4. La colonne « date » affiche la date à laquelle la personne s’est inscrite. Postgres requiert que les dates soient écrites au format aaaa-mm-jj.

Voici ce que vous devez voir si votre table a été créée correctement :

![Capture d’écran du message qui s’affiche en cas de création réussie de la table](./media/postgresql-install/no4.png)

Vous pouvez également vérifier la structure de la table avec la commande suivante :

![Capture d’écran de la commande permettant de vérifier la structure de la table](./media/postgresql-install/no5.png)

### <a name="add-data-to-a-table"></a>Ajouter des données à une table
Insérez d’abord des informations dans une ligne :

```sql
INSERT INTO potluck (name, food, confirmed, signup_date) VALUES('John', 'Casserole', 'Y', '2012-04-11');
```

Cette sortie doit s’afficher :

![Capture d’écran des informations de ligne ajoutées](./media/postgresql-install/no6.png)

Vous pouvez aussi ajouter quelques personnes de plus à la table. Voici quelques propositions, mais vous pouvez créer vos propres informations :

```sql
INSERT INTO potluck (name, food, confirmed, signup_date) VALUES('Sandy', 'Key Lime Tarts', 'N', '2012-04-14');

INSERT INTO potluck (name, food, confirmed, signup_date) VALUES ('Tom', 'BBQ','Y', '2012-04-18');

INSERT INTO potluck (name, food, confirmed, signup_date) VALUES('Tina', 'Salad', 'Y', '2012-04-18');
```

### <a name="show-tables"></a>Afficher les tables
Utilisez la commande suivante pour afficher une table :

```sql
select * from potluck;
```

La sortie est la suivante :

![Capture d’écran de la sortie de la commande permettant d’afficher une table](./media/postgresql-install/no7.png)

### <a name="delete-data-in-a-table"></a>Supprimer des données dans une table
Utilisez les commandes suivantes pour supprimer des données dans une table :

```sql
delete from potluck where name=’John’;
```

Cela supprime toutes les informations dans la ligne « John ». La sortie est la suivante :

![image](./media/postgresql-install/no8.png)

### <a name="update-data-in-a-table"></a>Mettre à jour des données dans une table
Utilisez les commandes suivantes pour mettre à jour des données dans une table : Pour celle-ci, Sandy a confirmé qu’elle participait et nous allons donc changer sa réponse de « N » en « Y » :

```sql
UPDATE potluck set confirmed = 'Y' WHERE name = 'Sandy';
```

## <a name="get-more-information-about-postgresql"></a>Obtenez davantage d’informations sur PostgreSQL
Maintenant que vous avez terminé l’installation de PostgreSQL dans une machine virtuelle Azure Linux, vous pouvez ne profiter dans Microsoft Azure. Pour en savoir plus sur PostgreSQL, consultez le [site web PostgreSQL](https://www.postgresql.org/).
