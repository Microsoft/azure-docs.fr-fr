---
title: Se préparer à un changement d’adresse IP SSL
description: Si votre adresse IP SSL va être modifiée, découvrez les choses à faire pour que votre application continue de fonctionner après la modification.
ms.topic: article
ms.date: 06/28/2018
ms.custom: seodec18
ms.openlocfilehash: dcfe11bcab25f6267a557de5faf7befab467bc29
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96020957"
---
# <a name="how-to-prepare-for-an-ssl-ip-address-change"></a>Comment se préparer à un changement d’adresse IP SSL

Si vous avez reçu une notification indiquant la modification de l’adresse IP SSL de votre application Azure App Service, suivez les instructions dans cet article pour libérer l’adresse IP SSL existante et en affecter une nouvelle.

## <a name="release-ssl-ip-addresses-and-assign-new-ones"></a>Libérer des adresses IP SSL et en affecter de nouvelles

1.  Ouvrez le [portail Azure](https://portal.azure.com).

2.  Dans le menu de navigation de gauche, sélectionnez **App Services**.

3.  Sélectionnez votre application App Service dans la liste.

4.  Sous l’en-tête **Paramètres**, cliquez sur **Paramètres SSL** dans le volet de navigation gauche.

1. Dans la section Liaisons TLS/SSL, sélectionnez l’enregistrement du nom d’hôte. Dans l’éditeur qui s’ouvre, choisissez **SNI SSL** dans le menu déroulant **Type SSL** et cliquez sur **Ajouter une liaison**. Lorsque le message de réussite d’opération s’affiche, l’adresse IP existante a été libérée.

6.  Dans la section **liaisons SSL**, sélectionnez à nouveau le même enregistrement de nom d’hôte avec le certificat. Dans l’éditeur qui s’ouvre, choisissez cette fois **SSL sur IP** dans le menu déroulant **Type SSL** et cliquez sur **Ajouter une liaison**. Lorsque vous voyez le message de réussite d’opération, vous avez acquis une nouvelle adresse IP.

7.  Si un enregistrement A (enregistrement DNS qui pointe directement vers votre adresse IP) est configuré dans votre portail d’inscription de domaine (fournisseur DNS tiers ou Azure DNS), remplacez l’adresse IP existante avec celle qui vient d’être générée. Vous trouverez la nouvelle adresse IP en suivant les instructions fournies dans la section suivante.

## <a name="find-the-new-ssl-ip-address-in-the-azure-portal"></a>Trouver la nouvelle adresse IP SSL sur le Portail Azure

1.  Patientez quelques minutes, puis ouvrez le [portail Azure](https://portal.azure.com).

2.  Dans le menu de navigation de gauche, sélectionnez **App Services**.

3.  Sélectionnez votre application App Service dans la liste.

4.  Sous l’en-tête **Paramètres**, cliquez sur **Propriétés** dans le volet de navigation de gauche, puis recherchez la section intitulée **Adresse IP virtuelle**.

5. Copiez l’adresse IP et reconfigurez votre enregistrement de domaine ou votre mécanisme d’IP.

## <a name="next-steps"></a>Étapes suivantes

Cet article a expliqué comment se préparer à un changement d’adresse IP initié par Azure. Pour plus d’informations sur les adresses IP dans Azure App Service, consultez [Inbound and Inbound IP addresses in Azure App Service](overview-inbound-outbound-ips.md) (Entrées et adresses IP entrantes dans Azure App Service).
