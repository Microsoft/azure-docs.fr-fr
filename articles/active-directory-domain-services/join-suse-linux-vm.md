---
title: Joindre une machine virtuelle SLA à Azure AD Domain Services | Microsoft Docs
description: Découvrez comment configurer une machine virtuelle SUSE Linux Enterprise et comment la joindre à un domaine managé Azure AD Domain Services.
services: active-directory-ds
author: justinha
manager: daveba
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.topic: how-to
ms.date: 08/12/2020
ms.author: justinha
ms.openlocfilehash: f2f421d95dfc376aed373c718198db33a870d9dc
ms.sourcegitcommit: 8192034867ee1fd3925c4a48d890f140ca3918ce
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/05/2020
ms.locfileid: "96619604"
---
# <a name="join-a-suse-linux-enterprise-virtual-machine-to-an-azure-active-directory-domain-services-managed-domain"></a>Joindre une machine virtuelle SUSE Linux Enterprise à un domaine managé par Azure Active Directory Domain Services

Pour permettre aux utilisateurs de se connecter à des machines virtuelles dans Azure en utilisant un ensemble unique d’informations d’identification, vous pouvez joindre les machines virtuelles à un domaine managé Azure Active Directory Domain Services (Azure AD DS). Quand vous joignez une machine virtuelle à un domaine managé Azure AD DS, les comptes d’utilisateurs et les informations d’identification du domaine peuvent être utilisés pour se connecter aux serveurs et les gérer. Les appartenances aux groupes du domaine managé sont également appliquées pour vous permettre de contrôler l’accès aux fichiers ou aux services sur la machine virtuelle.

Cet article montre comment joindre une machine virtuelle SUSE Linux Enterprise (SLE) à un domaine managé.

## <a name="prerequisites"></a>Prérequis

Pour effectuer ce tutoriel, vous avez besoin des ressources et des privilèges suivants :

