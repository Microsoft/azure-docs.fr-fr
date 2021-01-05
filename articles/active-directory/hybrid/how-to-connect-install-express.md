---
title: 'Azure AD Connect : Bien démarrer avec les paramètres express | Microsoft Docs'
description: Découvrez comment télécharger, installer et exécuter l’Assistant d’installation d’Azure AD Connect.
services: active-directory
author: billmath
manager: daveba
editor: curtand
ms.assetid: b6ce45fd-554d-4f4d-95d1-47996d561c9f
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 09/28/2018
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 0a655f355bb77d937f4daff2f8987769416ebd8c
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89279667"
---
# <a name="getting-started-with-azure-ad-connect-using-express-settings"></a>Prise en main d’Azure AD Connect à l’aide de paramètres express
La **configuration rapide** d’Azure AD Connect est utilisée lorsque vous disposez d’une topologie à une forêt unique et quand la [synchronisation de hachage du mot de passe](how-to-connect-password-hash-synchronization.md) est utilisée pour l’authentification. **configuration rapide** est l’option par défaut et s’applique à la plupart des scénarios de déploiement. L’extension de votre répertoire local dans le cloud n’est plus qu’à quelques clics.

Avant de commencer l’installation d’Azure AD Connect, veillez à [télécharger Azure AD Connect](https://go.microsoft.com/fwlink/?LinkId=615771) et effectuer les étapes préalables décrites dans [Azure AD Connect : matériel et prérequis](how-to-connect-install-prerequisites.md).

Si la configuration rapide ne correspond pas à votre topologie, consultez la [documentation connexe](#related-documentation) pour d’autres scénarios.

## <a name="express-installation-of-azure-ad-connect"></a>Installation rapide pour Azure AD Connect
Vous pouvez voir ces étapes en action dans la section [vidéos](#videos) .

1. Connectez-vous en tant qu’administrateur local au serveur sur lequel vous souhaitez installer Azure AD Connect. Il doit s’agir du serveur que vous choisissez comme serveur de synchronisation.
2. Accédez à **AzureADConnect.msi**et double-cliquez sur ce fichier.
3. Sur l'écran d’accueil, sélectionnez la case pour accepter les termes du contrat de licence et cliquez sur **Continuer**.  
4. Sur l'écran Paramètres Express, cliquez sur **Utiliser les paramètres Express**.  
   ![Bienvenue dans Azure AD Connect](./media/how-to-connect-install-express/express.png)
5. Sur l’écran Connexion à Azure AD, entrez le nom d’utilisateur et un mot de passe d’administrateur général pour votre instance Azure AD. Cliquez sur **Suivant**.  
   ![Connexion à Azure AD](./media/how-to-connect-install-express/connectaad.png)  
   Si vous recevez une erreur et que vous avez des problèmes de connectivité, consultez [Résoudre les problèmes de connectivité liés à Azure AD Connect](tshoot-connect-connectivity.md).
6. Sur l’écran Connexion à AD DS, entrez le nom d’utilisateur et le mot de passe d’un compte d’administrateur d’entreprise. Vous pouvez saisir la partie domaine au format NetBios ou nom de domaine complet, c’est-à-dire FABRIKAM\administrator ou fabrikam.com\administrator. Cliquez sur **Suivant**.  
   ![Connexion à AD DS](./media/how-to-connect-install-express/connectad.png)
7. La page [**Configuration de connexion Azure AD**](plan-connect-user-signin.md#azure-ad-sign-in-configuration) s’affiche uniquement si vous n’avez pas effectué l’étape de [vérification de vos noms de domaines](../fundamentals/add-custom-domain.md) dans les [conditions préalables](how-to-connect-install-prerequisites.md).
   ![Domaines non vérifiés](./media/how-to-connect-install-express/unverifieddomain.png)  
   Si vous voyez cette page, passez en revue chaque domaine marqué **Non ajouté** et **Non vérifié**. Assurez-vous que les domaines que vous utilisez ont été vérifiés dans Azure AD. Cliquez sur le symbole d’actualisation dès que vous avez vérifié vos domaines.
8. Sur l’écran Prêt à configurer, cliquez sur **Installer**.
   * Dans la page Prêt à configurer, vous pouvez éventuellement décocher la case **Démarrer le processus de synchronisation dès que la configuration est terminée** . Vous devez décocher cette case si vous souhaitez effectuer une configuration supplémentaire, tel que le [filtrage](how-to-connect-sync-configure-filtering.md). Si vous désélectionnez cette option, l’Assistant configure la synchronisation, mais laisse le planificateur désactivé. Il n’est pas exécuté jusqu’à ce que vous l’activiez manuellement en [exécutant de nouveau l’Assistant d’installation](how-to-connect-installation-wizard.md).
   * En laissant la case **Démarrer le processus de synchronisation dès que la configuration est terminée** cochée, cela déclenche immédiatement une synchronisation complète sur Azure AD de tous les utilisateurs, groupes et contacts.
   * Si Exchange est présent dans votre répertoire Active Directory local, vous avez également la possibilité d’activer le [**déploiement Exchange hybride**](/exchange/exchange-hybrid). Activez cette option si vous envisagez de disposer simultanément de boîtes aux lettres Exchange dans le cloud et en local.
     ![Azure AD Connect prêt à configurer](./media/how-to-connect-install-express/readytoconfigure.png)
9. Une fois l’installation terminée, cliquez sur **Quitter**.
10. Une fois l’installation terminée, déconnectez-vous puis reconnectez-vous à Windows avant d’utiliser le gestionnaire Synchronization Service Manager ou l’éditeur de règles de synchronisation.

## <a name="videos"></a>Videos
Pour une vidéo sur l’utilisation de l’installation rapide, voir :

> [!VIDEO https://channel9.msdn.com/Series/Azure-Active-Directory-Videos-Demos/Azure-Active-Directory-Connect-Express-Settings/player]
>
>

## <a name="next-steps"></a>Étapes suivantes
Azure AD Connect étant installé, vous pouvez passer à [Vérification de l’installation et affectation des licences](how-to-connect-post-installation.md).

Pour en savoir plus sur ces fonctionnalités activées lors de l’installation, consultez les pages : [Mise à niveau automatique](how-to-connect-install-automatic-upgrade.md), [Prévention des suppressions accidentelles](how-to-connect-sync-feature-prevent-accidental-deletes.md) et [Azure AD Connect Health](how-to-connect-health-sync.md).

Pour en savoir plus sur ces sujets courants, consultez l’article [Planificateur Azure AD Connect Sync](how-to-connect-sync-feature-scheduler.md).

En savoir plus sur l’ [intégration de vos identités locales avec Azure Active Directory](whatis-hybrid-identity.md).

## <a name="related-documentation"></a>documentation connexe

| Rubrique | Lien |
| --- | --- |
| Présentation d’Azure AD Connect | [Intégrer vos répertoires locaux à Azure Active Directory](whatis-hybrid-identity.md)
| Installation à l’aide des paramètres personnalisés | [Installation personnalisée d’Azure AD Connect](how-to-connect-install-custom.md) |
| Effectuer une mise à niveau à partir de DirSync | [Effectuer une mise à niveau à partir de l’outil de synchronisation Azure AD (DirSync)](how-to-dirsync-upgrade-get-started.md)|
| Comptes utilisés pour l’installation | [Autorisations et informations d’identification Azure AD Connect](reference-connect-accounts-permissions.md) |