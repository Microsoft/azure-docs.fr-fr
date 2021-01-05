---
title: Rejoindre une machine virtuelle Ubuntu à Azure AD Domain Services | Microsoft Docs
description: Découvrez comment configurer une machine virtuelle Ubuntu Linux et comment la joindre à un domaine managé Azure AD Domain Services.
services: active-directory-ds
author: justinha
manager: daveba
ms.assetid: 804438c4-51a1-497d-8ccc-5be775980203
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.topic: how-to
ms.date: 07/13/2020
ms.author: justinha
ms.custom: fasttrack-edit
ms.openlocfilehash: 8b6d022b9a1f3be70f69943a53754c9dd909ca99
ms.sourcegitcommit: 8192034867ee1fd3925c4a48d890f140ca3918ce
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/05/2020
ms.locfileid: "96619485"
---
# <a name="join-an-ubuntu-linux-virtual-machine-to-an-azure-active-directory-domain-services-managed-domain"></a>Joindre une machine virtuelle Ubuntu Linux à un domaine managé par Azure Active Directory Domain Services

Pour permettre aux utilisateurs de se connecter à des machines virtuelles dans Azure en utilisant un ensemble unique d’informations d’identification, vous pouvez joindre les machines virtuelles à un domaine managé Azure Active Directory Domain Services (Azure AD DS). Quand vous joignez une machine virtuelle à un domaine managé Azure AD DS, les comptes d’utilisateurs et les informations d’identification du domaine peuvent être utilisés pour se connecter aux serveurs et les gérer. Les appartenances aux groupes du domaine managé sont également appliquées pour vous permettre de contrôler l’accès aux fichiers ou aux services sur la machine virtuelle.

Cet article vous montre comment joindre une machine virtuelle Ubuntu Linux à un domaine managé.

## <a name="prerequisites"></a>Prérequis

Pour effectuer ce tutoriel, vous avez besoin des ressources et des privilèges suivants :

