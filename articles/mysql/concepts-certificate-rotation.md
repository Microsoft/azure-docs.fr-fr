---
title: Rotation de certificat pour Azure Database pour MySQL
description: En savoir plus sur les modifications à venir dues aux changements de certificat racine qui affectent Azure Database for MySQL
author: mksuni
ms.author: sumuth
ms.service: mysql
ms.topic: conceptual
ms.date: 09/02/2020
ms.openlocfilehash: 58143bde757a44cde4e4237715823c8556ac5e12
ms.sourcegitcommit: 99955130348f9d2db7d4fb5032fad89dad3185e7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93348438"
---
# <a name="understanding-the-changes-in-the-root-ca-change-for-azure-database-for-mysql"></a>Comprendre les modifications liées au changement d’autorité de certification racine pour Azure Database for MySQL

Azure Database pour MySQL va modifier le certificat racine d’application cliente/de pilote activé avec SSL et que vous utilisez pour [vous connecter au serveur de base de données](concepts-connectivity-architecture.md). Le certificat racine actuellement disponible est configuré pour expirer le 15 février 2021 (15/02/2021) dans le cadre de la maintenance standard et des meilleures pratiques pour la sécurité. Cet article vous donne plus de détails sur les modifications à venir, les ressources qui seront affectées et les étapes nécessaires pour garantir que votre application maintient sa connectivité à votre serveur de base de données.

>[!NOTE]
> En fonction des commentaires des clients 2020, nous avons étendu la désapprobation du certificat racine pour notre autorité de certification racine Baltimore existante du 26 octobre au 15 février 2021. Nous espérons que cette extension fournira suffisamment de temps pour permettre à nos utilisateurs d’implémenter les modifications du client si elles sont affectées.

## <a name="what-update-is-going-to-happen"></a>Quelle mise à jour va se produire ?

