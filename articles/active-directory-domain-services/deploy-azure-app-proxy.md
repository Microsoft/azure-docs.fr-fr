---
title: Déployer le proxy d'application Azure AD pour Azure AD Domain Services | Microsoft Docs
description: Découvrez comment fournir un accès sécurisé aux applications internes pour les travailleurs à distance en déployant et en configurant des Proxy d’application Azure Active Directory dans un domaine managé Azure Active Directory Domain Services
services: active-directory-ds
author: justinha
manager: daveba
ms.assetid: 938a5fbc-2dd1-4759-bcce-628a6e19ab9d
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.topic: how-to
ms.date: 07/09/2020
ms.author: justinha
ms.openlocfilehash: 8305b2190cf3b157973f5844c3237ffe73adba66
ms.sourcegitcommit: 8192034867ee1fd3925c4a48d890f140ca3918ce
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/05/2020
ms.locfileid: "96619914"
---
# <a name="deploy-azure-ad-application-proxy-for-secure-access-to-internal-applications-in-an-azure-active-directory-domain-services-managed-domain"></a>Déployer le proxy d’application Azure AD pour un accès sécurisé aux applications internes dans un domaine managé Azure Active Directory Domain Services

Grâce à Azure AD Domain Services (Azure AD DS), vous pouvez transférer des applications héritées lift-and-shift s’exécutant localement vers Azure. Le proxy d’application Azure Active Directory (AD) vous aide à prendre en charge les Workers à distance en publiant en toute sécurité les applications internes qui font partie d’un domaine managé Azure AD DS afin qu’elles soient accessibles via Internet.

Si vous débutez avec le Proxy d'application Azure AD et que vous souhaitez en savoir plus, consultez [Comment fournir un accès à distance sécurisé aux applications internes](../active-directory/manage-apps/application-proxy.md).

Cet article explique comment créer et configurer un connecteur de Proxy d’application Azure AD pour fournir un accès sécurisé aux applications dans un domaine managé.

## <a name="before-you-begin"></a>Avant de commencer

Pour faire ce qui est décrit dans cet article, vous avez besoin des ressources et des privilèges suivants :

