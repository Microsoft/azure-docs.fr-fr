---
title: Résoudre les problèmes de connexion à l’abonnement Azure
description: Aide à résoudre les problèmes de connexion au portail Azure ou au Centre des comptes Azure.
services: cost-management-billing
author: v-miegge
manager: dcscontentpm
tags: billing
ms.service: cost-management-billing
ms.subservice: billing
ms.topic: troubleshooting
ms.date: 08/20/2020
ms.author: v-miegge
ms.openlocfilehash: 2e9b14fa264f3286134913e3c279c4400ce5bcc3
ms.sourcegitcommit: 33368ca1684106cb0e215e3280b828b54f7e73e8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92132327"
---
# <a name="troubleshoot-azure-subscription-sign-in-issues"></a>Résoudre les problèmes de connexion à l’abonnement Azure

Ce guide aide à résoudre les problèmes de connexion au portail Azure ou au Centre des comptes Azure.

> [!NOTE]
> Si vous rencontrez des problèmes pour vous inscrire à un nouveau compte Azure, consultez [Résoudre les problèmes d’inscription aux abonnements Azure](./troubleshoot-azure-sign-up.md).

## <a name="page-hangs-in-the-loading-status"></a>La page se bloque à l’état de chargement

Si la page de votre navigateur Internet se bloque, essayez chacune des étapes suivantes jusqu’à ce que vous puissiez accéder au Portail Azure.

- Actualisez la page.
- Utilisez un autre navigateur Internet.
- Utilisez le mode de navigation privée de votre navigateur :

   - **Edge :** ouvrez **Paramètres** (les trois points en regard de l’image de votre profil), sélectionnez **Nouvelle fenêtre InPrivate**, puis recherchez le [portail Azure](https://portal.azure.com/) ou le [Centre des comptes Azure](https://account.azure.com/Subscriptions) afin de vous y connecter. 
   - **Chrome :** choisissez le mode **Navigation privée**.
   - **Safari :** choisissez **Fichier**, puis **Nouvelle fenêtre privée**.

- Effacez la mémoire cache et supprimez les cookies Internet :

   - **Edge :** ouvrez **Paramètres** et sélectionnez **Confidentialité et services**. Suivez les étapes sous **Effacer les données de navigation**. Vérifiez que les cases **Historique de navigation**, **Historique des téléchargements** et **Images et fichiers mis en cache** sont cochées, puis sélectionnez **Supprimer**.
   - **Chrome :** choisissez **Paramètres** et sélectionnez **Effacer les données de navigation** sous **Confidentialité et sécurité**.

## <a name="you-are-automatically-signed-in-as-a-different-user"></a>Vous êtes automatiquement connecté comme un autre utilisateur

Ce problème peut se produire si vous utilisez plusieurs comptes d’utilisateur dans un navigateur Internet.

Pour résoudre le problème, essayez l’une des méthodes suivantes :

- Effacez la mémoire cache et supprimez les cookies Internet.

   - **Edge :** ouvrez **Paramètres** et sélectionnez **Confidentialité et services**. Suivez les étapes sous **Effacer les données de navigation**. Vérifiez que les cases **Historique de navigation**, **Historique des téléchargements**, **Cookies** et **Images et fichiers mis en cache** sont cochées, puis sélectionnez **Supprimer**.
   - **Chrome :** choisissez **Paramètres** et sélectionnez **Effacer les données de navigation** sous **Confidentialité et sécurité**.
- Rétablissez les valeurs par défaut des paramètres de votre navigateur.
- Utilisez le mode de navigation privée de votre navigateur. 
   - **Edge :** ouvrez **Paramètres** (les trois points en regard de l’image de votre profil), sélectionnez **Nouvelle fenêtre InPrivate**, puis recherchez le [portail Azure](https://portal.azure.com/) ou le [Centre des comptes Azure](https://account.azure.com/Subscriptions) afin de vous y connecter. 
   - **Chrome :** choisissez le mode **Navigation privée**.
   - **Safari :** choisissez **Fichier**, puis **Nouvelle fenêtre privée**.

## <a name="i-can-sign-in-but-i-see-the-error-no-subscriptions-found"></a>Je peux me connecter mais l’erreur Aucun abonnement trouvé s’affiche

Ce problème se produit si vous avez sélectionné le mauvais annuaire ou si votre compte ne dispose pas des autorisations suffisantes.

**Scénario 1 :** Vous recevez l’erreur quand vous vous connectez au [portail Azure](https://portal.azure.com/)

Pour résoudre ce problème :

- Vérifiez que le répertoire Azure correct est sélectionné en sélectionnant votre compte dans l’angle supérieur droit.
- Si le répertoire Azure correct est sélectionné, mais que vous recevez néanmoins le messager d’erreur, [faites ajouter votre compte en tant que Propriétaire](./add-change-subscription-administrator.md).

**Scénario 2 :** Vous recevez l’erreur quand vous vous connectez au [Centre des comptes Azure](https://account.windowsazure.com/Subscriptions)

Vérifiez que le compte utilisé correspond à l’administrateur du compte. Pour vérifier qui est l’administrateur du compte, procédez comme suit :

1.  Connectez-vous à la [vue Abonnements dans le portail Azure](https://portal.azure.com/#blade/Microsoft_Azure_Billing/SubscriptionsBlade).
1.  Sélectionnez l’abonnement que vous souhaitez vérifier, puis choisissez **Paramètres**.
1.  Sélectionner **Propriétés**. L’administrateur de compte de l’abonnement s’affiche dans la zone **Administrateur de compte** .

## <a name="additional-help-resources"></a>Ressources d'aide supplémentaires

Autres articles sur la résolution des problèmes liés à la facturation et aux abonnements Azure

- [Carte refusée](./troubleshoot-declined-card.md)
- [Problèmes de souscription à l’abonnement](./troubleshoot-azure-sign-up.md)
- [Aucun abonnement trouvé](./no-subscriptions-found.md)
- [Désactivation de l’affichage des coûts d’entreprise](./enterprise-mgmt-grp-troubleshoot-cost-view.md)
- [Documentation sur la facturation Azure](../index.yml)

## <a name="contact-us-for-help"></a>Contactez-nous pour obtenir de l'aide

Si vous avez des questions ou besoin d’aide, [créez une demande de support](https://ms.portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest).