---
title: Configurer le chiffrement Kerberos NFSv4.1 pour Azure NetApp Files | Microsoft Docs
description: Décrit la procédure de configuration du chiffrement Kerberos NFSv4.1 pour Azure NetApp Files et l’impact sur les performances.
services: azure-netapp-files
documentationcenter: ''
author: b-juche
manager: ''
editor: ''
ms.assetid: ''
ms.service: azure-netapp-files
ms.workload: storage
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 11/09/2020
ms.author: b-juche
ms.openlocfilehash: 1ffbcc24a2ee386be1a8ce50b55375d5bd458df1
ms.sourcegitcommit: 17b36b13857f573639d19d2afb6f2aca74ae56c1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/10/2020
ms.locfileid: "94410094"
---
# <a name="configure-nfsv41-kerberos-encryption-for-azure-netapp-files"></a>Configurer le chiffrement Kerberos NFSv4.1 pour Azure NetApp Files

Azure NetApp Files prend en charge le chiffrement client NFS dans les modes Kerberos (krb5, krb5i, et krb5p) avec le chiffrement AES-256. Cet article décrit les configurations requises pour l’utilisation d’un volume NFSv4.1 avec le chiffrement Kerberos.

## <a name="requirements"></a>Spécifications

La configuration requise suivante s’applique au chiffrement client NFSv4.1 : 

* Connexion d’Active Directory Domain Services (AD DS) pour faciliter la création de tickets Kerberos 
* Création d’un enregistrement DNS A/PTR pour les adresses IP du client et du serveur NFS Azure NetApp Files
* Un client Linux  
    Cet article propose des conseils pour les clients RHEL et Ubuntu.  Les autres clients fonctionneront avec des étapes de configuration similaires. 
* Accès au serveur NTP  
    Vous pouvez utiliser un des contrôleurs de domaine Active Directory (AD DC) les plus utilisés.

## <a name="create-an-nfs-kerberos-volume"></a>Créer un volume NFS Kerberos

1.  Suivez les étapes décrites dans [Créer un volume NFS pour Azure NetApp Files](azure-netapp-files-create-volumes.md) pour créer le volume NFSv4.1.   

    Sur la page Créer un volume, définissez la version NFS sur **NFSv4.1** et définissez Kerberos sur **Activé**.

    > [!IMPORTANT] 
    > Vous ne pouvez pas modifier la sélection d’activation Kerberos après la création du volume.

    ![Créer un volume Kerberos NFSv4.1](../media/azure-netapp-files/create-kerberos-volume.png)  

