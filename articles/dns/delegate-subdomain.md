---
title: Déléguer un sous-domaine - Azure DNS
description: Avec ce parcours d’apprentissage, commencez à déléguer un sous-domaine Azure DNS.
services: dns
author: rohinkoul
ms.service: dns
ms.topic: how-to
ms.date: 2/7/2019
ms.author: rohink
ms.openlocfilehash: add7674771fd19f6029a94c46624006f0cf30f1a
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96011538"
---
# <a name="delegate-an-azure-dns-subdomain"></a>Déléguer un sous-domaine Azure DNS

Vous pouvez utiliser le portail Azure pour déléguer un sous-domaine DNS. Par exemple, si vous êtes propriétaire du domaine contoso.com, vous pouvez déléguer un sous-domaine nommé *engineering* à une autre zone que vous pouvez administrer séparément de la zone contoso.com.

Si vous préférez, vous pouvez déléguer un sous-domaine à l’aide d’[Azure PowerShell](delegate-subdomain-ps.md).

## <a name="prerequisites"></a>Conditions préalables requises

Pour déléguer un sous-domaine Azure DNS, vous devez tout d’abord déléguer votre domaine public à Azure DNS. Pour obtenir des instructions sur la façon de configurer vos serveurs de noms pour la délégation, consultez [Déléguer un domaine à Azure DNS](./dns-delegate-domain-azure-dns.md). Une fois votre domaine délégué à votre zone Azure DNS, vous pouvez déléguer votre sous-domaine.

> [!NOTE]
> Contoso.com est utilisé comme exemple tout au long de cet article. Changez votre nom de votre domaine par contoso.com.

## <a name="create-a-zone-for-your-subdomain"></a>Créer une zone pour votre sous-domaine

Tout d’abord, créez la zone pour le sous-domaine **engineering**.

1. Dans le portail Azure, sélectionnez **Créer une ressource**.
2. Dans la zone de recherche, tapez **DNS** et sélectionnez **Zone DNS**.
3. Sélectionnez **Create** (Créer).
4. Dans le volet **Créer une zone DNS**, tapez **engineering.contoso.com** dans la zone de texte **Nom**.
5. Sélectionnez le groupe de ressources pour votre zone. Vous souhaiterez peut-être utiliser le même groupe de ressources que celui de la zone parente, afin de regrouper les ressources similaires.
6. Cliquez sur **Créer**.
7. Une fois le déploiement réussi, accédez à la nouvelle zone.

## <a name="note-the-name-servers"></a>Noter les serveurs de noms

Ensuite, notez les quatre serveurs de noms pour le sous-domaine engineering.

Dans le volet de la zone **engineering**, notez les quatre serveurs de noms pour la zone. Vous les utiliserez ultérieurement.

## <a name="create-a-test-record"></a>Créer un enregistrement test

Créez un enregistrement **A** à des fins de test. Par exemple, créez un enregistrement A **www** et configurez-le avec une adresse IP **10.10.10.10**.

## <a name="create-an-ns-record"></a>Créer un enregistrement NS

Ensuite, créez un enregistrement de serveur de noms (NS) pour la zone **engineering**.

1. Accédez à la zone du domaine parent.
2. Sélectionnez **+ Jeu d’enregistrements**.
3. Dans le volet **Ajouter un jeu d’enregistrements**, tapez **engineering** dans la zone de texte **Nom**.
4. Pour **Type**, sélectionnez **NS**.
5. Sous **Serveur de noms**, entrez les quatre serveurs de noms que vous avez notés précédemment à partir de la zone **engineering**.
6. Cliquez sur **OK**.

## <a name="test-the-delegation"></a>Tester la délégation

Utilisez nslookup pour tester la délégation.

1. Ouvrez une fenêtre PowerShell.
2. À l’invite de commandes, tapez `nslookup www.engineering.contoso.com.`.
3. Vous devez recevoir une réponse ne faisant pas autorité montrant l’adresse **10.10.10.10**.

## <a name="next-steps"></a>Étapes suivantes

Découvrir comment [Configurer des DNS inversés dans les services hébergés par Azure](dns-reverse-dns-for-azure-services.md).