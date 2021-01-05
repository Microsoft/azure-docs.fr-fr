---
title: Héberger des applications web Azure à charge équilibrée à l’extrémité de la zone
description: Utilisez un enregistrement d’alias Azure DNS pour héberger des applications web à charge équilibrée à l’extrémité de la zone.
services: dns
author: rohinkoul
ms.service: dns
ms.topic: how-to
ms.date: 08/10/2019
ms.author: rohink
ms.openlocfilehash: 72adb2732eb0832589cbc25fb7e4288eb1899214
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94954509"
---
# <a name="host-load-balanced-azure-web-apps-at-the-zone-apex"></a>Héberger des applications web Azure à charge équilibrée à l’extrémité de la zone

Le protocole DNS empêche l’affectation de tout enregistrement autre qu’un enregistrement A ou AAAA au niveau de l’apex de zone. contoso.com est un exemple d’extrémité de zone. Cette restriction pose un problème pour les propriétaires d’application qui ont des applications à charge équilibrée derrière un profil Traffic Manager. Il est impossible de pointer vers le profil Traffic Manager à partir de l’enregistrement d’apex de zone. Les propriétaires d’application doivent donc trouver une solution de contournement. Une redirection au niveau de la couche Application doit rediriger vers un autre domaine à partir de l’apex de zone. Une redirection de contoso.com vers www\.contoso.com en est un exemple. Cette disposition présente un point de défaillance unique de la fonction de redirection.

Les enregistrements d’alias éliminent ce problème. Les propriétaires d’applications peuvent maintenant pointer leur enregistrement d’apex de zone vers un profil Traffic Manager disposant de points de terminaison externes. Les propriétaires d’applications peuvent pointer vers le même profil Traffic Manager que celui utilisé pour tout autre domaine dans leur zone DNS.

Par exemple, contoso.com et www\.contoso.com peuvent pointer vers le même profil Traffic Manager. C’est le cas tant que le profil Traffic Manager ne compte que des points de terminaison externes.

Cet article explique comment créer un enregistrement d’alias pour une extrémité de domaine et configurer des points de terminaison de profil Traffic Manager pour des applications web.

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="prerequisites"></a>Prérequis

Vous devez disposer d’un nom de domaine disponible, que vous pouvez héberger dans Azure DNS pour le test. Vous devez disposer d’un contrôle total de ce domaine. Le contrôle total comprend notamment la possibilité de définir les enregistrements de serveur de noms pour le domaine.

Pour obtenir des instructions sur l’hébergement de votre domaine dans Azure DNS, consultez [Tutoriel : Héberger votre domaine dans Azure DNS](dns-delegate-domain-azure-dns.md).

L’exemple de domaine utilisé pour ce tutoriel est contoso.com, mais utilisez votre propre nom de domaine.

## <a name="create-a-resource-group"></a>Créer un groupe de ressources

Créez un groupe de ressources qui contiendra toutes les ressources utilisées dans cet article.

## <a name="create-app-service-plans"></a>Créer des plans App Service

Créez deux plans Web App Service dans votre groupe de ressources en suivant les informations de configuration du tableau ci-dessous. Pour plus d’informations sur la création d’un plan App Service, voir [Gérer un plan App Service dans Azure](../app-service/app-service-plan-manage.md).


|Nom  |Système d’exploitation  |Emplacement  |Niveau de tarification  |
|---------|---------|---------|---------|
|ASP-01     |Windows|USA Est|Dev/Test D1-Shared|
|ASP-02     |Windows|USA Centre|Dev/Test D1-Shared|

## <a name="create-app-services"></a>Créer des applications App Service

Créez deux applications web, une dans chaque plan App Service.

1. En haut à gauche de la page du Portail Azure, sélectionnez **Créer une ressource**.
2. Tapez **Application web** dans la barre de recherche et appuyez sur Entrée.
3. Sélectionnez **Application web**.
4. Sélectionnez **Create** (Créer).
5. Acceptez les valeurs par défaut et utilisez le tableau suivant pour configurer les deux applications web :

   |Nom<br>(doit être unique sur . azurewebsites.net)|Groupe de ressources |Pile d’exécution|Région|Plan App Service/Emplacement
   |---------|---------|-|-|-------|
   |App-01|Utilisez l’existant<br>Sélectionnez votre groupe de ressources|.NET Core 2.2|USA Est|ASP-01(D1)|
   |App-02|Utilisez l’existant<br>Sélectionnez votre groupe de ressources|.NET Core 2.2|USA Centre|ASP-02(D1)|

### <a name="gather-some-details"></a>Collecter quelques informations

Notez maintenant l’adresse IP et le nom d’hôte des applications web.

1. Ouvrez votre groupe de ressources et sélectionnez votre première application web (**App-01** dans cet exemple).
2. Dans la colonne gauche, sélectionnez **Propriétés**.
3. Notez l’adresse sous **URL**, puis, sous **Adresses IP sortantes**, notez la première adresse IP de la liste. Vous utiliserez ces informations au moment de configurer vos points de terminaison Traffic Manager.
4. Répétez l’opération pour **App-02**.

## <a name="create-a-traffic-manager-profile"></a>Créer un profil Traffic Manager

Créez un profil Traffic Manager dans votre groupe de ressources. Utilisez les valeurs par défaut et tapez un nom unique sur l’espace de noms trafficmanager.net.

Pour plus d’informations sur la création d’un profil Traffic Manager, consultez [Démarrage rapide : Créer un profil Traffic Manager pour une application web hautement disponible](../traffic-manager/quickstart-create-traffic-manager-profile.md).

