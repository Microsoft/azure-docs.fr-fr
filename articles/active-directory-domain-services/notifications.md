---
title: Notifications par e-mail pour Azure AD Domain Services | Microsoft Docs
description: Découvrez comment configurer des notifications par e-mail pour être alerté des problèmes qui se produisent dans un domaine managé Azure Active Directory Domain Services
services: active-directory-ds
author: justinha
manager: daveba
ms.assetid: b9af1792-0b7f-4f3e-827a-9426cdb33ba6
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.topic: how-to
ms.date: 07/06/2020
ms.author: justinha
ms.openlocfilehash: 4242092af86f8bc06ef9c9eba068ce07725de01f
ms.sourcegitcommit: 8192034867ee1fd3925c4a48d890f140ca3918ce
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/05/2020
ms.locfileid: "96619162"
---
# <a name="configure-email-notifications-for-issues-in-azure-active-directory-domain-services"></a>Configurer des notifications par e-mail pour les problèmes rencontrés dans Azure Active Directory Domain Services

L’intégrité d’un domaine managé Azure Active Directory Domain Services (Azure AD DS) est supervisée par la plateforme Azure. Les alertes relatives au domaine managé s’affichent dans la page État d’intégrité du portail Azure. Pour assurer des interventions en temps voulu quand des problèmes surviennent, il est possible de configurer des notifications par e-mail pour signaler les alertes d’intégrité dès qu’elles sont détectées dans le domaine managé Azure AD DS.

Cet article vous montre comment configurer les destinataires des notifications par e-mail pour un domaine managé.

## <a name="email-notification-overview"></a>Vue d’ensemble des notifications par e-mail

Pour être alerté des problèmes qui touchent un domaine managé, vous pouvez configurer des notifications par e-mail. Celles-ci indiquent le domaine managé dans lequel l’alerte est présente, l’heure de détection et fournit un lien vers la page d’état d’intégrité du portail Azure. Vous pouvez ensuite suivre les conseils de dépannage fournis pour résoudre les problèmes.

L’exemple de notification par e-mail présenté ci-dessous indique qu’un avertissement ou une alerte critique a été généré dans le domaine managé :

![Exemple de notification par e-mail](./media/active-directory-domain-services-alerts/email-alert.png)

> [!WARNING]
> Assurez-vous toujours que l’e-mail provient d’un expéditeur Microsoft vérifié avant de cliquer sur les liens contenus dans le message. Les notifications par e-mail proviennent toujours de l’adresse `azure-noreply@microsoft.com`.

### <a name="why-would-i-receive-email-notifications"></a>Pourquoi est-ce que je recevrais des notifications par e-mail ?

Azure AD DS envoie des notifications par e-mail pour les faits nouveaux importants qui touchent le domaine managé. Ces notifications concernent uniquement les problèmes urgents qui ont un impact sur le service et qui doivent être traités immédiatement. Chaque notification par e-mail est déclenchée par une alerte sur le domaine managé. Les alertes s’affichent aussi sur le portail Azure et sont visibles dans la [page d’état d’intégrité d’Azure AD DS][check-health].

Azure AD DS n’envoie pas d’e-mails à des fins publicitaires, commerciales ou de mise à jour.

### <a name="when-will-i-receive-email-notifications"></a>Quand est-ce que je recevrai des notifications par e-mail ?

Les notifications sont envoyées aussitôt qu’une [nouvelle alerte][troubleshoot-alerts] est détectée dans un domaine managé. Si l’alerte n’est pas résolue, d’autres notifications de rappel sont envoyées par e-mail tous les quatre jours.

### <a name="who-should-receive-the-email-notifications"></a>Qui doit recevoir les notifications par e-mail ?

La liste des destinataires pour Azure AD DS doit être composée de personnes capables d’administrer le domaine managé et d’y apporter des modifications. Cette liste doit être considérée comme celle des « premiers intervenants » pour tout type d’alerte et de problème.

Vous pouvez ajouter jusqu’à cinq destinataires supplémentaires pour les notifications par e-mail. Si vous voulez que les notifications par e-mail soient envoyées à plus de cinq destinataires, créez une liste de distribution et ajoutez-la à la liste de notification.

Vous pouvez aussi décider que tous les *administrateurs généraux* de l’annuaire Azure AD et tous les membres du groupe *Administrateurs AAD DC* recevront les notifications par e-mail. Azure AD DS peut envoyer les notifications à 100 adresses e-mail au maximum, en incluant la liste des administrateurs généraux et des administrateurs AAD DC.

## <a name="configure-email-notifications"></a>Configurer les notifications par e-mail

Pour examiner les destinataires existants des notifications par e-mail ou pour ajouter des destinataires supplémentaires, effectuez les étapes suivantes :

1. Sur le portail Azure, recherchez et sélectionnez **Azure AD Domain Services**.
1. Sélectionnez votre domaine managé, par exemple *aaddscontoso.com*.
1. Dans la partie gauche de la fenêtre de ressources Azure AD DS, sélectionnez **Paramètres de notification**. Les destinataires existants des notifications par e-mail s’affichent.
1. Pour ajouter un destinataire, entrez son adresse e-mail dans le tableau des destinataires supplémentaires.
1. Quand vous avez terminé, sélectionnez **Enregistrer** dans le volet de navigation du haut.

> [!WARNING]
> Quand vous modifiez les paramètres de notification, vous mettez à jour les paramètres de notification du domaine managé tout entier, pas seulement les vôtres.

## <a name="frequently-asked-questions"></a>Forum aux questions

### <a name="i-received-an-email-notification-for-an-alert-but-when-i-logged-on-to-the-azure-portal-there-was-no-alert-what-happened"></a>J’ai reçu une notification par e-mail pour une alerte, mais lorsque j’ai ouvert une session, le portail Azure ne contenait pas d’alerte. Que s’est-il passé ?

Une alerte qui été résolue ne figure plus sur le portail Azure. La raison la plus probable est qu’un autre destinataire des notifications par e-mail a résolu l’alerte dans votre domaine managé ou elle a été résolue automatiquement par la plateforme Azure.

### <a name="why-can-i-not-edit-the-notification-settings"></a>Pourquoi ne puis-je pas modifier les paramètres de notification ?

Si vous ne parvenez pas à accéder à la page des paramètres de notification sur le portail Azure, c’est que vous n’avez pas les autorisations permettant de modifier le domaine managé. Contactez un administrateur général pour obtenir les autorisations permettant de modifier les ressources Azure AD DS ou pour être retiré de la liste des destinataires.

### <a name="i-dont-seem-to-be-receiving-email-notifications-even-though-i-provided-my-email-address-why"></a>Je ne reçois pas de notifications par e-mail, même si j’ai fourni mon adresse e-mail. Pourquoi ?

Recherchez des notifications dans le dossier de courrier indésirable de votre boîte de réception et veillez à autoriser l’expéditeur de `azure-noreply@microsoft.com`.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur la résolution de certains problèmes susceptibles d’être signalés, consultez [Résoudre les alertes dans un domaine managé][troubleshoot-alerts].

<!-- INTERNAL LINKS -->
[check-health]: check-health.md
[troubleshoot-alerts]: troubleshoot-alerts.md
