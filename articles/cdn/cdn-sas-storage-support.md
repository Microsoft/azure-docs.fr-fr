---
title: Utilisation d’Azure CDN avec SAP | Microsoft Docs
description: Azure CDN prend en charge l’utilisation de la signature d’accès partagé (SAP) pour accorder un accès limité à des conteneurs de stockage privé.
services: cdn
documentationcenter: ''
author: asudbring
manager: danielgi
editor: ''
ms.assetid: ''
ms.service: azure-cdn
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 06/21/2018
ms.author: allensu
ms.openlocfilehash: ff205069c31d50813a4fad71a3c9e2f8e2462844
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92778125"
---
# <a name="using-azure-cdn-with-sas"></a>Utilisation d’Azure CDN avec SAP

Quand vous configurez un compte de stockage pour Azure Content Delivery Network (CDN) en vue de l’utiliser comme cache de contenu, par défaut toute personne connaissant les URL de vos conteneurs de stockage peut accéder aux fichiers que vous avez chargés. Pour protéger les fichiers dans votre compte de stockage, vous pouvez rendre privé l’accès à vos conteneurs de stockage. Toutefois, si vous le faites, personne ne peut accéder à vos fichiers. 

Si vous souhaitez accorder un accès limité à des conteneurs de stockage privé, vous pouvez utiliser la fonctionnalité de signature d’accès partagé (SAP) de votre compte de stockage Azure. Une SAP est un URI qui octroie des droits d’accès restreints à vos ressources Stockage Azure sans exposer votre clé de compte. Vous pouvez fournir une SAP aux clients auxquels vous préférez ne pas donner votre clé de compte de stockage, mais auxquels vous souhaitez déléguer l’accès à certaines ressources de compte de stockage. Quand vous distribuez un URI de signature d’accès partagé à ces clients, vous leur accordez l’accès à une ressource pour une période spécifiée.
 
Avec une signature SAP, vous pouvez définir différents paramètres d’accès à un objet blob, tels que les heures de début et d’expiration, les autorisations (lecture/écriture) et les plages d’adresses IP. Cet article décrit comment utiliser la SAP avec Azure CDN. Pour plus d’informations sur les SAP, notamment sur la création de leurs options de paramètre, consultez [Utilisation des signatures d’accès partagé (SAP)](../storage/common/storage-sas-overview.md).

## <a name="setting-up-azure-cdn-to-work-with-storage-sas"></a>Configuration d’Azure CDN pour fonctionner avec une SAP de stockage
Les trois options suivantes sont recommandées pour l’utilisation de SAP avec Azure CDN. Toutes les options partent du principe que vous avez déjà créé une SAP opérationnelle (voir les prérequis). 
 