### <a name="create-endpoints"></a>Créer des points de terminaison

Vous pouvez maintenant créer les points de terminaison pour les deux applications web.

1. Ouvrez votre groupe de ressources, puis sélectionnez votre profil Traffic Manager.
2. Dans la colonne gauche, sélectionnez **Points de terminaison**.
3. Sélectionnez **Ajouter**.
4. Aidez-vous du tableau suivant pour configurer les points de terminaison :

   |Type  |Nom  |Cible  |Emplacement  |Paramètres d’en-tête personnalisé|
   |---------|---------|---------|---------|---------|
   |Point de terminaison externe     |End-01|Adresse IP enregistrée pour App-01|USA Est|Hôte : \<the URL you recorded for App-01\><br>Exemple : **host:app-01.azurewebsites.net**|
   |Point de terminaison externe     |End-02|Adresse IP enregistrée pour App-02|USA Centre|Hôte : \<the URL you recorded for App-02\><br>Exemple : **host:app-02.azurewebsites.net**

## <a name="create-dns-zone"></a>Créer une zone DNS

Vous pouvez utiliser une zone DNS existante à des fins de test, ou en créer une nouvelle. Pour créer et déléguer une nouvelle zone DNS dans Azure, consultez [Tutoriel : Héberger votre domaine dans Azure DNS](dns-delegate-domain-azure-dns.md).

## <a name="add-a-txt-record-for-custom-domain-validation"></a>Ajouter un enregistrement TXT pour la validation d’un domaine personnalisé

Lorsque vous ajoutez un nom d’hôte personnalisé à vos applications web, il recherche un enregistrement TXT spécifique pour valider votre domaine.

1. Ouvrez votre groupe de ressources, puis sélectionnez la zone DNS.
2. Sélectionnez **Jeu d’enregistrements**.
3. Ajoutez le jeu d’enregistrements suivant le tableau ci-dessous. Pour la valeur, utilisez l’URL de l’application Web réelle enregistrée précédemment :

   |Nom  |Type  |Valeur|
   |---------|---------|-|
   |@     |TXT|App-01.azurewebsites.net|


## <a name="add-a-custom-domain"></a>Ajouter un domaine personnalisé

Ajoutez un domaine personnalisé pour les applications web.

1. Ouvrez votre groupe de ressources et sélectionnez votre première application web.
2. Dans la colonne de gauche, sélectionnez **Domaines personnalisés**.
3. Sous **Domaines personnalisés**, sélectionnez **Ajouter un domaine personnalisé**.
4. Sous **Domaines personnalisés**, saisissez votre nom de domaine personnalisé. Par exemple, contoso.com.
5. Sélectionnez **Valider**.

   Votre domaine doit réussir la validation et présenter des coches vertes à côté de **disponibilité du nom d’hôte** et de **propriété du domaine**.
5. Sélectionnez **Ajouter un domaine personnalisé**.
6. Pour voir le nouveau nom d’hôte sous **Noms d’hôte affectés au site**, actualisez votre navigateur. Les modifications n’apparaissent pas forcément tout de suite.
7. Répétez cette procédure pour votre deuxième application web.

## <a name="add-the-alias-record-set"></a>Ajouter le jeu d’enregistrements d’alias

Ajoutez maintenant un enregistrement d’alias d’apex de zone.

1. Ouvrez votre groupe de ressources, puis sélectionnez la zone DNS.
2. Sélectionnez **Jeu d’enregistrements**.
3. Ajoutez le jeu d’enregistrements suivant le tableau ci-dessous :

   |Nom  |Type  |Jeu d’enregistrements d’alias  |Type d’alias  |Ressource Azure|
   |---------|---------|---------|---------|-----|
   |@     |Un|Oui|Ressource Azure|Traffic Manager – votre profil|


## <a name="test-your-web-apps"></a>Tester les applications web

Vous pouvez maintenant passer aux tests, pour vérifier que vous pouvez atteindre votre application web et qu’elle est à charge équilibrée.

1. Ouvrez un navigateur web et accédez à votre domaine. Par exemple, contoso.com. La page de l’application web par défaut devrait apparaître.
2. Arrêtez votre première application web.
3. Fermez votre navigateur web, puis attendez quelques minutes.
4. Démarrez votre navigateur web et accédez à votre domaine. Vous devriez toujours voir la page de l’application web par défaut.
5. Arrêtez votre deuxième application web.
6. Fermez votre navigateur web, puis attendez quelques minutes.
7. Démarrez votre navigateur web et accédez à votre domaine. L’erreur 403 devrait apparaître, indiquant que l’application web est arrêtée.
8. Démarrez votre deuxième application web.
9. Fermez votre navigateur web, puis attendez quelques minutes.
10. Démarrez votre navigateur web et accédez à votre domaine. La page de l’application web par défaut devrait réapparaître.

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur les enregistrements d’alias, consultez les articles suivant :

- [Tutoriel : Configurer un enregistrement d'alias pour faire référence à une adresse IP publique](tutorial-alias-pip.md)
- [Tutoriel : Configurer un enregistrement d'alias pour prendre en charge des noms de domaine d'apex avec Traffic Manager](tutorial-alias-tm.md)
- [FAQ DNS](./dns-faq.md#alias-records)

Pour savoir comment migrer un nom DNS actif, consultez [Migrer un nom DNS actif vers Azure App Service](../app-service/manage-custom-dns-migrate-domain.md).