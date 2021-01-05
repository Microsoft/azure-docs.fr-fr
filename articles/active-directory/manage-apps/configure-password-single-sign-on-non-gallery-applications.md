---
title: Comprendre l’authentification unique par mot de passe pour les applications dans Azure Active Directory
description: Comprendre l’authentification unique par mot de passe pour les applications dans Azure Active Directory
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: conceptual
ms.date: 07/29/2020
ms.author: kenwith
ms.openlocfilehash: 1b647e15d3fc99a7f15fbc24e2b6050fdfdc6e93
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94654709"
---
# <a name="understand-password-based-single-sign-on"></a>Comprendre l’authentification unique par mot de passe

Avec la [série de guides de démarrage rapide](view-applications-portal.md) sur la gestion des applications, vous avez appris à utiliser Azure AD comme fournisseur d’identité (IdP) pour une application. Dans le guide de démarrage rapide, vous configurez l’authentification unique par SAML ou OIDC. Une autre option est l’authentification unique par mot de passe. Cet article décrit plus en détail l’option d’authentification unique par mot de passe. 

Cette option est disponible pour tout site web doté d’une page de connexion HTML. L’authentification unique par mot de passe est également connue sous le nom d’archivage des mots de passe. L’authentification unique par mot de passe vous permet de gérer l’accès utilisateur et les mots de passe pour les applications web qui ne prennent pas en charge la fédération d’identité. Elle est également utile lorsque plusieurs utilisateurs doivent partager un seul compte, par exemple les comptes d’applications de réseaux sociaux de votre organisation.

L’authentification unique par mot de passe (SSO) est un excellent moyen pour commencer à intégrer rapidement des applications dans Azure AD et vous permet d’effectuer les opérations suivantes :

- activer l’authentification unique pour vos utilisateurs par le stockage et la relecture en toute sécurité des noms d’utilisateur et mots de passe ;

- prendre en charge les applications qui requièrent plusieurs champs de connexion pour les applications qui demandent d’autres champs en plus du nom d’utilisateur et du mot de passe pour la connexion ;

- personnaliser les étiquettes des champs de nom d’utilisateur et de mot de passe que vos utilisateurs voient dans [Mes applications](../user-help/my-apps-portal-end-user-access.md) quand ils entrent leurs informations d’identification ;

- autoriser vos utilisateurs à fournir leurs propres noms d’utilisateur et mots de passe de tout compte d’application existant qu’ils saisissent manuellement ;

- autoriser un membre du groupe d’entreprise à spécifier les noms d’utilisateur et mots de passe affectés à un utilisateur à l’aide de la fonctionnalité [Accès aux applications en libre-service](./manage-self-service-access.md) ;

-   autoriser un administrateur à spécifier un nom d’utilisateur et un mot de passe à utiliser par des individus ou des groupes lors de la connexion à l’application à l’aide de la fonctionnalité Mettre à jour les informations d’identification. 

## <a name="before-you-begin"></a>Avant de commencer

L’utilisation d’Azure AD comme fournisseur d’identité (IdP) et la configuration de l’authentification unique peuvent être simples ou complexes selon l’application utilisée. Certaines applications peuvent être configurées en quelques actions seulement. D’autres nécessitent une configuration approfondie. Pour être efficace rapidement, suivez la [série de guides de démarrage rapide](view-applications-portal.md) sur la gestion des applications. Si l’application que vous ajoutez est simple, vous n’avez probablement pas besoin de lire cet article. Si l’application que vous ajoutez demande une configuration personnalisée et que vous devez utiliser l’authentification unique par mot de passe, cet article vous concerne.

