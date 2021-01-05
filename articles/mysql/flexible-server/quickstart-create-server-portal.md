---
title: 'Démarrage rapide : Créer un serveur flexible Azure Database pour MySQL – portail Azure'
description: Cet article vous guide dans l’utilisation du portail Azure pour créer un serveur flexible Azure Database pour MySQL en quelques minutes.
author: savjani
ms.author: pariks
ms.service: mysql
ms.custom: mvc
ms.topic: quickstart
ms.date: 10/22/2020
ms.openlocfilehash: 864152d1f1d0074305cbba448946bc05888b4f3b
ms.sourcegitcommit: 04fb3a2b272d4bbc43de5b4dbceda9d4c9701310
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94566756"
---
# <a name="quickstart-use-the-azure-portal-to-create-an-azure-database-for-mysql-flexible-server"></a>Démarrage rapide : Utiliser le portail Azure pour créer un serveur flexible Azure Database pour MySQL

Le serveur flexible Azure Database pour MySQL est un service managé qui vous permet d’exécuter, de gérer et de mettre à l’échelle des serveurs MySQL hautement disponibles dans le cloud. Ce guide de démarrage rapide explique comment créer un serveur flexible à l’aide du portail Azure.

> [!IMPORTANT] 
> Le serveur flexible Azure Database pour MySQL est actuellement en préversion publique.