Dans certains cas, les applications utilisent un fichier de certificat local généré à partir du fichier de certificat d’une autorité de certification approuvée pour se connecter de façon sécurisée. Actuellement, les clients peuvent uniquement utiliser le certificat prédéfini pour se connecter à un serveur Azure Database pour MySQL, situé [ici](https://www.digicert.com/CACerts/BaltimoreCyberTrustRoot.crt.pem). Toutefois, le [forum Certificate Authority (CA) Browser](https://cabforum.org/)  a récemment publié des rapports indiquant que plusieurs certificats émis par les fournisseurs d’autorité de certification n’étaient pas conformes.

Conformément aux exigences de conformité du secteur, les fournisseurs d’autorité de certification ont commencé à révoquer les certificats d’autorité de certification non conformes, en demandant aux serveurs d’utiliser des certificats émis par des autorités de certification conformes et signés par des certificats d’autorité de certification provenant de ces dernières. Étant donné qu’Azure Database for MySQL utilise actuellement l’un de ces certificats non conformes utilisés par les applications clientes pour valider leurs connexions SSL, nous devons veiller à ce que les actions appropriées soient prises (décrites ci-dessous) afin de réduire au minimum l’impact potentiel sur vos serveurs MySQL.

Le nouveau certificat sera utilisé à partir du 15 février 2021 (02/15/2021). Si vous utilisez la validation de l’autorité de certification ou la validation complète du certificat de serveur lors de la connexion à partir d’un client MySQL (sslmode = Verify-ca ou sslmode = Verify-Full), vous devez mettre à jour la configuration de votre application avant le 15 février 2021 (03/15/2021).

## <a name="how-do-i-know-if-my-database-is-going-to-be-affected"></a>Comment savoir si ma base de données va être affectée ?

Toutes les applications qui utilisent SSL/TLS et vérifient le certificat racine doivent mettre à jour le certificat racine. Vous pouvez déterminer si vos connexions vérifient le certificat racine en examinant votre chaîne de connexion.
-    Si votre chaîne de connexion inclut `sslmode=verify-ca` ou `sslmode=verify-identity`, vous devez mettre à jour le certificat.
-    Si votre chaîne de connexion inclut `sslmode=disable`, `sslmode=allow`, `sslmode=prefer` ou `sslmode=require`, vous n’avez pas besoin de mettre à jour les certificats. 
-  Si vous utilisez des connecteurs Java et que votre chaîne de connexion comprend useSSL=false ou requireSSL=false, vous n’avez pas besoin de mettre à jour les certificats.
-    Si votre chaîne de connexion ne spécifie pas sslmode, vous n’avez pas besoin de mettre à jour les certificats.

Si vous utilisez un client qui abstrait la chaîne de connexion, consultez la documentation du client pour savoir s’il vérifie les certificats.
Pour comprendre le mode sslmode Azure Database for MySQL, consultez les [descriptions de mode SSL](concepts-ssl-connection-security.md#ssl-default-settings).

Pour éviter toute interruption de la disponibilité de votre application en raison de la révocation inattendue de certificats, ou pour mettre à jour un certificat qui a été révoqué, reportez-vous à la section [**Que dois-je faire pour maintenir la connectivité**](concepts-certificate-rotation.md#what-do-i-need-to-do-to-maintain-connectivity).

## <a name="what-do-i-need-to-do-to-maintain-connectivity"></a>Procédure à suivre pour maintenir la connectivité

Pour éviter toute interruption de la disponibilité de votre application en raison de la révocation inattendue de certificats, ou pour mettre à jour un certificat qui a été révoqué, suivez les étapes ci-dessous. L’idée est de créer un fichier *.pem* , qui combine le certificat actuel et le nouveau, et pendant la validation du certificat SSL, une des valeurs autorisées sera utilisée. Reportez-vous aux étapes ci-dessous :

*   Téléchargez l’autorité de certification racine BaltimoreCyberTrustRootDigiCertGlobalRootG2 à partir des liens ci-dessous :
    *   https://www.digicert.com/CACerts/BaltimoreCyberTrustRoot.crt.pem
    *   https://cacerts.digicert.com/DigiCertGlobalRootG2.crt.pem

*   Générez un magasin de certificats d’autorité de certification combiné où les certificats **BaltimoreCyberTrustRoot** et **DigiCertGlobalRootG2** sont inclus.
    *   Pour les utilisateurs Java (MySQL Connector/J), exécutez :

          ```console
          keytool -importcert -alias MySQLServerCACert  -file D:\BaltimoreCyberTrustRoot.crt.pem  -keystore truststore -storepass password -noprompt
          ```

          ```console
          keytool -importcert -alias MySQLServerCACert2  -file D:\DigiCertGlobalRootG2.crt.pem -keystore truststore -storepass password  -noprompt
          ```

          Remplacez ensuite le fichier de magasin de clés d’origine par le nouveau fichier généré :
        *   System.setProperty("javax.net.ssl.trustStore","path_to_truststore_file"); 
        *   System.setProperty("javax.net.ssl.trustStorePassword","password");

    *   Si vous utilisez .NET (MySQL Connector/NET, MySQLConnector), assurez-vous que **BaltimoreCyberTrustRoot** et **DigiCertGlobalRootG2** existent dans le magasin de certificats Windows et dans les autorités de certification racines de confiance. S’il n’existe aucun certificat, importez le certificat manquant.

        ![Certificat .net Azure Database pour MySQL](media/overview/netconnecter-cert.png)

    *   Si vous utilisez .NET sur Linux en utilisant SSL_CERT_DIR, assurez-vous que **BaltimoreCyberTrustRoot** et **DigiCertGlobalRootG2** existent tous deux dans le répertoire indiqué par SSL_CERT_DIR. Si des certificats n’existent pas, créez le fichier de certificat manquant.

    *   Si vous utilisez d’autres solutions (MySQL Client/MySQL Workbench/C/C++/Go/Python/Ruby/PHP/NodeJS/Perl/Swift), vous pouvez fusionner deux fichiers de certificat d’autorité de certification comme suit</b>

        </br>-----BEGIN CERTIFICATE-----
 </br>(Root CA1: BaltimoreCyberTrustRoot.crt.pem)
 </br>-----END CERTIFICATE-----
 </br>-----BEGIN CERTIFICATE-----
 </br>(Root CA2: DigiCertGlobalRootG2.crt.pem)
 </br>-----END CERTIFICATE-----

*   Remplacez le fichier .pem d’origine de l’autorité de certification racine par le fichier d’autorité de certification racine combiné et redémarrez votre application/client.
*    Ensuite, après le déploiement du nouveau certificat côté serveur, vous pouvez remplacer votre fichier .pem d’autorité de certification par DigiCertGlobalRootG2.crt.pem.

## <a name="what-can-be-the-impact-of-not-updating-the-certificate"></a>Quel peut être l’impact de l’absence de mise à jour du certificat ?
Si vous utilisez le certificat Azure Database pour MySQL émis, comme indiqué ici, la disponibilité de votre application peut être interrompue, car la base de données ne sera pas accessible. Selon votre application, vous pouvez recevoir différents messages d’erreur, notamment :
*    Certificat non valide/certificat révoqué
*    Délai de connexion dépassé

> [!NOTE]
> Ne supprimez pas ou ne modifiez pas **certificat Baltimore** tant que le changement de certificat n’est pas effectué. Nous enverrons une communication une fois la modification effectuée, après quoi il est possible de supprimer le certificat Baltimore. 

## <a name="frequently-asked-questions"></a>Forum aux questions

###    <a name="1-if-i-am-not-using-ssltls-do-i-still-need-to-update-the-root-ca"></a>1. Si je n’utilise pas SSL/TLS, dois-je quand même mettre à jour l’autorité de certification racine ?
Aucune action n’est requise si vous n’utilisez pas SSL/TLS. 

### <a name="2-if-i-am-using-ssltls-do-i-need-to-restart-my-database-server-to-update-the-root-ca"></a>2. Si j’utilise SSL/TLS, dois-je redémarrer mon serveur de base de données pour mettre à jour l’autorité de certification racine ?
Non, vous n’avez pas besoin de redémarrer le serveur de base de données pour commencer à utiliser le nouveau certificat. Ce certificat racine est une modification côté client et les connexions clientes entrantes doivent utiliser le nouveau certificat pour s’assurer qu’elles peuvent se connecter au serveur de base de données.

### <a name="3-what-will-happen-if-i-do-not-update-the-root-certificate-before-february-15-2021-02152021"></a>3. Que se passe-t-il si je ne mets pas à jour le certificat racine avant le 15 février 2021 (15/02/2021) ?
Si vous ne mettez pas à jour le certificat racine avant le 15 février 2021 (02/15/2021), vos applications qui se connectent via SSL/TLS et effectuent une vérification pour le certificat racine ne pourront pas communiquer avec le serveur de base de données MySQL, et l’application rencontrera des problèmes de connectivité à votre serveur de base de données MySQL.

### <a name="4-what-is-the-impact-if-using-app-service-with-azure-database-for-mysql"></a>4. Quel est l’impact de l’utilisation d’App Service avec Azure Database for MySQL ?
Pour les services Azure App Service qui se connectent à Azure Database for MySQL, deux scénarios sont possibles et dépendent de la façon dont vous utilisez SSL avec votre application.
*   Ce nouveau certificat a été ajouté à App Service au niveau de la plateforme. Si vous utilisez les certificats SSL inclus sur la plateforme App Service dans votre application, aucune action n’est nécessaire.
*   Si vous incluez explicitement le chemin d’accès au fichier de certificat SSL dans votre code, vous devez télécharger le nouveau certificat et mettre à jour le code de façon à ce qu’il utilise le nouveau certificat. Un bon exemple de ce scénario est lorsque vous utilisez des conteneurs personnalisés dans App Service comme partagés dans la[documentation App Service](../app-service/tutorial-multi-container-app.md#configure-database-variables-in-wordpress).

### <a name="5-what-is-the-impact-if-using-azure-kubernetes-services-aks-with-azure-database-for-mysql"></a>5. Quel est l’impact de l’utilisation d’Azure Kubernetes Services (AKS) avec Azure Database for MySQL ?
Si vous essayez de vous connecter au serveur Azure Database for MySQL à l’aide d’Azure Kubernetes Services (AKS), cela est similaire à un accès à partir d’un environnement d’hôte de clients dédié. Reportez-vous aux étapes indiquées [ici](../aks/ingress-own-tls.md).

### <a name="6-what-is-the-impact-if-using-azure-data-factory-to-connect-to-azure-database-for-mysql"></a>6. Quel est l’impact de l’utilisation d’Azure Data Factory pour se connecter à Azure Database for MySQL ?
Le connecteur qui utilise Azure Integration Runtime exploite les certificats du magasin de certificats Windows dans l’environnement hébergé par Azure. Comme ces certificats sont déjà compatibles avec les certificats récemment appliqués, aucune action n’est nécessaire.

Pour le connecteur utilisant le runtime d’intégration auto-hébergé où vous incluez explicitement le chemin d’accès au fichier de certificat SSL dans votre chaîne de connexion, vous devez télécharger le [nouveau certificat](https://cacerts.digicert.com/DigiCertGlobalRootG2.crt.pem) et mettre à jour la chaîne de connexion de façon à ce qu’elle l’utilise.

### <a name="7-do-i-need-to-plan-a-database-server-maintenance-downtime-for-this-change"></a>7. Dois-je prévoir un temps d’arrêt lié à la maintenance du serveur de base de données pour effectuer cette modification ?
Non. Étant donné que la modification est uniquement côté client pour la connexion au serveur de base de données, aucun temps d’arrêt lié à la maintenance n’est nécessaire pour le serveur de base de données lors de cette modification.

### <a name="8--what-if-i-cannot-get-a-scheduled-downtime-for-this-change-before-february-15-2021-02152021"></a>8.  Que se passe-t-il si je ne peux pas obtenir un temps d’arrêt planifié pour cette modification avant le 15 février 2021 (15/02/2021) ?
Étant donné que les clients utilisés pour la connexion au serveur doivent mettre à jour les informations de certificat comme décrit dans la section de correction disponible [ici](./concepts-certificate-rotation.md#what-do-i-need-to-do-to-maintain-connectivity), aucun temps d’arrêt n’est requis pour le serveur dans ce cas.

### <a name="9-if-i-create-a-new-server-after-february-15-2021-02152021-will-i-be-impacted"></a>9. Si je crée un serveur après le 15 février 2021 (15/02/2021), serai-je concerné ?
Pour les serveurs créés après le 15 février 2021 (15/02/2021), vous pouvez utiliser le certificat nouvellement émis pour que vos applications se connectent à l’aide de SSL.

###    <a name="10-how-often-does-microsoft-update-their-certificates-or-what-is-the-expiry-policy"></a>10. Quelle est la fréquence à laquelle Microsoft met à jour ses certificats ou quelle est la stratégie d’expiration ?
Les certificats utilisés par Azure Database for MySQL sont fournis par des autorités de certification approuvées. Par conséquent, la prise en charge de ces certificats sur Azure Database for MySQL est liée à la prise en charge de ces certificats par l’autorité de certification. Toutefois, comme dans le cas présent, il peut y avoir des bogues imprévus dans ces certificats prédéfinis et ceux-ci doivent être corrigés au plus tôt.

###    <a name="11-if-i-am-using-read-replicas-do-i-need-to-perform-this-update-only-on-source-server-or-the-read-replicas"></a>11. Si j’utilise des réplicas en lecture, dois-je effectuer cette mise à jour uniquement sur le serveur source ou sur les réplicas de lecture ?
Étant donné que cette mise à jour est une modification côté client, si le client avait l’habitude de lire les données du serveur de réplica, vous devez également appliquer les modifications pour ces clients.

###    <a name="12-if-i-am-using-data-in-replication-do-i-need-to-perform-any-action"></a>12. Si j'utilise la réplication des données entrantes, dois-je effectuer une action particulière ?
Si vous utilisez la [réplication des données entrantes](concepts-data-in-replication.md) pour vous connecter à Azure Database pour MySQL, deux éléments sont à prendre en compte :
*   Si la réplication des données s'effectue entre une machine virtuelle (locale ou Azure) et Azure Database pour MySQL, vous devez vérifier si SSL est utilisé pour créer le réplica. Exécutez **SHOW SLAVE STATUS** et vérifiez le paramètre suivant.  

    ```azurecli-interactive
    Master_SSL_Allowed            : Yes
    Master_SSL_CA_File            : ~\azure_mysqlservice.pem
    Master_SSL_CA_Path            :
    Master_SSL_Cert               : ~\azure_mysqlclient_cert.pem
    Master_SSL_Cipher             :
    Master_SSL_Key                : ~\azure_mysqlclient_key.pem
    ```

    Si vous voyez que le certificat est fourni pour CA_file, SSL_Cert et SSL_Key, vous devez mettre à jour le fichier en ajoutant le [nouveau certificat](https://cacerts.digicert.com/DigiCertGlobalRootG2.crt.pem).

*   Si la réplication de données se fait entre deux instances Azure Database pour MySQL, vous devez réinitialiser le réplica en exécutant **CALL mysql.az_replication_change_master** et en fournissant le nouveau certificat racine double en tant que dernier paramètre [master_ssl_ca](howto-data-in-replication.md#link-source-and-replica-servers-to-start-data-in-replication)

### <a name="13-do-we-have-server-side-query-to-verify-if-ssl-is-being-used"></a>13. Existe-t-il une requête côté serveur pour vérifier si SSL est utilisé ?
Pour vérifier si vous utilisez une connexion SSL pour vous connecter au serveur, consultez [Vérification SSL](howto-configure-ssl.md#step-4-verify-the-ssl-connection).

### <a name="14-is-there-an-action-needed-if-i-already-have-the-digicertglobalrootg2-in-my-certificate-file"></a>14. Une action est-elle nécessaire si je dispose déjà de DigiCertGlobalRootG2 dans mon fichier de certificat ?
Non. Aucune action n'est nécessaire si votre fichier de certificat contient déjà **DigiCertGlobalRootG2**.

###    <a name="15-what-if-i-have-further-questions"></a>15. Que se passe-t-il si j’ai d’autres questions ?
Si vous avez des questions, posez-les aux experts de la communauté dans [Microsoft Q&A](mailto:AzureDatabaseforMySQL@service.microsoft.com). Si vous disposez d'un plan de support et que vous avez besoin d'une aide technique, [contactez-nous](mailto:AzureDatabaseforMySQL@service.microsoft.com).