> [!IMPORTANT] 
> Dans certains cas, l’option **Authentification unique** n’est pas accessible pour une application sous **Applications d’entreprise**. 
>
> Si l’application a été inscrite à l’aide d’**Inscriptions d’applications**, la fonctionnalité d’authentification unique est configurée pour utiliser OIDC OAuth par défaut. Dans ce cas, l’option **Authentification unique** ne s’affiche pas dans le volet de navigation sous **Applications d’entreprise**. Quand vous utilisez **inscriptions d’applications** pour ajouter votre application personnalisée, vous configurez les options dans le fichier du manifeste. Pour en savoir plus sur le fichier manifeste, consultez [Manifeste d’application Azure Active Directory](../develop/reference-app-manifest.md). Pour en savoir plus sur les standards SSO, consultez [Authentification et autorisation avec la plateforme d’identités Microsoft](../develop/authentication-vs-authorization.md#authentication-and-authorization-using-microsoft-identity-platform). 
>
> D’autres scénarios dans lesquels **l’authentification unique** sera absente de la navigation incluent les cas où une application est hébergée dans un autre locataire ou si votre compte ne dispose pas des autorisations requises (administrateur général, administrateur d’application Cloud, administrateur d’application ou propriétaire du principal de service). Les autorisations peuvent également être à l’origine d’un scénario dans lequel vous pouvez ouvrir **l’authentification unique**, mais vous ne pourrez pas l’enregistrer. En savoir plus sur les rôles d’administration d’Azure AD voir (https://docs.microsoft.com/azure/active-directory/users-groups-roles/directory-assign-admin-roles).


## <a name="basic-configuration"></a>Configuration de base

Dans la [série d’articles de démarrage rapide](view-applications-portal.md), vous avez appris à ajouter une application à votre locataire, ce qui permet à Azure AD de savoir qu’il est utilisé comme fournisseur d’identité (IdP) pour l’application. Certaines applications sont déjà préconfigurées et s’affichent dans la galerie d’Azure AD. Les autres applications ne sont pas dans la galerie : vous devez créer une application générique et la configurer manuellement. Selon l’application, l’option d’authentification unique par mot de passe peut ne pas être disponible. Si vous ne voyez pas l’option par mot de passe répertoriée sur la page d’authentification unique de l’application, cela signifie qu’elle n’est pas disponible.

> [!IMPORTANT]
> L’extension de navigateur Mes applications est requise pour l’authentification unique par mot de passe. Pour plus d’informations, consultez [Planifier un déploiement Mes applications](access-panel-deployment-plan.md).

La page de configuration de l’authentification unique par mot de passe est simple. Elle comprend uniquement l’URL de la page d’authentification utilisée par l’application. Il doit s’agir de la page qui contient le champ d’entrée du nom d’utilisateur.

Une fois que vous avez entré l’URL, sélectionnez **Enregistrer**. Azure AD analyse le code HTML de la page de connexion pour les champs d’entrée du nom d’utilisateur et du mot de passe. Si la tentative réussit, vous avez terminé.
 
La prochaine étape consiste à [affecter des utilisateurs et des groupes à l’application](./assign-user-or-group-access-portal.md). Après avoir affecté des utilisateurs et des groupes, vous pouvez fournir des informations d’identification à utiliser pour un utilisateur lorsqu’il se connecte à l’application. Sélectionnez **Utilisateurs et groupes**, cochez la case pour la ligne de l’utilisateur ou du groupe, puis sélectionnez **Mettre à jour les informations d’identification**. Enfin, entrez le nom d’utilisateur et le mot de passe à utiliser pour l’utilisateur ou le groupe. Si vous ne le faites pas, les utilisateurs devront entrer les informations d’identification eux-mêmes lors du lancement.
 

## <a name="manual-configuration"></a>Configuration manuelle

Si la tentative d'analyse Azure AD échoue, vous pouvez configurer l’authentification manuellement.

1. Sous **Configuration \<application name>** , sélectionnez **Configurer les paramètres d’authentification unique par mot de passe \<application name>** pour afficher la page **Configurer l’authentification**. 

2. Sélectionnez **Détecter manuellement les champs de connexion**. Des instructions supplémentaires décrivant la détection manuelle des champs de connexion s’affichent.

   ![Configuration manuelle de l’authentification unique par mot de passe](./media/configure-password-single-sign-on/password-configure-sign-on.png)
3. Sélectionnez **Capturer les champs de connexion**. Une page d’état de la capture s’ouvre dans un nouvel onglet affichant le message **Capture des métadonnées en cours**.

4. Si la mention **Extension Mes applications requise** s’affiche dans un nouvel onglet, sélectionnez **Installer maintenant** pour installer l’extension de navigateur **Extension de connexion sécurisée à Mes applications**. (Cette extension de navigateur est compatible avec Microsoft Edge, Chrome ou Firefox.) Ensuite, installez, lancez et activez l’extension, puis actualisez la page d’état de la capture.

   L’extension de navigateur ouvre ensuite un nouvel onglet qui affiche l’URL entrée.
5. Dans l’onglet contenant l’URL saisie, suivez le processus de connexion. Renseignez les champs de nom d’utilisateur et de mot de passe et essayez de vous connecter. (Vous n’êtes pas obligé de fournir le mot de passe correct.)

   Un message vous invite à enregistrer les champs de connexion capturés.
6. Sélectionnez **OK**. L’extension de navigateur met à jour la page d’état de la capture avec le message **Les métadonnées ont été mises à jour pour l’application**. L'onglet du navigateur se ferme.

7. Dans la page Azure AD **Configurer l’authentification**, sélectionnez **J’ai pu me connecter à l’application**.

8. Sélectionnez **OK**.

## <a name="next-steps"></a>Étapes suivantes

- [Affecter des utilisateurs et des groupes à l’application](./assign-user-or-group-access-portal.md)
- [Configurer le provisionnement automatique d’un compte utilisateur](../app-provisioning/configure-automatic-user-provisioning-portal.md)