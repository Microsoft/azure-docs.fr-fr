---
title: Configurer un groupe de disponibilité pour un groupe de travail indépendant du domaine
description: Apprenez à configurer un groupe de disponibilité AlwaysOn pour un groupe de travail indépendant du domaine Active Directory sur une machine virtuelle SQL Server dans Azure.
services: virtual-machines-windows
documentationcenter: na
author: MashaMSFT
editor: ''
tags: azure-service-management
ms.assetid: 53981f7e-8370-4979-b26a-93a5988d905f
ms.service: virtual-machines-sql
ms.subservice: hadr
ms.topic: how-to
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 01/29/2020
ms.author: mathoma
ms.openlocfilehash: 0f194101720481f71434709c467d0e3130a0f1f9
ms.sourcegitcommit: dfc4e6b57b2cb87dbcce5562945678e76d3ac7b6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/12/2020
ms.locfileid: "97359453"
---
# <a name="configure-a-workgroup-availability-group"></a>Configurer un groupe de disponibilité pour un groupe de travail 
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

Cet article décrit les étapes nécessaires à la création d'un cluster indépendant du domaine Active Directory avec un groupe de disponibilité Always On ; c'est ce que l'on appelle également un cluster de groupe de travail. Cet article se concentre sur les étapes de préparation et de configuration du groupe de travail et du groupe de disponibilité, sans s'attarder sur les étapes décrites dans d'autres articles, comme la création du cluster ou le déploiement du groupe de disponibilité. 


## <a name="prerequisites"></a>Prérequis

Afin de configurer un groupe de disponibilité pour un groupe de travail, vous devez disposer de ce qui suit :
- Au moins deux machines virtuelles Windows Server 2016 (ou version ultérieure) exécutant SQL Server 2016 (ou version ultérieure), déployées sur le même groupe à haute disponibilité, ou dans des zones de disponibilité différentes, et utilisant des adresses IP statiques 
- Un réseau local avec un minimum de 4 adresses IP gratuites sur le sous-réseau 
- Un compte sur chaque ordinateur du groupe d'administrateurs qui dispose également de droits d'administrateur système dans SQL Server 
- Ports ouverts : TCP 1433, TCP 5022, TCP 59999 

Pour référence, cet article utilise les paramètres suivants, mais ceux-ci peuvent être modifiés si nécessaire : 

| **Nom** | **Paramètre** |
| :------ | :---------------------------------- |
| **Nœud 1**   | AGNode1 (10.0.0.4) |
| **Nœud 2**   | AGNode2 (10.0.0.5) |
| **Nom du cluster** | AGWGAG (10.0.0.6) |
| **Port d'écoute** | AGListener (10.0.0.7) | 
| **Suffixe DNS** | ag.wgcluster.example.com | 
| **Nom du groupe de travail** | AGWorkgroup | 
| &nbsp; | &nbsp; |

## <a name="set-a-dns-suffix"></a>Définir un suffixe DNS 

Au cours de cette étape, vous allez configurer le suffixe DNS des deux serveurs. Par exemple : `ag.wgcluster.example.com`. Cela vous permettra d'utiliser le nom de l'objet auquel vous souhaitez vous connecter en tant qu'adresse complète sur votre réseau, par exemple `AGNode1.ag.wgcluster.example.com`. 

Pour configurer le suffixe DNS, procédez comme suit :

1. Connectez-vous à distance (via RDP) à votre premier nœud et ouvrez Gestionnaire de serveur. 
1. Sélectionnez **Serveur local**, puis sélectionnez le nom de votre machine virtuelle sous **Nom de l'ordinateur**. 
1. Sélectionnez **Modifier...** sous **Pour renommer cet ordinateur...** . 
1. Renommez le groupe de travail afin de lui donner un nom significatif, par exemple `AGWORKGROUP` : 

   ![Changer le nom du groupe de travail](./media/availability-group-clusterless-workgroup-configure/1-change-workgroup-name.png)

1. Sélectionnez **Autres...** pour ouvrir la boîte de dialogue **Nom d'ordinateur NetBIOS et suffixe DNS**. 
1. Entrez le nom de votre suffixe DNS sous **Suffixe DNS principal de cet ordinateur**, par exemple `ag.wgcluster.example.com`, puis sélectionnez **OK** : 

   ![Capture d’écran montrant le suffixe D N S et la boîte de dialogue Nom de l’ordinateur NetBIOS dans laquelle vous pouvez entrer la valeur.](./media/availability-group-clusterless-workgroup-configure/2-add-dns-suffix.png)