2. Sélectionnez **Exporter la stratégie** pour correspondre à l’option (Kerberos 5, Kerberos 5i ou Kerberos 5p) de niveau d’accès et de sécurité voulu pour le volume.   

    Pour connaitre l’impact de Kerberos sur les performances, consultez [Impact de Kerberos sur les performances sur NFSv4.1](#kerberos_performance).  

    Vous pouvez également modifier les méthodes de sécurité Kerberos concernant le volume en cliquant sur Exporter la stratégie dans le volet de navigation Azure NetApp Files.

3.  Cliquez sur **Vérifier + Créer** pour créer le volume NFSv4.1.

## <a name="configure-the-azure-portal"></a>Configurer le portail Azure 

1.  Suivez les instructions dans [Créer une connexion Active Directory](azure-netapp-files-create-volumes-smb.md#create-an-active-directory-connection).  

    Kerberos requiert la création d’au moins un compte d’ordinateur dans Active Directory. Les informations de compte que vous fournissez sont utilisées pour créer les comptes pour les volumes SMB *et* Kerberos NFSv4.1. Cet ordinateur est automatiquement créé lors de la création du volume.

2.  Sous **Domaine Kerberos** , entrez le **nom du serveur AD** et l’adresse **IP KDC**.

    Le serveur AD et l’adresse IP du KDC peuvent être le même serveur. Ces informations permettent de créer le compte d’ordinateur SPN utilisé par Azure NetApp Files. Une fois le compte d’ordinateur créé, Azure NetApp Files utilisera les enregistrements du serveur DNS pour localiser des serveurs KDC supplémentaires si nécessaire. 

    ![Domaine Kerberos](../media/azure-netapp-files/kerberos-realm.png)
 
3.  Cliquez sur **Joindre** pour enregistrer la configuration.

## <a name="configure-active-directory-connection"></a>Configurer la connexion Azure Active Directory 

La configuration de Kerberos NFSv4.1 crée deux comptes d’ordinateur dans Active Directory :
* Un compte d’ordinateur pour les partages SMB
* Un compte d’ordinateur pour NFSv4.1 : vous pouvez identifier ce compte par le biais du préfixe `NFS-`. 

Après la création du premier volume Kerberos NFSv4.1, définissez le type de chiffrement pour le compte d’ordinateur à l’aide de la commande PowerShell suivante :

`Set-ADComputer $NFSCOMPUTERACCOUNT -KerberosEncryptionType AES256`

## <a name="configure-the-nfs-client"></a>Configurer le client NFS 

Suivez les instructions dans [Configurer un client NFS pour Azure NetApp Files](configure-nfs-clients.md) pour configurer le client NFS.  

## <a name="mount-the-nfs-kerberos-volume"></a><a name="kerberos_mount"></a>Monter un volume NFS Kerberos

1. Sur la page **Volumes** , sélectionnez le volume NFS que vous souhaitez monter.

2. Sélectionnez **Instructions de montage** à partir du volume pour afficher les instructions.

    Par exemple : 

    ![Instructions de montage pour les volumes Kerberos](../media/azure-netapp-files/mount-instructions-kerberos-volume.png)  

3. Créez le répertoire (point de montage) pour le nouveau volume.  

4. Définissez le type de chiffrement par défaut sur AES 256 pour le compte d’ordinateur :  
    `Set-ADComputer $NFSCOMPUTERACCOUNT -KerberosEncryptionType AES256 -Credential $ANFSERVICEACCOUNT`

    * Vous ne devez exécuter cette commande qu’une seule fois pour chaque compte d’ordinateur.
    * Vous pouvez exécuter cette commande à partir d’un contrôleur de domaine ou d’un PC sur lequel [RSAT](https://support.microsoft.com/help/2693643/remote-server-administration-tools-rsat-for-windows-operating-systems) est installé. 
    * La variable `$NFSCOMPUTERACCOUNT` est le compte d’ordinateur créé dans Active Directory lorsque vous déployez le volume Kerberos. Il s’agit du compte avec le préfixe `NFS-`. 
    * La variable `$ANFSERVICEACCOUNT` est un compte d'utilisateur Active Directory non privilégié avec contrôles délégués sur l’unité d'organisation dans laquelle le compte d'ordinateur a été créé. 

5. Montez le volume sur l’hôte : 

    `sudo mount -t nfs -o sec=krb5p,rw,hard,rsize=1048576,wsize=1048576,vers=4.1,tcp $ANFEXPORT $ANFMOUNTPOINT`

    * La variable `$ANFEXPORT` est le chemin d’accès `host:/export` trouvé dans les instructions de montage.
    * La variable `$ANFMOUNTPOINT` est le dossier créé par l’utilisateur sur l’hôte Linux.

## <a name="performance-impact-of-kerberos-on-nfsv41"></a><a name="kerberos_performance"></a>Impact de Kerberos sur les performances de NFSv4.1 

Cette section vous aide à comprendre l’impact de Kerberos sur les performances de NFSv4.1.

### <a name="available-security-options"></a>Options de sécurité disponibles 

Les options de sécurité actuellement disponibles pour les volumes NFSv4.1 sont les suivantes : 

* **sec = sys** utilise des UID et GID UNIX locaux à l’aide d’AUTH_SYS pour authentifier les opérations NFS.
* **sec = krb5** utilise Kerberos V5 au lieu des UID et GID UNIX locaux pour authentifier les utilisateurs.
* **sec=krb5i** utilise Kerberos V5 pour authentifier les utilisateurs et effectuer des vérifications d’intégrité des opérations NFS qui utilisent des sommes de contrôle sécurisées pour prévenir la falsification de données.
* **s = krb5p** utilise Kerberos V5 pour authentifier les utilisateurs et effectuer des vérifications d’intégrité. Il chiffre le trafic NFS pour empêcher l’espionnage du trafic. Cette option est le paramètre le plus sécurisé, mais elle implique également la surcharge de performance la plus importante.

### <a name="performance-vectors-tested"></a>Vecteurs de performances testés

Cette section décrit l’impact des différentes options de `sec=*` sur les performances côté client uniquement.

* L’impact sur les performances a été testé à deux niveaux : faible concurrence (faible charge) et concurrence élevée (limites supérieures d’E/S et débit).  
* Trois types de charges de travail ont été testés :  
    * Lecture/écriture aléatoire de petites opérations (à l’aide de FIO)
    * Lecture/écriture aléatoire de grandes opérations (à l’aide de FIO)
    * Charge de travail importante de métadonnées telle que générée par les applications, par exemple, git

### <a name="expected-performance-impact"></a>Impact des performances attendues 

Il existe deux zones principales : la charge faible et la limite supérieure. Les listes suivantes décrivent l’impact sur les performances de paramètre de sécurité à paramètre de sécurité et de scénario à scénario. Toutes les comparaisons sont effectuées par rapport au paramètre de sécurité `sec=sys`. Le test a été effectué sur un seul volume, à l’aide d’un seul client. 

Impact de krb5 sur les performances :

* Concurrence faible (R/W) :
    * La latence séquentielle a augmenté de 0,3 ms.
    * La latence d’E/S aléatoire a augmenté de 0,2 ms.
    * La latence d’E/S de métadonnées a augmenté de 0,2 ms.
* Concurrence élevée (R/W) : 
    * Le débit séquentiel maximal n’a pas été affecté par krb5.
    * Le taux d’E/S maximal aléatoire a baissé de 30 % pour les charges de travail en lecture seule avec l’impact global dégringolant à zéro lorsque la charge de travail passe en écriture seule. 
    * La charge de travail de métadonnées maximale a baissé de 30 %.

Impact de krb5 sur les performances : 

* Concurrence faible (R/W) :
    * La latence séquentielle a augmenté de 0,5 ms.
    * La latence d’E/S aléatoire a augmenté de 0,2 ms.
    * La latence d’E/S de métadonnées a augmenté de 0,2 ms.
* Concurrence élevée (R/W) : 
    * Débit séquentiel maximal réduit de 70 % en général, quel que soit le mélange de la charge de travail.
    * Le taux d’E/S maximal aléatoire a baissé de 50 % pour les charges de travail en lecture seule avec l’impact global diminuant de 25 % lorsque la charge de travail passe en écriture seule. 
    * La charge de travail de métadonnées maximale a baissé de 30 %.

Impact de krb5 sur les performances :

* Concurrence faible (R/W) :
    * La latence séquentielle a augmenté de 0,8 ms.
    * La latence d’E/S aléatoire a augmenté de 0,2 ms.
    * La latence d’E/S de métadonnées a augmenté de 0,2 ms.
* Concurrence élevée (R/W) : 
    * Débit séquentiel maximal réduit de 85 % en général, quel que soit le mélange de la charge de travail. 
    * Le taux d’E/S maximal aléatoire a baissé de 65 % pour les charges de travail en lecture seule avec l’impact global diminuant de 43 % lorsque la charge de travail passe en écriture seule. 
    * La charge de travail de métadonnées maximale a baissé de 30 %.

## <a name="next-steps"></a>Étapes suivantes  

* [Questions fréquentes (FAQ) sur Azure NetApp Files](azure-netapp-files-faqs.md)
* [Créer un volume NFS pour Azure NetApp Files](azure-netapp-files-create-volumes.md)
* [Créer une connexion Active Directory](azure-netapp-files-create-volumes-smb.md#create-an-active-directory-connection)
* [Configurer un client NFS pour Azure NetApp Files](configure-nfs-clients.md) 