Si vous n’avez pas d’abonnement Azure, créez un [compte Azure gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="sign-in-to-the-azure-portal"></a>Connectez-vous au portail Azure.
Accédez au [portail Azure](https://portal.azure.com/). Entrez vos informations d’identification pour vous connecter au portail. Il s’ouvre par défaut sur le tableau de bord des services.

## <a name="create-an-azure-database-for-mysql-flexible-server"></a>Créer un serveur flexible Azure Database pour MySQL

Vous créez un serveur flexible avec un ensemble défini de [ressources de calcul et de stockage](./concepts-compute-storage.md). Vous créez ce serveur dans un [groupe de ressources Azure](../../azure-resource-manager/management/overview.md).

Pour créer un serveur flexible, procédez comme suit :

1. Dans le portail, cherchez et sélectionnez **Serveurs Azure Database pour MySQL** :
    
    > :::image type="content" source="./media/quickstart-create-server-portal/find-mysql-portal.png" alt-text="Capture d’écran montrant une recherche de serveurs Azure Database pour MySQL.":::

2. Sélectionnez **Ajouter**. 

3. Dans la page **Sélectionner une option de déploiement Azure Database pour MySQL**, sélectionnez **Serveur flexible** comme option de déploiement :
     
    > :::image type="content" source="./media/quickstart-create-server-portal/deployment-option.png" alt-text="Capture d’écran montrant l’option Serveur flexible.":::    

4. Sous l’onglet **Informations de base**, entrez les informations suivantes : 

    > :::image type="content" source="./media/quickstart-create-server-portal/create-form.png" alt-text="Capture d’écran montrant l’onglet De base de la page Serveur flexible."::: 
                                    
    |**Paramètre**|**Valeur suggérée**|**Description**|
    |---|---|---|
    Abonnement|Votre nom d’abonnement|Abonnement Azure que vous souhaitez utiliser pour votre serveur. Si vous avez plusieurs abonnements, choisissez celui dans lequel vous souhaitez que la ressource soit facturée.|
    Resource group|**myresourcegroup**| Un nouveau nom de groupe de ressources ou un nom de groupe existant dans votre abonnement.|
    Nom du serveur |**mydemoserver**|Nom unique qui identifie votre serveur flexible. Le nom de domaine `mysql.database.azure.com` est ajouté au nom de serveur que vous fournissez. Le nom de serveur ne peut contenir que des lettres minuscules, des chiffres et le caractère de trait d’union (-). Il doit contenir entre 3 et 63 caractères.|
    Nom d’utilisateur administrateur |**mydemouser**| Votre compte de connexion à utiliser lorsque vous vous connectez au serveur. Le nom d’utilisateur administrateur ne peut pas être **azure_superuser**, **admin**, **administrator**, **root**, **guest** ni **public**.|
    Mot de passe |Votre mot de passe| Un nouveau mot de passe pour le compte Administrateur du serveur. Il doit contenir entre 8 et 128 caractères. Il doit également contenir des caractères de trois des catégories suivantes : lettres majuscules, lettres minuscules, chiffres (0 à 9) et caractères non alphanumériques (!, $, #, %, etc.).|
    Region|La région la plus proche de vos utilisateurs| L’emplacement géographique le plus proche de vos utilisateurs.|
    Version|**5.7**| Version principale MySQL.|
    Calcul + stockage | **Modulable**, **Standard_B1ms**, **10 Gio**, **7 jours** | Les configurations de calcul, de stockage et de sauvegarde pour votre nouveau serveur. Sélectionnez **Configurer le serveur**. **Modulable**, **Standard_B1ms**, **10 Gio** et **7 jours** sont les valeurs par défaut des paramètres **Niveau de calcul**, **Taille de calcul**, **Taille du stockage** et **Période de conservation** des sauvegardes. Vous pouvez laisser ces valeurs en l’état ou les ajuster. Pour enregistrer cette sélection de calcul et de stockage, sélectionnez **Enregistrer** afin de poursuivre la configuration. La capture d’écran ci-dessous montre les options de calcul et de stockage.|
    
    > :::image type="content" source="./media/quickstart-create-server-portal/compute-storage.png" alt-text="Capture d’écran montrant les options de calcul et de stockage.":::

5. Configurez les options de mise en réseau.

    Sous l’onglet **Mise en réseau**, vous pouvez choisir la façon dont votre serveur est accessible. Un serveur flexible Azure Database pour MySQL vous fournit deux possibilités de vous connecter à votre serveur : 
   - Accès public (adresses IP autorisées)
   - Accès privé (intégration au réseau virtuel) 
   
   Lorsque vous utilisez l’accès public, l’accès à votre serveur est limité aux adresses IP autorisées que vous ajoutez à une règle de pare-feu. Cette méthode empêche les applications et les outils externes de se connecter à ce serveur et à toute base de données sur ce serveur, sauf si vous créez une règle pour ouvrir le pare-feu pour une adresse IP ou une plage d’adresses IP spécifique. Lorsque vous utilisez l’accès privé (intégration au réseau virtuel), l’accès à votre serveur est limité à votre réseau virtuel. Dans ce guide de démarrage rapide, vous découvrirez comment activer l’accès public pour vous connecter au serveur. [Pour en savoir plus sur les méthodes de connectivité, consultez l’article consacré aux concepts.](./concepts-networking.md)

    > [!NOTE]
    > Vous ne pouvez pas modifier la méthode de connectivité après avoir créé le serveur. Par exemple, si vous sélectionnez **Accès public (adresses IP autorisées)** quand vous créez le serveur, vous ne pouvez pas passer à **Accès privé (intégration au réseau virtuel)** une fois le serveur créé. Nous vous recommandons vivement de créer votre serveur avec un accès privé pour contribuer à sécuriser l’accès à votre serveur via l’intégration au réseau virtuel. [Pour en savoir plus sur l’accès privé, consultez l’article consacré aux concepts.](./concepts-networking.md)

    > :::image type="content" source="./media/quickstart-create-server-portal/networking.png" alt-text="Capture d’écran montrant l’onglet Mise en réseau.":::  

6. Sélectionnez **Vérifier + créer** pour passer en revue la configuration de votre serveur flexible.

7. Sélectionnez **Créer** pour approvisionner le serveur. L'approvisionnement peut prendre quelques minutes.

8. Dans la barre d’outils, sélectionnez **Notifications** (icône de cloche) pour surveiller le processus de déploiement. Une fois le déploiement effectué, vous pouvez sélectionner **Épingler au tableau de bord**. Une vignette est alors créée pour ce serveur flexible dans le tableau de bord du portail Azure. Cette vignette est un raccourci vers la page **Vue d’ensemble** du serveur. Lorsque vous sélectionnez **Accéder à la ressource**, la page **Vue d’ensemble** du serveur s’ouvre.

Par défaut, ces bases de données sont créées sous votre serveur : information_schema, mysql, performance_schema et sys.

> [!NOTE]
> Afin d’éviter des problèmes de connectivité, vérifiez que votre réseau autorise le trafic sortant via le port 3306 qu’utilise le serveur flexible Azure Database pour MySQL.  

## <a name="connect-to-the-server-by-using-mysqlexe"></a>Se connecter au serveur à l’aide de mysql.exe

Si vous avez créé votre serveur flexible à l’aide d’un accès privé (intégration au réseau virtuel), vous devez vous connecter à votre serveur à partir d’une ressource figurant au sein du même réseau virtuel que votre serveur. Vous pouvez créer une machine virtuelle et l’ajouter au réseau virtuel créé avec votre serveur flexible.

Si vous avez créé votre serveur flexible en utilisant un accès public (adresses IP autorisées), vous pouvez ajouter votre adresse IP locale à la liste des règles de pare-feu sur votre serveur.

Vous pouvez utiliser [mysql.exe](https://dev.mysql.com/doc/refman/8.0/en/mysql.html) ou [MySQL Workbench](./connect-workbench.md) pour vous connecter au serveur à partir de votre environnement local. 

Si vous utilisez mysql.exe, connectez-vous à l’aide de la commande suivante. Utilisez le nom du serveur, le nom d’utilisateur et le mot de passe dans la commande. 

```bash
 mysql -h mydemoserver.mysql.database.azure.com -u mydemouser -p
```
## <a name="clean-up-resources"></a>Nettoyer les ressources
Vous venez de créer un serveur flexible Azure Database pour MySQL dans un groupe de ressources. Si vous ne pensez pas avoir besoin de ces ressources à l’avenir, vous pouvez les supprimer en supprimant le groupe de ressources ou en supprimant simplement le serveur MySQL. Pour supprimer le groupe de ressources, effectuez ces étapes :

1. Depuis le portail Azure, recherchez et sélectionnez **Groupes de ressources**.
1. Dans la liste des groupes de ressources, sélectionnez le nom de votre groupe de ressources.
1. Dans la page **Vue d’ensemble** de votre groupe de ressources, sélectionnez **Supprimer le groupe de ressources**.
1. Dans la boîte de dialogue de confirmation, entrez le nom de votre groupe de ressources, puis sélectionnez **Supprimer**.

Pour supprimer le serveur, vous pouvez sélectionner **Supprimer** dans la page **Vue d’ensemble** de votre serveur, comme indiqué ici :

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/quickstart-create-server-portal/delete-server.png" alt-text="Capture d’écran montrant comment supprimer un serveur.":::

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Créer une application web PHP (Laravel) avec MySQL](tutorial-php-database-app.md)