* Un abonnement Azure actif.
    * Si vous n’avez pas d’abonnement Azure, [créez un compte](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* Un locataire Azure Active Directory associé à votre abonnement, synchronisé avec un annuaire local ou un annuaire cloud uniquement.
    * Si nécessaire, [créez un locataire Azure Active Directory][create-azure-ad-tenant] ou [associez un abonnement Azure à votre compte][associate-azure-ad-tenant].
    * Une **licence Azure AD Premium** est nécessaire pour utiliser le proxy d’application Azure AD.
* Un domaine managé Azure Active Directory Domain Services activé et configuré dans votre locataire Azure AD.
    * Si nécessaire, [créez et configurez un domaine managé Azure Active Directory Domain Services][create-azure-ad-ds-instance].

## <a name="create-a-domain-joined-windows-vm"></a>Créer une machine virtuelle Windows jointe à un domaine

Pour acheminer le trafic vers des applications qui s’exécutent dans votre environnement, vous devez installer le composant connecteur de Proxy d’application Azure AD. Ce connecteur Azure AD Application Proxy doit être installé sur une machine virtuelle Windows Server jointe au domaine managé. Pour certaines applications, vous pouvez déployer plusieurs serveurs sur lesquels le connecteur est installé. Cette option de déploiement vous donne une plus grande disponibilité et vous permet de traiter des charges d’authentification plus importantes.

La machine virtuelle qui exécute le connecteur Azure AD Application Proxy doit se trouver sur le même réseau virtuel que votre domaine managé ou sur un réseau virtuel appairé. Les machines virtuelles qui hébergent ensuite les applications que vous publiez à l’aide du Proxy d’application doivent également être déployées sur le même réseau virtuel Azure.

Pour créer une machine virtuelle pour le connecteur de Proxy d’application Azure AD, procédez comme suit :

1. [Créer une unité d’organisation personnalisée](create-ou.md) Vous pouvez déléguer des autorisations pour gérer cette unité d’organisation personnalisée aux utilisateurs du domaine managé. Les machines virtuelles pour le Proxy d'application Azure AD et qui exécutent vos applications doivent faire partie de l’unité d’organisation personnalisée et non de l’unité d’organisation des *Ordinateurs AAD DC* par défaut.
1. [Joignez au domaine les machines virtuelles][create-join-windows-vm], celle qui exécute le connecteur de Proxy d’application Azure AD et celles qui exécutent vos applications, sur le domaine managé. Créez ces comptes d’ordinateur dans l’unité d’organisation personnalisée de l’étape précédente.

## <a name="download-the-azure-ad-application-proxy-connector"></a>Télécharger le connecteur de Proxy d’application Azure AD

Pour télécharger le connecteur de Proxy d'application Azure AD, effectuez les étapes suivantes. Le fichier d’installation que vous téléchargez est copié sur votre machine virtuelle de Proxy d’application dans la section suivante.

1. Connectez-vous au [Portail Azure](https://portal.azure.com) avec un compte d’utilisateur disposant des autorisations *Administrateur d’entreprise* dans Azure AD.
1. Recherchez et sélectionnez **Azure Active Directory** en haut du portail, puis choisissez **Applications d’entreprise**.
1. Sélectionnez **Proxy d’application** dans le menu de gauche. Pour créer votre premier connecteur et activer le Proxy d’application, sélectionnez le lien pour **télécharger un connecteur**.
1. Sur la page de téléchargement, acceptez les termes du contrat de licence et l'accord de confidentialité, puis sélectionnez **Accepter les termes et télécharger**.

    ![Télécharger le connecteur de Proxy d’application Azure AD](./media/app-proxy/download-app-proxy-connector.png)

## <a name="install-and-register-the-azure-ad-application-proxy-connector"></a>Installer et inscrire le connecteur de Proxy d'application Azure AD

Une fois qu’une machine virtuelle est prête à être utilisée en tant que connecteur de Proxy d'application Azure AD, copiez et exécutez le fichier d’installation téléchargé à partir du Portail Azure.

1. Copiez le fichier d’installation du connecteur de Proxy d'application Azure AD sur votre machine virtuelle.
1. Exécutez le fichier d’installation, tel que *AADApplicationProxyConnectorInstaller.exe*. Acceptez les termes du contrat de licence logicielle.
1. Au cours de l’installation, vous êtes invité à inscrire le connecteur auprès du Proxy d’application de votre répertoire Azure AD.
   * Fournissez vos informations d’identification d’administrateur général dans votre répertoire Azure AD. Les informations d’identification d’administrateur général Azure AD peuvent être différentes de vos informations d’identification Azure dans le portail

        > [!NOTE]
        > Le compte d'administrateur général utilisé pour inscrire le connecteur doit appartenir au même répertoire que celui dans lequel vous avez activé le service Proxy d’application.
        >
        > Par exemple, si le domaine Azure AD est *contoso.com*, l’administrateur général doit être `admin@contoso.com` ou tout autre alias valide sur ce domaine.

   * Si l’option Configuration de sécurité renforcée d’Internet Explorer est activée sur la machine virtuelle sur laquelle vous installez le connecteur, l’écran d’inscription risque d’être bloqué. Pour autoriser l’accès, suivez les instructions du message d’erreur ou désactivez la sécurité renforcée d’Internet Explorer au cours du processus d’installation.
   * Si l’inscription du connecteur échoue, consultez[Détecter un problème du Proxy d’application](../active-directory/manage-apps/application-proxy-troubleshoot.md).
1. À la fin de l’installation, une remarque s’affiche pour les environnements avec un proxy sortant. Pour configurer le connecteur de Proxy d’application Azure AD et qu’il fonctionne par le biais du proxy sortant, exécutez le script fourni, tel que `C:\Program Files\Microsoft AAD App Proxy connector\ConfigureOutBoundProxy.ps1`.
1. Dans la page du proxy d’application du Portail Azure, le nouveau connecteur est répertorié avec l’état *Actif*, comme indiqué dans l’exemple suivant :

    ![Le nouveau connecteur de Proxy d’application Azure AD affiché comme actif dans le Portail Azure](./media/app-proxy/connected-app-proxy.png)

> [!NOTE]
> Pour fournit une haute disponibilité pour l’authentification des applications par le biais du Proxy d’application Azure AD, vous pouvez installer des connecteurs sur plusieurs machines virtuelles. Effectuez les étapes répertoriées dans la section précédente pour installer le connecteur sur d’autres serveurs joints au domaine managé.

## <a name="enable-resource-based-kerberos-constrained-delegation"></a>Activer la délégation Kerberos contrainte basée sur les ressources

Si vous voulez utiliser l’authentification unique pour vos applications avec l’authentification Windows intégrée, octroyez aux connecteurs de Proxy d’application Azure AD l’autorisation d’emprunter l’identité des utilisateurs et d’envoyer et de recevoir des jetons en leur nom. Pour octroyer ces autorisations, configurez la délégation Kerberos contrainte (KCD) pour le connecteur afin d’accéder aux ressources sur le domaine managé. Comme vous n’avez pas de privilèges d’administrateur de domaine dans un domaine managé, les KCD de niveau de compte traditionnels ne peuvent pas être configurés sur un domaine managé. Utilisez à la place KCD basée sur des ressources.

Pour plus d’informations, consultez [Configurer la délégation Kerberos contrainte (KCD) dans Azure Active Directory Domain Services](deploy-kcd.md).

> [!NOTE]
> Vous devez vous connecter à un compte d’utilisateur membre du groupe *Administrateurs Azure AD DC* de votre abonné Azure AD pour exécuter les cmdlets PowerShell suivantes.
>
> Les comptes d’ordinateur de votre machine virtuelle du connecteur Proxy d’application et machines virtuelles d’application doivent se trouver dans une unité d’organisation personnalisée dans laquelle vous êtes autorisé à configurer le KCD basé sur des ressources. Vous ne pouvez pas configurer le mécanisme KCD basé sur les ressources pour un compte d’ordinateur situé dans le conteneur intégré *Ordinateurs du contrôleur de domaine AAD*.

Utilisez la commande [Get-ADComputer][Get-ADComputer] pour récupérer les paramètres de l’ordinateur sur lequel est installé le connecteur de Proxy d’application Azure AD. Sur votre machine virtuelle de gestion jointe au domaine, connectez-vous avec un compte d’utilisateur membre du groupe *Azure AD DC Administrators*, puis exécutez les applets de commande suivantes.

L’exemple suivant obtient des informations sur le compte d’ordinateur nommé *appproxy.aaddscontoso.com*. Fournissez votre propre nom d’ordinateur pour la machine virtuelle de Proxy d'application Azure AD configurée dans les étapes précédentes.

```powershell
$ImpersonatingAccount = Get-ADComputer -Identity appproxy.aaddscontoso.com
```

Pour chaque serveur d’applications qui exécute les applications derrière le Proxy d'application Azure AD, utilisez la cmdlet PowerShell [Set-ADComputer][Set-ADComputer] pour configurer des KCD basés sur des ressources. Dans l’exemple suivant, le connecteur de Proxy d’application Azure AD est autorisé à utiliser l’ordinateur *appserver.aaddscontoso.com* :

```powershell
Set-ADComputer appserver.aaddscontoso.com -PrincipalsAllowedToDelegateToAccount $ImpersonatingAccount
```

Si vous déployez plusieurs connecteurs de Proxy d’application Azure AD, vous devez configurer des KCD basés sur des ressources pour chaque instance de connecteur.

## <a name="next-steps"></a>Étapes suivantes

Avec le Proxy d'application Azure AD intégré à Azure AD DS, publiez des applications auxquelles les utilisateurs peuvent accéder. Pour plus d’informations, consultez [Publier des applications à l’aide du Proxy d’application Azure AD](../active-directory/manage-apps/application-proxy-add-on-premises-application.md).

<!-- INTERNAL LINKS -->
[create-azure-ad-tenant]: ../active-directory/fundamentals/sign-up-organization.md
[associate-azure-ad-tenant]: ../active-directory/fundamentals/active-directory-how-subscriptions-associated-directory.md
[create-azure-ad-ds-instance]: tutorial-create-instance.md
[create-join-windows-vm]: join-windows-vm.md
[azure-bastion]: ../bastion/tutorial-create-host-portal.md
[Get-ADComputer]: /powershell/module/addsadministration/get-adcomputer
[Set-ADComputer]: /powershell/module/addsadministration/set-adcomputer