1. Vérifiez que le **nom complet de l'ordinateur** contient maintenant le suffixe DNS, puis sélectionnez **OK** pour enregistrer vos modifications : 

   ![Capture d’écran indiquant où trouver le nom complet de votre ordinateur.](./media/availability-group-clusterless-workgroup-configure/3-confirm-full-computer-name.png)

1. Lorsque vous y êtes invité, redémarrez le serveur. 
1. Répétez ces étapes sur tous les autres nœuds à utiliser pour le groupe de disponibilité. 

## <a name="edit-a-host-file"></a>Modifier un fichier hôte

En l'absence de répertoire actif, il n'y a aucun moyen d'authentifier les connexions Windows. Par conséquent, vous devez attribuer un niveau de confiance en modifiant le fichier hosts à l'aide d'un éditeur de texte. 

Pour modifier le fichier hosts, procédez comme suit :

1. Connectez-vous à distance (via RDP) à votre machine virtuelle. 
1. Utilisez l'**Explorateur de fichiers** pour accéder à `c:\windows\system32\drivers\etc`. 
1. Cliquez avec le bouton droit sur le fichier **hosts** et ouvrez-le dans le **Bloc-notes** (ou dans un autre éditeur de texte).
1. À la fin du fichier, ajoutez une entrée pour chaque nœud, ainsi que le groupe de disponibilité et l'écouteur sous la forme `IP Address, DNS Suffix #comment`, par exemple : 

   ```
   10.0.0.4 AGNode1.ag.wgcluster.example.com #Availability group node
   10.0.0.5 AGNode2.ag.wgcluster.example.com #Availability group node
   10.0.0.6 AGWGAG.ag.wgcluster.example.com #Cluster IP
   10.0.0.7 AGListener.ag.wgcluster.example.com #Listener IP
   ```
 
   ![Ajouter les entrées correspondant à l'adresse IP, au cluster et à l'écouteur dans le fichier hosts](./media/availability-group-clusterless-workgroup-configure/4-host-file.png)

## <a name="set-permissions"></a>Définir des autorisations

En l'absence d'Active Directory pour gérer les autorisations, vous devez autoriser manuellement un compte Administrateur local non intégré à créer le cluster. 

Pour ce faire, sur chaque nœud, exécutez la cmdlet PowerShell suivante dans une session PowerShell administrative : 

```PowerShell

new-itemproperty -path HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System -Name LocalAccountTokenFilterPolicy -Value 1
```

## <a name="create-the-failover-cluster"></a>Créer le cluster de basculement

Au cours de cette étape, vous allez créer le cluster de basculement. Pour en savoir plus sur ces étapes, suivez le [tutoriel consacré au cluster de basculement](failover-cluster-instance-storage-spaces-direct-manually-configure.md).

Différences notables entre le tutoriel et ce qui doit être fait pour un cluster de groupes de travail :
- Désactivez **Stockage** et **Espaces de stockage direct** lors de l'exécution de la validation du cluster. 
- Lors de l'ajout des nœuds au cluster, ajoutez le nom complet, par exemple :
   - `AGNode1.ag.wgcluster.example.com`
   - `AGNode2.ag.wgcluster.example.com`
- Décochez **Ajouter la totalité du stockage disponible au cluster**. 

Une fois le cluster créé, attribuez une adresse IP de cluster statique. Pour ce faire, procédez comme suit :

1. Sur un des nœuds, ouvrez **Gestionnaire du cluster de basculement**, sélectionnez le cluster, cliquez avec le bouton droit sur **Nom : \<ClusterNam>** sous **Principales ressources du cluster**, puis sélectionnez **Propriétés**. 

   ![Lancer les propriétés du nom de cluster](./media/availability-group-clusterless-workgroup-configure/5-launch-cluster-name-properties.png)

1. Sélectionnez l'adresse IP sous **Adresses IP**, puis choisissez **Modifier**. 
1. Sélectionnez **Utiliser une adresse statique**, fournissez l'adresse IP du cluster, puis sélectionnez **OK** : 

   ![Fournir une adresse IP statique pour le cluster](./media/availability-group-clusterless-workgroup-configure/6-provide-static-ip-for-cluster.png)