### <a name="prerequisites"></a>Prérequis
Pour commencer, créez un compte de stockage, puis générez une SAP pour votre ressource. Vous pouvez générer deux types de signatures d’accès partagé : une SAP de service ou une SAP de compte. Pour plus d’informations, consultez [Types de signatures d’accès partagé](../storage/common/storage-sas-overview.md#types-of-shared-access-signatures).

Après avoir généré un jeton SAP, vous pouvez accéder à votre fichier de stockage d’objets blob en ajoutant `?sv=<SAS token>` à l’URL. Cette URL a le format suivant : 

`https://<account name>.blob.core.windows.net/<container>/<file>?sv=<SAS token>`
 
Par exemple :
 ```
https://democdnstorage1.blob.core.windows.net/container1/demo.jpg?sv=2017-07-29&ss=b&srt=co&sp=r&se=2038-01-02T21:30:49Z&st=2018-01-02T13:30:49Z&spr=https&sig=QehoetQFWUEd1lhU5iOMGrHBmE727xYAbKJl5ohSiWI%3D
```

Pour plus d’informations sur la définition des paramètres, consultez [Considérations relatives aux paramètres SAP](#sas-parameter-considerations) et [Paramètres de la signature d’accès partagé (SAP)](../storage/common/storage-sas-overview.md#how-a-shared-access-signature-works).

![Paramètres de SAP CDN](./media/cdn-sas-storage-support/cdn-sas-settings.png)

### <a name="option-1-using-sas-with-pass-through-to-blob-storage-from-azure-cdn"></a>Option 1 : Utilisation de SAP avec transfert direct vers le stockage blob à partir d'Azure CDN

Cette option est la plus simple. Elle utilise un jeton SAP unique, qui est passé d’Azure CDN au serveur d’origine.
 
1. Sélectionnez un point de terminaison, sélectionnez **Règles de mise en cache** , puis sélectionnez **Mettre en cache chaque URL unique** dans la liste **Mise en cache de chaîne de requête** .

    ![Règles de mise en cache de CDN](./media/cdn-sas-storage-support/cdn-caching-rules.png)

2. Après avoir configuré SAP sur votre compte de stockage, vous devez utiliser le jeton SAP avec les URL du point de terminaison CDN et du serveur d’origine pour accéder au fichier. 
   
   L’URL de point de terminaison CDN qui en résulte a le format suivant : `https://<endpoint hostname>.azureedge.net/<container>/<file>?sv=<SAS token>`

   Par exemple :   
   ```
   https://demoendpoint.azureedge.net/container1/demo.jpg/?sv=2017-07-29&ss=b&srt=c&sp=r&se=2027-12-19T17:35:58Z&st=2017-12-19T09:35:58Z&spr=https&sig=kquaXsAuCLXomN7R00b8CYM13UpDbAHcsRfGOW3Du1M%3D
   ```
   
3. Affinez la durée du cache à l’aide de règles de mise en cache ou en ajoutant des en-têtes `Cache-Control` au serveur d’origine. Étant donné qu’Azure CDN traite le jeton SAP comme une chaîne de requête simple, la bonne pratique consiste à définir une durée de mise en cache qui expire au plus tard au moment de l’expiration de la signature SAP. Dans le cas contraire, si un fichier est mis en cache pour une durée plus longue que celle pendant laquelle la signature SAP est active, le fichier peut être accessible à partir du serveur d’origine Azure CDN après l’expiration de la SAP. Si ce cas se produit et que vous souhaitez rendre votre fichier de mise en cache inaccessible, vous devez effectuer une opération de vidage sur le fichier afin de le supprimer du cache. Pour plus d’informations sur la définition de la durée du cache sur Azure CDN, consultez [Contrôler le comportement de mise en cache d’Azure CDN avec des règles de mise en cache](cdn-caching-rules.md).

### <a name="option-2-hidden-cdn-sas-token-using-a-rewrite-rule"></a>Option n°2 : Jeton SAP CDN masqué utilisant une règle de réécriture
 
Cette option est disponible uniquement pour les profils **Azure CDN Premium de Verizon** . Avec cette option, vous pouvez sécuriser le stockage blob sur le serveur d’origine. Vous pouvez utiliser cette option si vous n’avez pas besoin de restrictions d’accès spécifiques pour le fichier, mais que vous voulez empêcher les utilisateurs d’accéder à l’origine du stockage directement afin d’accélérer le temps de déchargement d’Azure CDN. Le jeton SAP, qui est inconnu de l’utilisateur, est nécessaire à quiconque accède aux fichiers dans le conteneur spécifié du serveur d’origine. Toutefois, en raison de la règle de réécriture d’URL, le jeton SAP n’est pas nécessaire sur le point de terminaison CDN.
 
1. Utilisez le [moteur de règles](./cdn-verizon-premium-rules-engine.md) pour créer une règle de réécriture d’URL. La propagation de nouvelles règles peut prendre jusqu’à 4 heures.

   ![Bouton Gérer du CDN](./media/cdn-sas-storage-support/cdn-manage-btn.png)

   ![Bouton de moteur de règles de CDN](./media/cdn-sas-storage-support/cdn-rules-engine-btn.png)

   L’exemple de règle de réécriture d’URL suivant utilise un modèle d’expression régulière avec un groupe de capture et un point de terminaison nommé *sasstoragedemo*  :
   
   Source :   
   `(container1\/.*)`


   Destination :   
   ```
   $1?sv=2017-07-29&ss=b&srt=c&sp=r&se=2027-12-19T17:35:58Z&st=2017-12-19T09:35:58Z&spr=https&sig=kquaXsAuCLXomN7R00b8CYM13UpDbAHcsRfGOW3Du1M%3D
   ```
   ![Règle de réécriture d’URL CDN – gauche](./media/cdn-sas-storage-support/cdn-url-rewrite-rule.png)
   ![Règle de réécriture d’URL CDN – droite](./media/cdn-sas-storage-support/cdn-url-rewrite-rule-option-4.png)

2. Une fois la nouvelle règle active, tous les utilisateurs peuvent accéder aux fichiers dans le conteneur spécifié sur le point de terminaison CDN, même s’ils n’utilisent pas de jeton SAP dans l’URL. Voici le format : `https://<endpoint hostname>.azureedge.net/<container>/<file>`
 
   Par exemple :   
   `https://sasstoragedemo.azureedge.net/container1/demo.jpg`
       

3. Affinez la durée du cache à l’aide de règles de mise en cache ou en ajoutant des en-têtes `Cache-Control` au serveur d’origine. Étant donné qu’Azure CDN traite le jeton SAP comme une chaîne de requête simple, la bonne pratique consiste à définir une durée de mise en cache qui expire au plus tard au moment de l’expiration de la signature SAP. Dans le cas contraire, si un fichier est mis en cache pour une durée plus longue que celle pendant laquelle la signature SAP est active, le fichier peut être accessible à partir du point de terminaison Azure CDN après l’expiration de la SAP. Si ce cas se produit et que vous souhaitez rendre votre fichier de mise en cache inaccessible, vous devez effectuer une opération de vidage sur le fichier afin de le supprimer du cache. Pour plus d’informations sur la définition de la durée du cache sur Azure CDN, consultez [Contrôler le comportement de mise en cache d’Azure CDN avec des règles de mise en cache](cdn-caching-rules.md).

### <a name="option-3-using-cdn-security-token-authentication-with-a-rewrite-rule"></a>Option 3 : Utilisation de l'authentification de jeton de sécurité de CDN avec une règle de réécriture

Pour utiliser l’authentification de jeton de sécurité d’Azure CDN, vous devez avoir un profil **Azure CDN Premium de Verizon** . Cette option est la plus sécurisée et personnalisable. L’accès du client est basé sur les paramètres de sécurité que vous avez définis sur le jeton de sécurité. Une fois que vous aurez créé et configuré le jeton de sécurité, il sera nécessaire sur toutes les URL de point de terminaison CDN. Toutefois, en raison de la règle de réécriture d’URL, le jeton SAP n’est pas nécessaire sur le point de terminaison CDN. Si le jeton SAP devient non valide, Azure CDN ne peut plus revalider le contenu à partir du serveur d’origine.

1. [Créez un jeton de sécurité Azure CDN](./cdn-token-auth.md#setting-up-token-authentication) et activez-le à l’aide du moteur de règles pour le point de terminaison de CDN et du chemin où les utilisateurs peuvent accéder au fichier.

   Une URL de point de terminaison de jeton de sécurité a le format suivant :   
   `https://<endpoint hostname>.azureedge.net/<container>/<file>?<security_token>`
 
   Par exemple :   
   ```
   https://sasstoragedemo.azureedge.net/container1/demo.jpg?a4fbc3710fd3449a7c99986bkquaXsAuCLXomN7R00b8CYM13UpDbAHcsRfGOW3Du1M%3D
   ```
       
   Les options de paramètres pour l’authentification de jeton de sécurité diffèrent de celles pour un jeton SAP. Si vous choisissez d’utiliser un délai d’expiration quand vous créez un jeton de sécurité, vous devez lui affecter la même valeur que le délai d’expiration du jeton SAP. Cela garantit le caractère prévisible du délai d’expiration. 
 
2. Utilisez le [moteur de règles](./cdn-verizon-premium-rules-engine.md) pour créer une règle de réécriture d’URL visant à activer l’accès du jeton SAP à tous les objets blob dans le conteneur. La propagation de nouvelles règles peut prendre jusqu’à 4 heures.

   L’exemple de règle de réécriture d’URL suivant utilise un modèle d’expression régulière avec un groupe de capture et un point de terminaison nommé *sasstoragedemo*  :
   
   Source :   
   `(container1\/.*)`
   
   Destination :   
   ```
   $1&sv=2017-07-29&ss=b&srt=c&sp=r&se=2027-12-19T17:35:58Z&st=2017-12-19T09:35:58Z&spr=https&sig=kquaXsAuCLXomN7R00b8CYM13UpDbAHcsRfGOW3Du1M%3D
   ```
   ![Règle de réécriture d’URL CDN – gauche](./media/cdn-sas-storage-support/cdn-url-rewrite-rule.png)
   ![Règle de réécriture d’URL CDN – droite](./media/cdn-sas-storage-support/cdn-url-rewrite-rule-option-4.png)

3. Si vous renouvelez la signature SAP, n’oubliez pas de mettre à jour la règle de réécriture d’URL avec le nouveau jeton SAP. 

## <a name="sas-parameter-considerations"></a>Considérations relatives aux paramètres SAP

Azure CDN ne peut pas changer son comportement de remise en se basant sur les paramètres SAP, car il ne le voit pas. Les restrictions de paramètres définies s’appliquent uniquement aux requêtes envoyées par Azure CDN au serveur d’origine, et non aux requêtes du client vers Azure CDN. Cette distinction est importante quand vous définissez des paramètres SAP. Si ces fonctionnalités avancées sont nécessaires et que vous utilisez [l’Option 3](#option-3-using-cdn-security-token-authentication-with-a-rewrite-rule), définissez les restrictions appropriées sur le jeton de sécurité Azure CDN.

| Nom du paramètre SAP | Description |
| --- | --- |
| Démarrer | Heure à laquelle Azure CDN peut commencer à accéder au fichier blob. En raison des variations d’horloges (quand un signal d’horloge arrive à des moments différents pour différents composants), vous devez choisir une heure 15 minutes plus tôt si vous souhaitez que la ressource soit disponible immédiatement. |
| End | Heure après laquelle Azure CDN ne peut plus accéder au fichier blob. Les fichiers précédemment mis en cache sur Azure CDN sont toujours accessibles. Pour contrôler le délai d’expiration de fichier, définissez le délai d’expiration approprié sur le jeton de sécurité Azure CDN ou nettoyez la ressource. |
| Adresses IP autorisées | facultatif. Si vous utilisez **Azure CDN de Verizon** , vous pouvez affecter à ce paramètre les plages définies dans [Azure CDN from Verizon Edge Server IP Ranges (Plages d’adresses IP de serveur Edge Azure CDN de Verizon)](./cdn-pop-list-api.md). Si vous utilisez **CDN Azure d’Akamai** , vous ne pouvez pas définir le paramètre de plages IP, car les adresses IP ne sont pas statiques.|
| Protocoles autorisés | Protocole(s) autorisé(s) pour une requête effectuée avec la SAP de compte. Le paramètre HTTPS est recommandé.|

## <a name="next-steps"></a>Étapes suivantes

Pour plus d'informations sur les SAP, voir les articles suivants :
- [Utilisation des signatures d’accès partagé (SAP)](../storage/common/storage-sas-overview.md)
- [Signatures d’accès partagé, partie 2 : Créer et utiliser une signature d’accès partagé avec Blob Storage](../storage/common/storage-sas-overview.md)

Pour plus d’informations sur la configuration de l’authentification par jeton, voir [Sécuriser des ressources Azure Content Delivery Network avec l’authentification par jeton](./cdn-token-auth.md).