* Un abonnement Azure actif.
    * Si vous n’avez pas d’abonnement Azure, [créez un compte](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* Un locataire Azure Active Directory associé à votre abonnement, synchronisé avec un annuaire local ou un annuaire cloud uniquement.
    * Si nécessaire, [créez un locataire Azure Active Directory][create-azure-ad-tenant] ou [associez un abonnement Azure à votre compte][associate-azure-ad-tenant].
* Un domaine managé Azure Active Directory Domain Services activé et configuré dans votre locataire Azure AD.
    * Si nécessaire, le premier tutoriel [crée et configure un domaine managé Azure Active Directory Domain Services][create-azure-ad-ds-instance].
* Un compte d’utilisateur membre du domaine managé.

## <a name="create-and-connect-to-an-ubuntu-linux-vm"></a>Créer une machine virtuelle Ubuntu Linux et s’y connecter

Si vous disposez d’une machine virtuelle Ubuntu Linux dans Azure, connectez-vous-y en utilisant SSH, puis passez à l’étape suivante pour [commencer à configurer la machine virtuelle](#configure-the-hosts-file).

Si vous avez besoin de créer une machine virtuelle Ubuntu Linux ou si vous souhaitez créer une machine virtuelle de test dans le cadre de cet article, vous pouvez employer l’une des méthodes suivantes :

* [Azure portal](../virtual-machines/linux/quick-create-portal.md)
* [Azure CLI](../virtual-machines/linux/quick-create-cli.md)
* [Azure PowerShell](../virtual-machines/linux/quick-create-powershell.md)

Au moment de créer la machine virtuelle, faites attention aux paramètres de réseau virtuel et veillez à ce que la machine virtuelle puisse communiquer avec le domaine managé :

* Déployez la machine virtuelle dans le réseau virtuel où vous avez activé Azure Active Directory Domain Services ou dans un réseau virtuel appairé.
* Déployez la machine virtuelle dans un sous-réseau différent de celui de votre instance Azure Active Directory Domain Services.

Une fois la machine virtuelle déployée, suivez les étapes pour vous connecter à la machine virtuelle avec SSH.

## <a name="configure-the-hosts-file"></a>Configurer le fichier hosts

Pour que le nom d’hôte de la machine virtuelle soit correctement configuré pour le domaine managé, modifiez le fichier */etc/hosts* et définissez le nom d’hôte :

```console
sudo vi /etc/hosts
```

Dans le fichier *hosts*, mettez à jour l’adresse *localhost*. Dans l’exemple suivant :

* *aaddscontoso.com* est le nom de domaine DNS de votre domaine managé.
* *ubuntu* est le nom d’hôte de votre machine virtuelle Ubuntu que vous joignez au domaine managé.

Mettez à jour ces noms avec vos propres valeurs :

```console
127.0.0.1 ubuntu.aaddscontoso.com ubuntu
```

Quand vous avez terminé, enregistrez et quittez le fichier *hosts* à l’aide de la commande `:wq` de l’éditeur.

## <a name="install-required-packages"></a>Installer les packages nécessaires

La machine virtuelle a besoin de packages supplémentaires pour être jointe au domaine managé. Pour installer et configurer ces packages, mettez à jour et installez les outils de jonction de domaine à l’aide de `apt-get`

Pendant l’installation de Kerberos, le package *Krb5-user* demande le nom de domaine TOUT EN MAJUSCULES. Par exemple, si le nom de votre domaine managé est *aaddscontoso.com*, entrez *AADDSCONTOSO.COM* en guise de domaine. L’installation écrit les sections `[realm]` et `[domain_realm]` dans le fichier de configuration */etc/krb5.conf*. Veillez à spécifier le domaine TOUT EN MAJUSCULES :

```console
sudo apt-get update
sudo apt-get install krb5-user samba sssd sssd-tools libnss-sss libpam-sss ntp ntpdate realmd adcli
```

## <a name="configure-network-time-protocol-ntp"></a>Configurer le protocole NTP (Network Time Protocol)

Pour que la communication fonctionne correctement dans le domaine, la date et l’heure de votre machine virtuelle Ubuntu doit être synchronisée avec celle du domaine managé. Ajoutez le nom d’hôte NTP de votre domaine managé au fichier */etc/ntp.conf*.

1. Ouvrez le fichier *ntp.conf* avec un éditeur :

    ```console
    sudo vi /etc/ntp.conf
    ```

1. Dans le fichier *ntp.conf*, créez une ligne pour ajouter le nom DNS de votre domaine managé. Dans l’exemple suivant, une entrée est ajoutée pour *aaddscontoso.com*. Utilisez votre propre nom DNS :

    ```console
    server aaddscontoso.com
    ```

    Quand vous avez terminé, enregistrez et quittez le fichier *ntp.conf* à l’aide de la commande `:wq` de l’éditeur.

1. Pour assurer la synchronisation de la machine virtuelle avec le domaine managé, les étapes suivantes sont nécessaires :

    * Arrêter le serveur NTP
    * Mettre à jour la date et l’heure à partir du domaine managé
    * Démarrer le service NTP

    Exécutez les commandes suivantes pour effectuer ces étapes. Utilisez votre propre nom DNS avec la commande `ntpdate` :

    ```console
    sudo systemctl stop ntp
    sudo ntpdate aaddscontoso.com
    sudo systemctl start ntp
    ```

## <a name="join-vm-to-the-managed-domain"></a>Joindre la machine virtuelle au domaine managé

Maintenant que les packages nécessaires sont installés sur la machine virtuelle et que NTP est configuré, joignez la machine virtuelle au domaine managé.

1. Utilisez la commande `realm discover` pour découvrir le domaine managé. L’exemple suivant découvre le domaine *AADDSCONTOSO.COM*. Spécifiez votre propre nom de domaine managé TOUT EN MAJUSCULES :

    ```console
    sudo realm discover AADDSCONTOSO.COM
    ```

   Si la commande `realm discover` ne trouve pas votre domaine managé, examinez les étapes de dépannage suivantes :

    * Vérifiez que le domaine est accessible à partir de la machine virtuelle. Essayez `ping aaddscontoso.com` pour voir si une réponse positive est retournée.
    * Vérifiez que la machine virtuelle est déployée dans le réseau virtuel où le domaine managé est disponible ou dans un réseau virtuel appairé.
    * Vérifiez que les paramètres de serveur DNS du réseau virtuel ont été mis à jour pour pointer vers les contrôleurs de domaine du domaine managé.

1. À présent, initialisez Kerberos à l’aide de la commande `kinit`. Spécifiez un utilisateur membre du domaine managé. Si nécessaire, [ajoutez un compte d’utilisateur à un groupe dans Azure AD](../active-directory/fundamentals/active-directory-groups-members-azure-portal.md).

    Là encore, le nom de domaine managé doit être entré TOUT EN MAJUSCULES. Dans l’exemple suivant, le compte nommé `contosoadmin@aaddscontoso.com` est utilisé pour initialiser Kerberos. Entrez votre propre compte d’utilisateur qui fait partie du domaine managé :

    ```console
    kinit -V contosoadmin@AADDSCONTOSO.COM
    ```

1. Enfin, joignez la machine virtuelle au domaine managé à l’aide de la commande `realm join`. Utilisez le même compte d’utilisateur faisant partie du domaine managé et spécifié dans la commande précédente `kinit`, à savoir `contosoadmin@AADDSCONTOSO.COM` :

    ```console
    sudo realm join --verbose AADDSCONTOSO.COM -U 'contosoadmin@AADDSCONTOSO.COM' --install=/
    ```

La jonction de la machine virtuelle au domaine managé prend quelques instants. L’exemple de sortie suivant montre que la machine virtuelle a bien été jointe au domaine managé :

```output
Successfully enrolled machine in realm
```

Si votre machine virtuelle ne peut pas accomplir le processus de jonction de domaine, vérifiez que le groupe de sécurité réseau de la machine virtuelle autorise le trafic Kerberos sortant sur le port TCP + UDP 464 vers le sous-réseau de réseau virtuel de votre domaine managé.

Si vous avez reçu l’erreur *Échec GSS non spécifié.  Du code mineur peut fournir plus d’informations (serveur introuvable dans la base de données Kerberos)* , ouvrez le fichier */etc/krb5.conf* et ajouter le code suivant dans la section `[libdefaults]` et réessayez :

```console
rdns=false
```

## <a name="update-the-sssd-configuration"></a>Mettre à jour la configuration SSSD

L’un des packages installés à une étape précédente était destiné à SSSD (System Security Services Daemon). Quand un utilisateur tente de se connecter à une machine virtuelle à l’aide d’informations d’identification de domaine, SSSD relaie la demande à un fournisseur d’authentification. Dans ce scénario, SSSD utilise Azure AD DS pour authentifier la demande.

1. Ouvrez le fichier *sshd.conf* avec un éditeur :

    ```console
    sudo vi /etc/sssd/sssd.conf
    ```

1. Commentez la ligne pour *use_fully_qualified_names* comme ceci :

    ```console
    # use_fully_qualified_names = True
    ```

    Quand vous avez terminé, enregistrez et quittez le fichier *sshd.conf* à l’aide de la commande `:wq` de l’éditeur.

1. Pour appliquer la modification, redémarrez le service SSSD :

    ```console
    sudo systemctl restart sssd
    ```

## <a name="configure-user-account-and-group-settings"></a>Configurer les paramètres de groupe et de compte d’utilisateur

Une fois la machine virtuelle jointe au domaine managé et configurée pour l’authentification, il reste quelques options de configuration utilisateur à définir. Parmi ces changements de configuration figurent l’autorisation de l’authentification par mot de passe et la création automatique de répertoires de base sur la machine virtuelle locale quand les utilisateurs du domaine se connectent pour la première fois.

### <a name="allow-password-authentication-for-ssh"></a>Autoriser l’authentification par mot de passe pour SSH

Par défaut, les utilisateurs ne peuvent se connecter à une machine virtuelle qu’avec l’authentification par clé publique SSH. L’authentification par mot de passe échoue. Quand vous joignez la machine virtuelle à un domaine managé, ces comptes de domaine doivent utiliser l’authentification par mot de passe. Mettez à jour la configuration SSH pour autoriser l’authentification par mot de passe comme suit.

1. Ouvrez le fichier *sshd_conf* avec un éditeur :

    ```console
    sudo vi /etc/ssh/sshd_config
    ```

1. Mettez à jour la ligne pour *PasswordAuthentication* sur *yes* :

    ```console
    PasswordAuthentication yes
    ```

    Quand vous avez terminé, enregistrez et quittez le fichier *sshd_conf* à l’aide de la commande `:wq` de l’éditeur.

1. Pour appliquer les modifications et permettre aux utilisateurs de se connecter à l’aide d’un mot de passe, redémarrez le service SSH :

    ```console
    sudo systemctl restart ssh
    ```

### <a name="configure-automatic-home-directory-creation"></a>Configurer la création automatique du répertoire de base

Pour activer la création automatique du répertoire de base quand un utilisateur se connecte pour la première fois, procédez comme suit :

1. Ouvrez le fichier */etc/pam.d/common-session* dans un éditeur :

    ```console
    sudo vi /etc/pam.d/common-session
    ```

1. Ajoutez la ligne suivante dans ce fichier en dessous de la ligne `session optional pam_sss.so` :

    ```console
    session required pam_mkhomedir.so skel=/etc/skel/ umask=0077
    ```

    Quand vous avez terminé, enregistrez et quittez le fichier *common-session* à l’aide de la commande `:wq` de l’éditeur.

### <a name="grant-the-aad-dc-administrators-group-sudo-privileges"></a>Accorder les privilèges sudo au groupe « Administrateurs du contrôleur de domaine AAD »

Pour accorder des privilèges d’administration aux membres du groupe *Administrateurs AAD DC* sur la machine virtuelle Ubuntu, ajoutez une entrée au fichier */etc/sudoers*. Une fois ajoutés, les membres du groupe *Administrateurs AAD DC* peuvent utiliser la commande `sudo` sur la machine virtuelle Ubuntu.

1. Ouvrez le fichier *sudoers* pour le modifier :

    ```console
    sudo visudo
    ```

1. Ajoutez l’entrée suivante à la fin du fichier */etc/sudoers* :

    ```console
    # Add 'AAD DC Administrators' group members as admins.
    %AAD\ DC\ Administrators ALL=(ALL) NOPASSWD:ALL
    ```

    Quand vous avez terminé, enregistrez et quittez l’éditeur à l’aide de la commande `Ctrl-X`.

## <a name="sign-in-to-the-vm-using-a-domain-account"></a>Se connecter à la machine virtuelle à l’aide d’un compte de domaine

Pour vérifier que la machine virtuelle a bien été jointe au domaine managé, démarrez une nouvelle connexion SSH en utilisant un compte d’utilisateur du domaine. Vérifiez qu’un répertoire de base a été créé et que l’appartenance au groupe du domaine est appliquée.

1. Créez une connexion SSH à partir de votre console. Utilisez un compte de domaine qui appartient au domaine managé à l’aide de la commande `ssh -l`, par exemple `contosoadmin@aaddscontoso.com`, puis entrez l’adresse de votre machine virtuelle, par exemple *ubuntu.aaddscontoso.com*. Si vous utilisez Azure Cloud Shell, utilisez l’adresse IP publique de la machine virtuelle plutôt que le nom DNS interne.

    ```console
    ssh -l contosoadmin@AADDSCONTOSO.com ubuntu.aaddscontoso.com
    ```

1. Une fois connecté à la machine virtuelle, vérifiez que le répertoire de base a bien été initialisé :

    ```console
    pwd
    ```

    Vous devez vous trouver dans le répertoire de base */home* et votre propre répertoire doit correspondre au compte d’utilisateur.

1. Vérifiez maintenant que les appartenances aux groupes sont correctement résolues :

    ```console
    id
    ```

    Vos appartenances aux groupes du domaine managé doivent s’afficher.

1. Si vous vous êtes connecté à la machine virtuelle en tant que membre du groupe *Administrateurs AAD DC*, vérifiez que vous pouvez bien utiliser la commande `sudo` :

    ```console
    sudo apt-get update
    ```

## <a name="next-steps"></a>Étapes suivantes

Si vous avez des difficultés à connecter la machine virtuelle au domaine managé ou à vous connecter avec un compte de domaine, consultez [Résoudre les problèmes de jonction à un domaine](join-windows-vm.md#troubleshoot-domain-join-issues).

<!-- INTERNAL LINKS -->
[create-azure-ad-tenant]: ../active-directory/fundamentals/sign-up-organization.md
[associate-azure-ad-tenant]: ../active-directory/fundamentals/active-directory-how-subscriptions-associated-directory.md
[create-azure-ad-ds-instance]: tutorial-create-instance.md