1. Vérifiez que vos paramètres sont corrects, puis sélectionnez **OK** pour les enregistrer :

   ![Vérifier les propriétés du cluster](./media/availability-group-clusterless-workgroup-configure/7-verify-cluster-properties.png)

## <a name="create-a-cloud-witness"></a>Créer un témoin cloud 

Au cours de cette étape, vous allez configurer un témoin de partage cloud. Pour en savoir plus sur ces étapes, consultez [Déployer un témoin cloud pour un cluster de basculement](/windows-server/failover-clustering/deploy-cloud-witness). 

## <a name="enable-the-availability-group-feature"></a>Activer la fonctionnalité de groupes de disponibilité 

Au cours de cette étape, vous allez activer la fonctionnalité Groupes de disponibilité. Pour en savoir plus sur ces étapes, suivez le [tutoriel consacré aux groupes de disponibilité](availability-group-manually-configure-tutorial.md#enable-availability-groups). 

## <a name="create-keys-and-certificates"></a>Créer des clés et des certificats

Au cours de cette étape, vous allez créer les certificats utilisés par la connexion SQL sur le point de terminaison chiffré. Créez un dossier sur chaque nœud pour y conserver les sauvegardes des certificats, par exemple `c:\certs`. 

Pour configurer le premier nœud, procédez comme suit : 

1. Ouvrez **SQL Server Management Studio**, puis connectez-vous à votre premier nœud, par exemple `AGNode1`. 
1. Ouvrez une fenêtre **Nouvelle requête** et exécutez l'instruction Transact-SQL (T-SQL) suivante après la mise à jour vers un mot de passe complexe et sécurisé :

   ```sql
   USE master;  
   CREATE MASTER KEY ENCRYPTION BY PASSWORD = 'PassWOrd123!';  
   GO

   --create a cert from the master key
   USE master;  
   CREATE CERTIFICATE AGNode1Cert   
      WITH SUBJECT = 'AGNode1 Certificate';  
   GO  

   --Backup the cert and transfer it to AGNode2
   BACKUP CERTIFICATE AGNode1Cert TO FILE = 'C:\certs\AGNode1Cert.crt';  
   GO  
   ```

1. Créez ensuite le point de terminaison HADR et utilisez le certificat d'authentification en exécutant l'instruction Transact-SQL (T-SQL) suivante :

   ```sql
   --CREATE or ALTER the mirroring endpoint
   CREATE ENDPOINT hadr_endpoint  
      STATE = STARTED  
      AS TCP (  
         LISTENER_PORT=5022  
         , LISTENER_IP = ALL  
      )   
      FOR DATABASE_MIRRORING (   
         AUTHENTICATION = CERTIFICATE AGNode1Cert  
         , ENCRYPTION = REQUIRED ALGORITHM AES  
         , ROLE = ALL  
      );  
   GO  
   ```

1. Utilisez l'**Explorateur de fichiers** pour accéder à l'emplacement de votre certificat, par exemple `c:\certs`. 
1. Procédez manuellement à une copie du certificat, par exemple `AGNode1Cert.crt`, à partir du premier nœud, puis transférez-la au même emplacement sur le deuxième nœud. 

Pour configurer le deuxième nœud, procédez comme suit : 

1. Connectez-vous au deuxième nœud avec **SQL Server Management Studio** (SSMS), par exemple `AGNode2`. 
1. Dans une fenêtre **Nouvelle requête**, exécutez l'instruction Transact-SQL (T-SQL) suivante après la mise à jour vers un mot de passe complexe et sécurisé : 

   ```sql
   USE master;  
   CREATE MASTER KEY ENCRYPTION BY PASSWORD = 'PassWOrd123!';  
   GO 
   
   --create a cert from the master key
   USE master;  
   CREATE CERTIFICATE AGNode2Cert   
      WITH SUBJECT = 'AGNode2 Certificate';  
   GO  
   --Backup the cert and transfer it to AGNode1
   BACKUP CERTIFICATE AGNode2Cert TO FILE = 'C:\certs\AGNode2Cert.crt';  
   GO
   ```

1. Créez ensuite le point de terminaison HADR et utilisez le certificat d'authentification en exécutant l'instruction Transact-SQL (T-SQL) suivante :

   ```sql
   --CREATE or ALTER the mirroring endpoint
   CREATE ENDPOINT hadr_endpoint  
      STATE = STARTED  
      AS TCP (  
         LISTENER_PORT=5022  
         , LISTENER_IP = ALL  
      )   
      FOR DATABASE_MIRRORING (   
         AUTHENTICATION = CERTIFICATE AGNode2Cert  
         , ENCRYPTION = REQUIRED ALGORITHM AES  
         , ROLE = ALL  
      );  
   GO  
   ```

1. Utilisez l'**Explorateur de fichiers** pour accéder à l'emplacement de votre certificat, par exemple `c:\certs`. 
1. Procédez manuellement à une copie du certificat, par exemple `AGNode2Cert.crt`, à partir du deuxième nœud, puis transférez-la au même emplacement sur le premier nœud. 

Si le cluster comprend d'autres nœuds, répétez-y ces étapes, en modifiant les noms des certificats correspondants. 

## <a name="create-logins"></a>Créer des connexions

L'authentification par certificat est utilisée pour synchroniser les données entre les nœuds. Créez une connexion pour l'autre nœud, créez un utilisateur pour la connexion, créez un certificat pour la connexion afin d'utiliser le certificat sauvegardé, puis accordez la connexion sur le point de terminaison en miroir. 

Pour ce faire, commencez par exécuter la requête Transact-SQL (T-SQL) suivante sur le premier nœud, par exemple `AGNode1` : 

```sql
--create a login for the AGNode2
USE master;  
CREATE LOGIN AGNode2_Login WITH PASSWORD = 'PassWord123!';  
GO  

--create a user from the login
CREATE USER AGNode2_User FOR LOGIN AGNode2_Login;  
GO  

--create a certificate that the login uses for authentication
CREATE CERTIFICATE AGNode2Cert  
   AUTHORIZATION AGNode2_User  
   FROM FILE = 'C:\certs\AGNode2Cert.crt'  
GO 

--grant connect for login
GRANT CONNECT ON ENDPOINT::hadr_endpoint TO [AGNode2_login];  
GO
```

Exécutez ensuite la requête Transact-SQL (T-SQL) suivante sur le deuxième nœud, par exemple `AGNode2` : 

```sql
--create a login for the AGNode1
USE master;  
CREATE LOGIN AGNode1_Login WITH PASSWORD = 'PassWord123!';  
GO  

--create a user from the login
CREATE USER AGNode1_User FOR LOGIN AGNode1_Login;  
GO  

--create a certificate that the login uses for authentication
CREATE CERTIFICATE AGNode1Cert  
   AUTHORIZATION AGNode1_User  
   FROM FILE = 'C:\certs\AGNode1Cert.crt'  
GO 

--grant connect for login
GRANT CONNECT ON ENDPOINT::hadr_endpoint TO [AGNode1_login];  
GO
```

Si le cluster comprend d'autres nœuds, répétez-y ces étapes, en modifiant les noms des certificats et des utilisateurs correspondants. 

## <a name="configure-an-availability-group"></a>Configurer un groupe de disponibilité

Au cours de cette étape, vous allez configurer votre groupe de disponibilité et y ajouter vos bases de données. Ne créez pas d'écouteur pour le moment. Pour en savoir plus sur ces étapes, suivez le [tutoriel consacré aux groupes de disponibilité](availability-group-manually-configure-tutorial.md#create-the-availability-group). Lancez une procédure de basculement et de restauration automatique pour vérifier que tout fonctionne bien. 

   > [!NOTE]
   > En cas d'échec du processus de synchronisation, vous devrez peut-être accorder temporairement à l'administrateur système `NT AUTHORITY\SYSTEM` les droits nécessaires à la création des ressources de cluster sur le premier nœud, par exemple `AGNode1`. 

## <a name="configure-a-load-balancer"></a>Configurer un équilibreur de charge

Dans cette dernière étape, vous allez configurer l’équilibreur de charge à l’aide du [portail Azure](availability-group-load-balancer-portal-configure.md) ou de [PowerShell](availability-group-listener-powershell-configure.md).


## <a name="next-steps"></a>Étapes suivantes

Vous pouvez également utiliser [AZ SQL VM CLI](./availability-group-az-commandline-configure.md) pour configurer un groupe de disponibilité.