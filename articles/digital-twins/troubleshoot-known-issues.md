---
title: Problèmes connus - Azure Digital Twins
description: Recevez de l’aide pour la reconnaissance et l’atténuation des problèmes connus avec Azure Digital Twins.
author: baanders
ms.author: baanders
ms.topic: troubleshooting
ms.service: digital-twins
ms.date: 07/14/2020
ms.openlocfilehash: 549e1808a3b449f7d29b968cde76ef29391880b3
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93100612"
---
# <a name="known-issues-in-azure-digital-twins"></a>Problèmes connus dans Azure Digital Twins

Cet article fournit des informations sur les problèmes connus associés à Azure Digital Twins.

## <a name="400-client-error-bad-request-in-cloud-shell"></a>« Erreur 400 du client : Requête incorrecte » dans Cloud Shell

Les commandes dans Cloud Shell qui s’exécutent sur *https://shell.azure.com* peuvent échouer par intermittence avec l’erreur « Erreur 400 du client : Requête incorrecte pour l’URL : http://localhost:50342/oauth2/token » suivie par l’arborescence complète des appels de procédure.

Pour Azure Digital Twins, cela affecte les groupes de commandes suivants :
* `az dt route`
* `az dt model`
* `az dt twin`

### <a name="troubleshooting-steps"></a>Étapes de dépannage

Cela peut être résolu en réexécutant la commande `az login` dans Cloud Shell et en suivant les étapes de connexion suivantes. Après cela, vous devriez pouvoir réexécuter la commande.

Autrement, vous pouvez ouvrir le volet Cloud Shell dans le portail Azure et effectuer votre travail Cloud Shell à partir de là :

:::image type="content" source="media/includes/portal-cloud-shell.png" alt-text="Vue du portail Azure avec l’icône « Cloud Shell » mise en évidence, et l’éditeur Cloud Shell s’affichant dans la partie inférieure de la fenêtre du portail" lightbox="media/includes/portal-cloud-shell.png":::

Une dernière solution consiste à [installer Azure CLI](/cli/azure/install-azure-cli?view=azure-cli-latest&preserve-view=true) sur votre ordinateur afin de pouvoir exécuter les commandes Azure CLI localement. L’interface CLI locale ne rencontre pas ce problème.

### <a name="possible-causes"></a>Causes possibles

Voici le résultat d’un problème connu dans Cloud Shell : [*L’obtention d’un jeton à partir de Cloud Shell échoue par intermittence avec l’Erreur 400 du client : Requête incorrecte*](https://github.com/Azure/azure-cli/issues/11749).

Cela pose un problème avec les jetons d’authentification d’instance Azure Digital Twins et l’authentification par défaut basée sur l’[identité managée](../active-directory/managed-identities-azure-resources/overview.md) de Cloud Shell. L’étape de résolution des problèmes consistant à exécuter `az login` vous déconnecte de l’authentification basée sur l’identité managée, ce qui vous permet de contourner ce problème.

Cela n’affecte pas les commandes Azure Digital Twins des groupes de commandes `az dt` ou `az dt endpoint`, car elles utilisent un autre type de jeton d’authentification (basé sur ARM) qui n’a pas de problème avec l’authentification par identité managée de Cloud Shell.

## <a name="missing-role-assignment-after-scripted-setup"></a>Attribution de rôle manquante après l’installation par script

Certains utilisateurs peuvent rencontrer des problèmes dans la partie « Attribution des rôles » de [*Guide pratique : Configurer une instance et l’authentification (procédure scriptée)*](how-to-set-up-instance-scripted.md). Le script n’indique pas d’échec, mais le rôle *Propriétaire de données Azure Digital Twins* n’est pas correctement attribué à l’utilisateur, et ce problème aura un impact sur la capacité à créer d’autres ressources ultérieurement.

[!INCLUDE [digital-twins-role-rename-note.md](../../includes/digital-twins-role-rename-note.md)]

Pour déterminer si votre attribution de rôle a été correctement configurée après l’exécution du script, suivez les instructions de la section [*Vérifier l’attribution du rôle utilisateur*](how-to-set-up-instance-scripted.md#verify-user-role-assignment) de l’article sur l’installation. Si votre utilisateur n’est pas associé à ce rôle, ce problème vous concerne.

### <a name="troubleshooting-steps"></a>Étapes de dépannage

Pour résoudre ce problème, vous pouvez configurer manuellement votre attribution de rôle à l’aide de l’interface CLI ou du portail Azure. 

Suivez ces instructions :
* [INTERFACE DE LIGNE DE COMMANDE](how-to-set-up-instance-cli.md#set-up-user-access-permissions)
* [portal](how-to-set-up-instance-portal.md#set-up-user-access-permissions)

### <a name="possible-causes"></a>Causes possibles

Pour les utilisateurs connectés avec un [compte Microsoft (MSA)](https://account.microsoft.com/account) personnel, l’ID principal d’utilisateur qui vous identifie dans des commandes comme celle-ci peut être différent de votre e-mail de connexion d’utilisateur, ce qui rend difficile la découverte et l’utilisation du script pour attribuer le rôle correctement.

## <a name="issue-with-interactive-browser-authentication"></a>Problème avec l’authentification interactive du navigateur

Lorsque vous écrivez du code d’authentification dans vos applications Azure Digital Twins à l’aide de la version **1.2.0** de la bibliothèque **[Azure.Identity](/dotnet/api/azure.identity?view=azure-dotnet&preserve-view=true)** , vous pouvez rencontrer des problèmes avec la méthode [InteractiveBrowserCredential](/dotnet/api/azure.identity.interactivebrowsercredential?view=azure-dotnet&preserve-view=true).

Il ne s’agit pas de la version la plus récente de la bibliothèque. La version la plus récente est **1.2.2**.

La méthode concernée est utilisée dans les articles suivants : 
* [*Tutoriel : Coder une application cliente*](tutorial-code.md)
* [*Guide pratique : Écrire le code d’authentification de l’application*](how-to-authenticate-client.md)
* [*Guide pratique : Utiliser les API et les Kits de développement logiciel (SDK) Azure Digital Twins*](how-to-use-apis-sdks.md)

Le problème comprend une réponse d’erreur « Azure.Identity.AuthenticationFailedException » lors de la tentative d’authentification dans une fenêtre du navigateur. La fenêtre du navigateur peut ne pas démarrer complètement ou sembler réussir à authentifier l’utilisateur, alors que l’application cliente échoue toujours avec l’erreur.

### <a name="troubleshooting-steps"></a>Étapes de dépannage

Pour résoudre ce problème, mettez à jour vos applications pour utiliser la version  **1.2.2** d’`Azure.Identity`. Avec cette version de la bibliothèque, le navigateur doit se charger et s’authentifier comme prévu.

### <a name="possible-causes"></a>Causes possibles

Cela est lié à un problème ouvert avec la dernière version de la bibliothèque `Azure.Identity` (version  **1.2.0** ) : [*Échec de l’authentification lors de l’utilisation de InteractiveBrowserCredential*](https://github.com/Azure/azure-sdk-for-net/issues/13940).

Vous rencontrez ce problème si vous utilisez la version **1.2.0** dans votre application Azure Digital Twins ou si vous ajoutez la bibliothèque à votre projet sans spécifier de version (car la version la plus récente est également utilisée par défaut).

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur la sécurité et les autorisations sur Azure Digital Twins :
* [*Concepts : Sécurité pour les solutions Azure Digital Twins*](concepts-security.md)