---
title: Questions fréquentes (FAQ) sur les accélérateurs de solution IoT - Azure | Microsoft Docs
description: Cet article répond aux questions fréquentes sur les accélérateurs de solution IoT. Il comporte les liens des référentiels GitHub.
author: dominicbetts
manager: timlt
ms.service: iot-accelerators
services: iot-accelerators
ms.topic: conceptual
ms.date: 02/15/2018
ms.author: dobett
ms.openlocfilehash: 1fd2b8461bd66c826dc4890c331b740c4703f896
ms.sourcegitcommit: 80c1056113a9d65b6db69c06ca79fa531b9e3a00
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96903987"
---
# <a name="frequently-asked-questions-for-iot-solution-accelerators"></a>Questions fréquentes (FAQ) sur les accélérateurs de solution IoT

Consultez également les [questions fréquentes (FAQ) spécifiques à l’usine connectée](iot-accelerators-faq-cf.md).

### <a name="where-can-i-find-the-source-code-for-the-solution-accelerators"></a>Où trouver le code source des accélérateurs de solution ?

Le code source est stocké dans les référentiels GitHub suivants :

* [Accélérateur de solution Usine connectée](https://github.com/Azure/azure-iot-connected-factory)
* [Accélérateur de solution de simulation d’appareil](https://github.com/Azure/device-simulation-dotnet)

### <a name="where-can-i-find-the-remote-monitoring-and-predictive-maintenance-solution-accelerators"></a>Où trouver les accélérateurs de solutions de maintenance prédictive et de supervision à distance ?

Depuis le 10 décembre 2020, les accélérateurs de maintenance prédictive et de supervision à distance ont été supprimés du site des [accélérateurs de solutions Azure IoT](https://www.azureiotsolutions.com/Accelerators) et ne sont plus disponibles pour les nouveaux déploiements. Les dépôts GitHub pour les deux accélérateurs ont été archivés. Le code est toujours disponible et accessible à tous, mais les dépôts ne prennent pas de nouvelles contributions.

### <a name="what-happens-to-my-existing-remote-monitoring-and-predictive-maintenance-deployments"></a>Que deviennent mes déploiements de maintenance prédictive et de supervision à distance existants ?

Les déploiements existants ne sont pas affectés par la suppression des accélérateurs de solutions de maintenance prédictive et de supervision à distance et continuent de fonctionner. Les dépôts dupliqués ne sont pas non plus affectés. Les dépôts principaux sur GitHub ont été archivés.

### <a name="how-do-i-deploy-device-simulation-solution-accelerator"></a>Comment déployer l’accélérateur de solution de simulation d’appareil ?

Pour déployer l’accélérateur de solution de simulation d’appareil, reportez-vous au dépôt GitHub de la [simulation d’appareil](https://github.com/Azure/device-simulation-dotnet/blob/master/README.md).

### <a name="where-can-i-find-information-about-the-removed-solution-accelerators"></a>Où trouver des informations sur les accélérateurs de solutions supprimés ?

Consultez les pages suivantes sur le site des versions précédentes :

* [Surveillance à distance](/previous-versions/azure/iot-accelerators/about-iot-accelerators)
* [Maintenance prédictive](/previous-versions/azure/iot-accelerators/about-iot-accelerators)

### <a name="what-sdks-can-i-use-to-develop-device-clients-for-the-solution-accelerators"></a>Quels SDK puis-je utiliser afin de développer des clients d’appareils pour les accélérateurs de solution ?

Vous trouverez des liens vers les kits de développement logiciel des appareils IoT en différentes langues (C, .NET, Java, Node.js, Python) dans les référentiels GitHub [Kits de développement logiciel Microsoft Azure IoT](https://github.com/Azure/azure-iot-sdks).

Si vous utilisez l’appareil DevKit, vous pouvez trouver des ressources et des exemples dans le dépôt GitHub [SDK IoT DevKit](https://github.com/Microsoft/devkit-sdk).

### <a name="im-a-service-administrator-and-id-like-to-change-the-directory-mapping-between-my-subscription-and-a-specific-azure-ad-tenant-how-do-i-complete-this-task"></a>Je suis administrateur de service et je souhaite changer le mappage d’annuaire entre mon abonnement et un locataire Azure AD spécifique. Comment mener à bien cette tâche ?

Voir [Comment ajouter un abonnement existant à votre répertoire Azure AD](../active-directory/fundamentals/active-directory-how-subscriptions-associated-directory.md#to-associate-an-existing-subscription-to-your-azure-ad-directory)

### <a name="i-want-to-change-a-service-administrator-or-co-administrator-when-logged-in-with-an-organizational-account"></a>Je souhaite modifier la fonctionnalité administrateur de service ou coadministrateur lors d’une connexion avec un compte de société

Voir l’article de support [Modification de la fonctionnalité Administrateur de service et Coadministrateur lors d’une connexion avec un compte de société](https://azure.microsoft.com/support/changing-service-admin-and-co-admin).

### <a name="why-am-i-seeing-this-error-your-account-does-not-have-the-proper-permissions-to-create-a-solution-please-check-with-your-account-administrator-or-try-with-a-different-account"></a>Pourquoi est-ce que je reçois cette erreur ? « Votre compte n’a pas les autorisations suffisantes pour créer une solution. Veuillez contacter votre administrateur de compte ou essayer avec un autre compte. »

Examinez le schéma suivant pour obtenir des conseils :

![Organigramme des autorisations](media/iot-accelerators-faq/flowchart.png)

> [!NOTE]
> Si l’erreur persiste après votre validation en tant qu’administrateur global du locataire Azure AD et que co-administrateur de l’abonnement, demandez à votre administrateur de compte de supprimer l’utilisateur et de réattribuer les autorisations nécessaires dans l’ordre suivant : tout d’abord, ajoutez l’utilisateur en tant qu’administrateur global, puis ajoutez un utilisateur en tant que coadministrateur sur l’abonnement Azure. Si les problèmes persistent, contactez le service [Aide et support](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade).

### <a name="why-am-i-seeing-this-error-when-i-have-an-azure-subscription-an-azure-subscription-is-required-to-create-pre-configured-solutions-you-can-create-a-free-trial-account-in-just-a-couple-of-minutes"></a>Pourquoi affiche-t-il cette erreur alors que j’ai un abonnement Azure ? « Vous devez avoir un abonnement Azure pour créer des solutions préconfigurées. Vous pouvez créer un compte d'essai gratuit en quelques minutes seulement. »

Si vous êtes sûr de disposer d’un abonnement Azure, validez le mappage de locataire pour votre abonnement et vérifiez que c’est le locataire correct qui est sélectionné dans la liste déroulante. Si vous avez validé que le locataire est correct, suivez le schéma précédent et validez le mappage de votre abonnement et ce locataire Azure AD.

### <a name="whats-the-difference-between-deleting-a-resource-group-in-the-azure-portal-and-clicking-delete-on-a-solution-accelerator-in-azureiotsolutionscom"></a>Quelle est la différence entre supprimer un groupe de ressources dans le portail Azure et cliquer sur Supprimer pour un accélérateur de solution dans azureiotsolutions.com ?

* Si vous supprimez l’accélérateur de solution dans [azureiotsolutions.com](https://www.azureiotsolutions.com/), vous supprimez toutes les ressources qui ont été déployées lors de la création de cet accélérateur. Si vous avez ajouté des ressources supplémentaires au groupe de ressources, elles sont également supprimées.
* Si vous supprimez le groupe de ressources sur le [portail Azure](https://portal.azure.com), vous supprimez uniquement les ressources de ce groupe de ressources. Vous devez également supprimer l’application Azure Active Directory associée à l’accélérateur de solution.

### <a name="can-i-continue-to-leverage-my-existing-investments-in-azure-iot-solution-accelerators"></a>Puis-je continuer à tirer parti de mes investissements existants dans les accélérateurs de solution Azure IoT ?

Oui. Toute solution qui existe aujourd’hui continue à fonctionner dans votre abonnement Azure et le code source reste disponible dans GitHub.

### <a name="how-many-iot-hub-instances-can-i-provision-in-a-subscription"></a>Combien d’instances d’IoT Hub puis-je configurer dans un abonnement ?

Par défaut, vous pouvez approvisionner [10 instances IoT Hub par abonnement](../azure-resource-manager/management/azure-subscription-service-limits.md#iot-hub-limits). Vous pouvez créer un [ticket de support Azure](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade) pour augmenter cette limite. Par conséquent, étant donné que chaque accélérateur de solution provisionne un nouvel IoT Hub, vous ne pouvez configurer que 10 accélérateurs de solution au maximum dans un abonnement.

### <a name="how-many-azure-cosmos-db-instances-can-i-provision-in-a-subscription"></a>Combien d’instances d’Azure Cosmos DB puis-je configurer dans un abonnement ?

Cinquante. Vous pouvez créer un [ticket de support Azure](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade) pour augmenter cette limite, mais par défaut, vous ne pouvez approvisionner que 50 instances de Cosmos DB par abonnement.

### <a name="can-i-create-a-solution-accelerator-if-i-have-microsoft-azure-for-dreamspark"></a>Puis-je créer un accélérateur de solution si je dispose de Microsoft Azure pour DreamSpark ?

> [!NOTE]
> Microsoft Azure pour DreamSpark s’appelle désormais Microsoft Imagine pour les étudiants.

Il n’est pas possible de créer un accélérateur de solution avec un compte [Microsoft Azure pour DreamSpark](https://azure.microsoft.com/pricing/member-offers/imagine/). Vous pouvez toutefois créer en quelques minutes un [compte d’essai gratuit pour Azure](https://azure.microsoft.com/free/), que vous pouvez utiliser pour créer un accélérateur de solution.

### <a name="how-do-i-delete-an-azure-ad-tenant"></a>Comment supprimer un client Azure AD ?

Consultez le billet de blog d’Eric Golpe, [Procédure pas à pas pour la suppression d’un client Azure AD](/archive/blogs/ericgolpe/walkthrough-of-deleting-an-azure-ad-tenant).

### <a name="next-steps"></a>Étapes suivantes

Vous pouvez également explorer certaines des autres fonctionnalités des accélérateurs de solution IoT :

* [Déployer l’accélérateur de solution Usine connectée](quickstart-connected-factory-deploy.md)
* [Sécurisation de l’Internet des objets de bout en bout](../iot-fundamentals/iot-security-ground-up.md)