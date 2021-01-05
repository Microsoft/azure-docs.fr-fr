---
title: Connecter un service Azure App Service - Azure Database pour MySQL
description: Instructions pour connecter correctement un service Azure App Service existant à une base de données Azure pour MySQL
author: savjani
ms.author: pariks
ms.service: mysql
ms.topic: how-to
ms.date: 3/18/2020
ms.openlocfilehash: 6355afe6ce5decbed029db4536b1b1b19f5a876c
ms.sourcegitcommit: 6ab718e1be2767db2605eeebe974ee9e2c07022b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94541502"
---
# <a name="connect-an-existing-azure-app-service-to-azure-database-for-mysql-server"></a>Connecter un service Azure App Service existant à un serveur de base de données Azure pour MySQL
Cette rubrique explique comment connecter un service Azure App Service existant à votre serveur de base de données Azure pour MySQL.

## <a name="before-you-begin"></a>Avant de commencer
Connectez-vous au [portail Azure](https://portal.azure.com). Créez un serveur de base de données Azure pour MySQL. Pour plus d’informations, reportez-vous à [Guide pratique pour créer un serveur de base de données Azure pour MySQL à partir du portail](quickstart-create-mysql-server-database-using-azure-portal.md) ou à [Guide pratique pour créer un serveur de base de données Azure pour MySQL à l’aide de l’interface CLI](quickstart-create-mysql-server-database-using-azure-cli.md).

Il existe actuellement deux solutions pour activer l’accès à partir d’un service Azure App Service vers une base de données Azure pour MySQL. Ces deux solutions impliquent de configurer des règles de pare-feu au niveau du serveur.

## <a name="solution-1---allow-azure-services"></a>Solution 1 - Autoriser les services Azure
La base de données Azure pour MySQL fournit une sécurité d’accès à l’aide d’un pare-feu afin de protéger vos données. Lors de la connexion d’un service Azure App Service à une base de données Azure pour MySQL, gardez à l’esprit que les adresses IP sortantes d’App Service sont dynamiques par nature. Choisir l’option « Autoriser l’accès aux services Azure » permet au service d’application de se connecter au serveur MySQL.

1. Dans le panneau du serveur MySQL, sous le titre Paramètres, cliquez sur **Sécurité de la connexion** pour ouvrir le panneau Sécurité de la connexion pour la base de données Azure pour MySQL.

   :::image type="content" source="./media/howto-connect-webapp/1-connection-security.png" alt-text="Portail Azure - cliquez sur Sécurité des connexions":::

2. Sélectionnez **ON** dans **Autoriser l’accès aux services Azure**, puis **Enregistrer**.
   :::image type="content" source="./media/howto-connect-webapp/allow-azure.png" alt-text="Portail Azure - Autoriser l’accès à Azure":::

## <a name="solution-2---create-a-firewall-rule-to-explicitly-allow-outbound-ips"></a>Solution 2 - Créer une règle de pare-feu pour autoriser explicitement des adresses IP sortantes
Vous pouvez ajouter explicitement toutes les adresses IP sortantes de votre service Azure App Service.

1. Dans le panneau Propriétés d’App Service, affichez votre **ADRESSE IP SORTANTE**.

   :::image type="content" source="./media/howto-connect-webapp/2_1-outbound-ip-address.png" alt-text="Portail Azure - Afficher des adresses IP sortantes":::

2. Dans le panneau Sécurité de la connexion de MySQL, ajoutez une par une des adresses IP sortantes.

   :::image type="content" source="./media/howto-connect-webapp/2_2-add-explicit-ips.png" alt-text="Portail Azure - Ajouter des adresses IP explicites":::

3. N’oubliez pas d’**Enregistrer** vos règles de pare-feu.

Bien que le service Azure App Service tente de maintenir les adresses IP constantes au fil du temps, il existe des cas où les adresses IP peuvent changer. Par exemple, cela peut se produire quand l’application est recyclée, qu’une opération de mise à l’échelle se produit ou que de nouveaux ordinateurs sont ajoutés dans des centres de données régionaux Azure pour accroître la capacité. Quand les adresses IP changent, l’application peut subir des temps d’arrêt s’il ne peut plus se connecter au serveur MySQL. Gardez cet élément de considération à l’esprit au moment de choisir l’une des solutions précédentes.

## <a name="ssl-configuration"></a>Configuration SSL
SSL est activé par défaut dans la base de données Azure pour MySQL. Si votre application n’utilise pas le protocole SSL pour se connecter à la base de données, vous devez le désactiver sur le serveur MySQL. Pour plus d’informations sur la configuration de SSL, consultez [Utilisation de SSL avec une base de données Azure pour MySQL](howto-configure-ssl.md).

### <a name="django-pymysql"></a>Django (PyMySQL)
```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'quickstartdb',
        'USER': 'myadmin@mydemoserver',
        'PASSWORD': 'yourpassword',
        'HOST': 'mydemoserver.mysql.database.azure.com',
        'PORT': '3306',
        'OPTIONS': {
            'ssl': {'ssl-ca': '/var/www/html/BaltimoreCyberTrustRoot.crt.pem'}
        }
    }
}
```

## <a name="next-steps"></a>Étapes suivantes
Pour plus d’informations sur les chaînes de connexion, reportez-vous à [Chaînes de connexions](howto-connection-string.md).