* Un abonnement Azure actif.
    * Si vous n’avez pas d’abonnement Azure, [créez un compte](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* Un locataire Azure Active Directory associé à votre abonnement, synchronisé avec un annuaire local ou un annuaire cloud uniquement.
    * Si nécessaire, [créez un locataire Azure Active Directory][create-azure-ad-tenant] ou [associez un abonnement Azure à votre compte][associate-azure-ad-tenant].
* Un domaine managé Azure Active Directory Domain Services activé et configuré dans votre locataire Azure AD.
    * Si nécessaire, le premier tutoriel [crée et configure un domaine managé Azure Active Directory Domain Services][create-azure-ad-ds-instance].
* Un compte d’utilisateur membre du domaine managé.

## <a name="create-and-connect-to-a-sle-linux-vm"></a>Créer une machine virtuelle Linux SLE et s’y connecter

Si vous disposez d’une machine virtuelle Linux SLE dans Azure, connectez-vous-y en utilisant SSH, puis passez à l’étape suivante pour [commencer à configurer la machine virtuelle](#configure-the-hosts-file).

Si vous avez besoin de créer une machine virtuelle Linux SLE ou si vous souhaitez créer une machine virtuelle de test dans le cadre de cet article, vous pouvez employer l’une des méthodes suivantes :

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
* *linux-q2gr* est le nom d’hôte de votre machine virtuelle SLE que vous joignez au domaine managé.

Mettez à jour ces noms avec vos propres valeurs :

```console
127.0.0.1 linux-q2gr linux-q2gr.aaddscontoso.com
```

Quand vous avez terminé, enregistrez et quittez le fichier *hosts* à l’aide de la commande `:wq` de l’éditeur.

## <a name="join-vm-to-the-managed-domain-using-sssd"></a>Joindre la machine virtuelle au domaine managé en utilisant SSSD

Pour joindre le domaine managé à l’aide de **SSSD** et du module *User Logon Management* de YaST, procédez comme suit :

1. Installez le module YaST *User Logon Management* :

    ```bash
    sudo zypper install yast2-auth-client
    ```

1. Ouvrez YaST.

1. Pour utiliser la découverte automatique DNS ultérieurement, configurez les adresses IP de domaine managé (*Active Directory server*) en tant que serveur de nom pour votre client.

    Dans YaST, sélectionnez **System > Network Settings**.

1. Sélectionnez l’onglet *Hostname/DNS*, puis entrez la ou les adresses IP du domaine managé dans la zone de texte *Name Server 1*. Ces adresses IP sont affichées dans la fenêtre *Propriétés* du portail Azure pour votre domaine managé, par exemple *10.0.2.4* et *10.0.2.5*.

    Ajoutez vos propres adresses IP de domaine managé, puis sélectionnez **OK**.

1. Dans la fenêtre principale de YaST, choisissez *Network Services* > *User Logon Management*.

    Le module s’ouvre avec une vue d’ensemble montrant les différentes propriétés réseau de votre ordinateur et la méthode d’authentification actuellement utilisée, comme le montre l’exemple de capture d’écran suivant :

    ![Exemple de capture d’écran de la fenêtre User Login Management dans YaST](./media/join-suse-linux-vm/overview-window.png)

    Pour commencer la modification, sélectionnez **Change Settings**.

Pour joindre une machine virtuelle à un domaine managé, procédez comme suit :

1. Dans la boîte de dialogue, sélectionnez **Add Domain**.

1. Spécifiez le nom de domaine (*Domain name*) correct, par exemple *aaddscontoso.com*, puis spécifiez les services à utiliser pour les données d’identité et l’authentification. Sélectionnez *Microsoft Active Directory* pour les deux.

    Assurez-vous que l’option *Enable the domain* est sélectionnée.

1. Lorsque vous êtes prêt, sélectionnez **OK**.

1. Acceptez les paramètres par défaut dans la boîte de dialogue suivante, puis sélectionnez **OK**.

1. La machine virtuelle installe des logiciels supplémentaires le cas échéant, puis vérifie si le domaine managé est disponible.

    Si tout est correct, la boîte de dialogue suivante s’affiche pour indiquer que la machine virtuelle a découvert le domaine managé, mais que vous n’êtes pas encore inscrit (*Not yet enrolled*).

    ![Exemple de capture d’écran de la fenêtre d’inscription Active Directory dans YaST](./media/join-suse-linux-vm/enroll-window.png)

1. Dans la boîte de dialogue, spécifiez le nom d’utilisateur (*Username*) et le mot de passe (*Password*) d’un utilisateur qui fait partie du domaine managé. Si nécessaire, [ajoutez un compte d’utilisateur à un groupe dans Azure AD](../active-directory/fundamentals/active-directory-groups-members-azure-portal.md).

    Pour vous assurer que le domaine actuel est activé pour Samba, activez l’option *Overwrite Samba configuration to work with this AD*.

1. Pour vous inscrire, sélectionnez **OK**.

1. Un message s’affiche pour confirmer que vous êtes correctement inscrit. Pour terminer, sélectionnez **OK**.

Une fois la machine virtuelle inscrite dans le domaine managé, configurez le client en utilisant *Manage Domain User Logon*, comme illustré dans la capture d’écran suivante :

![Exemple de capture d’écran de la fenêtre Manage Domain User Logon dans YaST](./media/join-suse-linux-vm/manage-domain-user-logon-window.png)

1. Pour autoriser les connexions à l’aide des données fournies par le domaine managé, cochez la case de l’option *Allow Domain User Logon*.

1. Si vous le souhaitez, sous *Enable domain data source*, cochez les sources de données supplémentaires nécessaires pour votre environnement. Ces options incluent les utilisateurs autorisés à utiliser **sudo** ou les lecteurs réseau disponibles.

1. Pour permettre aux utilisateurs du domaine managé d’avoir des répertoires de base sur la machine virtuelle, cochez la case de l’option *Create Home Directories*.

1. Dans la barre latérale, sélectionnez **Service Options › Nom à changer**, puis *Extended Options*. Dans cette fenêtre, sélectionnez *fallback_homedir* ou *override_homedir*, puis sélectionnez **Add**.

1. Spécifiez une valeur pour l’emplacement du répertoire de base. Pour que les répertoires de base respectent le format */home/USER_NAME*, utilisez */home/%u*. Pour plus d’informations sur les variables possibles, consultez la page man sssd.conf (`man 5 sssd.conf`), section *override_homedir*.

1. Sélectionnez **OK**.

1. Pour enregistrer les modifications, sélectionnez **OK**. Vérifiez ensuite que les valeurs affichées sont désormais correctes. Pour quitter la boîte de dialogue, sélectionnez **Cancel**.

1. Si vous avez l’intention d’exécuter simultanément SSSD et Winbind (par exemple, lors de la jointure via SSSD, mais en exécutant un serveur de fichiers Samba), l’option Samba *kerberos method* doit être définie sur *secrets and keytab* dans smb.conf. L’option SSSD *ad_update_samba_machine_account_password* doit également être définie sur *true* dans sssd.conf. Ces options empêchent le keytab système de se désynchroniser.

## <a name="join-vm-to-the-managed-domain-using-winbind"></a>Joindre la machine virtuelle au domaine managé en utilisant Winbind

Pour joindre le domaine managé à l’aide de **Winbind** et du module *Windows Domain Membership* de YaST, procédez comme suit :

1. Dans YaST, sélectionnez **Network Services > Windows Domain Membership**.

1. Entrez le domaine à joindre dans *Domain or Workgroup* dans l’écran *Windows Domain Membership*. Entrez le nom de domaine managé, par exemple *aaddscontoso.com*.

    ![Exemple de capture d’écran de la fenêtre Windows Domain Membership dans YaST](./media/join-suse-linux-vm/samba-client-window.png)

1. Afin d’utiliser la source SMB pour l’authentification Linux, activez l’option *Use SMB Information for Linux Authentication*.

1. Pour créer automatiquement un répertoire de base local pour les utilisateurs de domaine managé sur la machine virtuelle, activez l’option *Create Home Directory on Login*.

1. Activez l’option *Offline Authentication* pour permettre aux utilisateurs du domaine de se connecter même si le domaine managé est temporairement indisponible.

1. Si vous souhaitez modifier les plages UID et GID pour les utilisateurs et groupes Samba, sélectionnez *Expert Settings*.

1. Configurez la synchronisation de l’heure NTP (Network Time Protocol) pour votre domaine managé en sélectionnant *NTP Configuration*. Entrez les adresses IP du domaine managé. Ces adresses IP sont affichées dans la fenêtre *Propriétés* du portail Azure pour votre domaine managé, par exemple *10.0.2.4* et *10.0.2.5*.

1. Sélectionnez **OK** et confirmez la jonction de domaine lorsque vous y êtes invité.

1. Indiquez le mot de passe d’un administrateur du domaine managé, puis sélectionnez **OK**.

    ![Exemple de capture d’écran de l’invite de la boîte de dialogue d’authentification lorsque vous joignez une machine virtuelle SLE au domaine managé](./media/join-suse-linux-vm/domain-join-authentication-prompt.png)

Une fois que vous avez joint le domaine managé, vous pouvez vous y connecter à partir de votre station de travail à l’aide du gestionnaire d’affichage de votre bureau ou de la console.

## <a name="join-vm-to-the-managed-domain-using-winbind-from-the-yast-command-line-interface"></a>Joindre la machine virtuelle au domaine managé à l’aide de Winbind à partir de l’interface de ligne de commande YaST

Pour joindre le domaine managé à l’aide de **winbind** et de l’interface de ligne de commande *YaST* :

* Joindre le domaine :

  ```console
  sudo yast samba-client joindomain domain=aaddscontoso.com user=<admin> password=<admin password> machine=<(optional) machine account>
  ```

## <a name="join-vm-to-the-managed-domain-using-winbind-from-the-terminal"></a>Joindre la machine virtuelle au domaine managé à l’aide de Winbind à partir du terminal

Pour joindre le domaine managé à l’aide de **Winbind** et de l’interface de ligne de *commande `samba net`* :

1. Installez le client Kerberos et samba-winbind :

   ```console
   sudo zypper in krb5-client samba-winbind
   ```

2. Modifiez les fichiers de configuration :

   * /etc/samba/smb.conf
   
     ```ini
     [global]
         workgroup = AADDSCONTOSO
         usershare allow guests = NO #disallow guests from sharing
         idmap config * : backend = tdb
         idmap config * : range = 1000000-1999999
         idmap config AADDSCONTOSO : backend = rid
         idmap config AADDSCONTOSO : range = 5000000-5999999
         kerberos method = secrets and keytab
         realm = AADDSCONTOSO.COM
         security = ADS
         template homedir = /home/%D/%U
         template shell = /bin/bash
         winbind offline logon = yes
         winbind refresh tickets = yes
     ```

   * /etc/krb5.conf
   
     ```ini
     [libdefaults]
         default_realm = AADDSCONTOSO.COM
         clockskew = 300
     [realms]
         AADDSCONTOSO.COM = {
             kdc = PDC.AADDSCONTOSO.COM
             default_domain = AADDSCONTOSO.COM
             admin_server = PDC.AADDSCONTOSO.COM
         }
     [domain_realm]
         .aaddscontoso.com = AADDSCONTOSO.COM
     [appdefaults]
         pam = {
             ticket_lifetime = 1d
             renew_lifetime = 1d
             forwardable = true
             proxiable = false
             minimum_uid = 1
         }
     ```

   * /etc/security/pam_winbind.conf
   
     ```ini
     [global]
         cached_login = yes
         krb5_auth = yes
         krb5_ccache_type = FILE
         warn_pwd_expire = 14
     ```

   * /etc/nsswitch.conf
   
     ```ini
     passwd: compat winbind
     group: compat winbind
     ```

3. Vérifiez que la date et l’heure dans Azure AD et Linux sont synchronisées. Pour ce faire, vous pouvez ajouter le serveur Azure AD au service NTP :
   
   1. Ajoutez la ligne suivante au fichier /etc/ntp.conf :
     
      ```console
      server aaddscontoso.com
      ```

   1. Redémarrez le service NTP :
     
      ```console
      sudo systemctl restart ntpd
      ```

4. Joignez le domaine :

   ```console
   sudo net ads join -U Administrator%Mypassword
   ```

5. Activez winbind comme source de connexion dans les modules PAM (Pluggable Authentication Modules) Linux :

   ```console
   pam-config --add --winbind
   ```

6. Activez la création automatique des répertoires de démarrage afin que les utilisateurs puissent se connecter :

   ```console
   pam-config -a --mkhomedir
   ```

7. Démarrez et activez le service Winbind :

   ```console
   sudo systemctl enable winbind
   sudo systemctl start winbind
   ```

## <a name="allow-password-authentication-for-ssh"></a>Autoriser l’authentification par mot de passe pour SSH

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
    sudo systemctl restart sshd
    ```

## <a name="grant-the-aad-dc-administrators-group-sudo-privileges"></a>Accorder les privilèges sudo au groupe « Administrateurs du contrôleur de domaine AAD »

Pour accorder des privilèges administratifs aux membres du groupe *AAD DC Administrators* sur la machine virtuelle SLE, ajoutez une entrée au fichier */etc/sudoers*. Une fois ajoutés, les membres du groupe *AAD DC Administrators* peuvent utiliser la commande `sudo` sur la machine virtuelle SLE.

1. Ouvrez le fichier *sudoers* pour le modifier :

    ```console
    sudo visudo
    ```

1. Ajoutez l’entrée suivante à la fin du fichier */etc/sudoers*. Étant donné que le nom de groupe *Administrateurs du contrôleur de domaine AAD* contient un espace, incluez-y une barre oblique inverse en guise de caractère d’échappement. Ajoutez votre propre nom de domaine, par exemple *aaddscontoso.com* :

    ```console
    # Add 'AAD DC Administrators' group members as admins.
    %AAD\ DC\ Administrators@aaddscontoso.com ALL=(ALL) NOPASSWD:ALL
    ```

    Quand vous avez terminé, enregistrez et quittez l’éditeur à l’aide de la commande `:wq` de l’éditeur.

## <a name="sign-in-to-the-vm-using-a-domain-account"></a>Se connecter à la machine virtuelle à l’aide d’un compte de domaine

Pour vérifier que la machine virtuelle a bien été jointe au domaine managé, démarrez une nouvelle connexion SSH en utilisant un compte d’utilisateur du domaine. Vérifiez qu’un répertoire de base a été créé et que l’appartenance au groupe du domaine est appliquée.

1. Créez une connexion SSH à partir de votre console. Utilisez un compte de domaine qui appartient au domaine managé à l’aide de la commande `ssh -l`, par exemple `contosoadmin@aaddscontoso.com`, puis entrez l’adresse de votre machine virtuelle, par exemple *linux-q2gr.aaddscontoso.com*. Si vous utilisez Azure Cloud Shell, utilisez l’adresse IP publique de la machine virtuelle plutôt que le nom DNS interne.

    ```console
    ssh -l contosoadmin@AADDSCONTOSO.com linux-q2gr.aaddscontoso.com
    ```

2. Une fois connecté à la machine virtuelle, vérifiez que le répertoire de base a bien été initialisé :

    ```console
    pwd
    ```

    Vous devez vous trouver dans le répertoire de base */home* et votre propre répertoire doit correspondre au compte d’utilisateur.

3. Vérifiez maintenant que les appartenances aux groupes sont correctement résolues :

    ```console
    id
    ```

    Vos appartenances aux groupes du domaine managé doivent s’afficher.

4. Si vous vous êtes connecté à la machine virtuelle en tant que membre du groupe *Administrateurs AAD DC*, vérifiez que vous pouvez bien utiliser la commande `sudo` :

    ```console
    sudo zypper update
    ```

## <a name="next-steps"></a>Étapes suivantes

Si vous avez des difficultés à connecter la machine virtuelle au domaine managé ou à vous connecter avec un compte de domaine, consultez [Résoudre les problèmes de jonction à un domaine](join-windows-vm.md#troubleshoot-domain-join-issues).

<!-- INTERNAL LINKS -->
[create-azure-ad-tenant]: ../active-directory/fundamentals/sign-up-organization.md
[associate-azure-ad-tenant]: ../active-directory/fundamentals/active-directory-how-subscriptions-associated-directory.md
[create-azure-ad-ds-instance]: tutorial-create-instance.md
