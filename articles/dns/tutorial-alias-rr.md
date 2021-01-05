---
title: 'Tutoriel : Créer un enregistrement d’alias pour référencer un enregistrement de ressource dans une zone'
titleSuffix: Azure DNS
description: Ce tutoriel vous montre comment configurer un enregistrement d’alias Azure DNS pour référencer un enregistrement de ressource dans la zone.
services: dns
author: rohinkoul
ms.service: dns
ms.topic: tutorial
ms.date: 9/25/2018
ms.author: rohink
ms.openlocfilehash: 2b122a34cfd382a58f7680743d3a1cb1ae598fd1
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "76939247"
---
# <a name="tutorial-create-an-alias-record-to-refer-to-a-zone-resource-record"></a>Tutoriel : Créer un enregistrement d’alias pour référencer un enregistrement de ressource de zone

Les enregistrements d’alias peuvent référencer d’autres jeux d’enregistrements du même type. Par exemple, un jeu d’enregistrements CNAME DNS peut être un alias pour un autre jeu d’enregistrements CNAME du même type. Cette fonctionnalité est pratique si vous voulez que certains jeux d’enregistrements se comportent comme des alias et d’autres comme des non-alias.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Créez un enregistrement d’alias pour un enregistrement de ressource dans la zone.
> * Tester l’enregistrement d’alias.


Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="prerequisites"></a>Prérequis
Vous devez disposer d’un nom de domaine disponible, que vous pouvez héberger dans Azure DNS pour le test. Vous devez disposer d’un contrôle total de ce domaine. Le contrôle total comprend notamment la possibilité de définir les enregistrements de serveur de noms pour le domaine.

Pour obtenir des instructions sur l’hébergement de votre domaine dans Azure DNS, consultez [Tutoriel : Héberger votre domaine dans Azure DNS](dns-delegate-domain-azure-dns.md).


## <a name="create-an-alias-record"></a>Créer un enregistrement d’alias

Créez un enregistrement d’alias qui pointe vers un enregistrement de ressource dans la zone.

### <a name="create-the-target-resource-record"></a>Créer l’enregistrement de ressource cible
1. Sélectionnez votre zone Azure DNS pour ouvrir la zone.
2. Sélectionnez **Jeu d’enregistrements**.
3. Dans la zone de texte **Nom**, entrez **server**.
4. Pour le **Type**, sélectionnez **A**.
5. Dans la zone de texte **ADRESSE IP**, tapez **10.10.10.10**.
6. Sélectionnez **OK**.

### <a name="create-the-alias-record"></a>Créer l’enregistrement d’alias
1. Sélectionnez votre zone Azure DNS pour ouvrir la zone.
2. Sélectionnez **Jeu d’enregistrements**.
3. Dans la zone de texte **Nom**, entrez **test**.
4. Pour le **Type**, sélectionnez **A**.
5. Sélectionnez **Oui** dans la case à cocher **Alias du jeu d’enregistrements**. Ensuite, sélectionnez l’option **Jeu d’enregistrements de zone**.
6. Pour le **Jeu d’enregistrements de zone**, sélectionnez l’enregistrement **server**.
7. Sélectionnez **OK**.

## <a name="test-the-alias-record"></a>Tester l’enregistrement d’alias

1. Lancez votre outil nslookup préféré. Une option est d’accéder à [https://network-tools.com/nslook](https://network-tools.com/nslook).
2. Définissez le type de requête pour les enregistrements A et recherchez **test.\<your domain name\>** . La réponse est **10.10.10.10**.
3. Dans le portail Azure, changez l’enregistrement A **server** en **10.11.11.11**.
4. Attendez quelques minutes, puis utilisez à nouveau nslookup pour l’enregistrement **test**. La réponse est **10.11.11.11**.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Lorsque vous n’avez plus besoin des ressources créées pour ce didacticiel, supprimez les enregistrements de ressources **server** et **test** dans votre zone.


## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez créé un enregistrement d’alias pour référencer un enregistrement de ressource dans la zone. Pour en savoir plus sur Azure DNS et les applications web, passez au tutoriel sur les applications web.

> [!div class="nextstepaction"]
> [Créer des enregistrements DNS pour une application web dans un domaine personnalisé](./dns-web-sites-custom-domain.md)
