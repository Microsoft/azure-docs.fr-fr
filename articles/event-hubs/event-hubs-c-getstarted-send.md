---
title: 'Démarrage rapide : Envoyer des événements avec C – Azure Event Hubs'
description: 'Démarrage rapide : Cet article décrit la procédure à suivre pour créer une application C qui envoie des événements à Azure Event Hubs.'
ms.topic: quickstart
ms.date: 06/23/2020
ms.openlocfilehash: bfe1ca1a45f7b33d7431aed13446d8d72f79fb90
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "85315660"
---
# <a name="quickstart-send-events-to-azure-event-hubs-using-c"></a>Démarrage rapide : Envoyer des événements vers Azure Event Hubs avec C

## <a name="introduction"></a>Introduction
Azure Event Hubs est une plateforme de diffusion de données volumineuses et un service d’ingestion d’événements, capable de recevoir et de traiter des millions d’événements par seconde. Les concentrateurs d’événements peuvent traiter et stocker des événements, des données ou la télémétrie produits par des logiciels et appareils distribués. Les données envoyées à un concentrateur d’événements peuvent être transformées et stockées à l’aide d’adaptateurs de traitement par lot/stockage ou d’un fournisseur d’analyse en temps réel. Pour une présentation détaillée d’Event Hubs, consultez [Vue d’ensemble d’Event Hubs](event-hubs-about.md) et [Fonctionnalités d’Event Hubs](event-hubs-features.md).

Ce didacticiel explique comment envoyer des événements vers un Event Hub à l’aide d’une application console en C. 

## <a name="prerequisites"></a>Prérequis
Pour réaliser ce didacticiel, vous avez besoin des éléments suivants :

* Un environnement de développement en C. Ce didacticiel présuppose que la pile GCC est sur une machine virtuelle Linux Azure dotée du système d’exploitation Ubuntu 14.04.
* [Microsoft Visual Studio](https://www.visualstudio.com/).
* **Créez un espace de noms Event Hubs et un Event Hub**. Utilisez le [portail Azure](https://portal.azure.com) pour créer un espace de noms de type Event Hubs et obtenez les informations de gestion nécessaires à votre application pour communiquer avec le hub d’événements. Pour créer un espace de noms et un hub d’événements, suivez la procédure décrite dans [cet article](event-hubs-create.md). Obtenez la valeur de la clé d’accès du hub d’événements en suivant les instructions de l’article : [Obtenir la chaîne de connexion](event-hubs-get-connection-string.md#get-connection-string-from-the-portal). Vous utilisez la clé d’accès dans le code que vous écrivez plus loin dans ce tutoriel. Le nom de clé par défaut est : **RootManageSharedAccessKey**.

## <a name="write-code-to-send-messages-to-event-hubs"></a>Écrire du code pour envoyer des messages à Event Hubs
Cette section explique comment écrire une application en C pour envoyer des événements à votre hub d’événements. Le code utilise la bibliothèque Proton AMQP du [projet Apache Qpid](https://qpid.apache.org/). Cette approche est similaire à l’utilisation de rubriques et de files d’attente Service Bus avec AMQP en partant du langage C comme indiqué dans [cet exemple](https://code.msdn.microsoft.com/Using-Apache-Qpid-Proton-C-afd76504). Pour plus d’informations, consultez la [Documentation Qpid Proton](https://qpid.apache.org/proton/index.html).

1. Dans la [page Qpid AMQP Messenger](https://qpid.apache.org/proton/messenger.html), suivez les instructions d’installation de Qpid Proton correspondant à votre environnement.
2. Pour compiler la bibliothèque Proton, installez les packages suivants :
   
    ```shell
    sudo apt-get install build-essential cmake uuid-dev openssl libssl-dev
    ```
3. Téléchargez la [bibliothèque Qpid Proton](https://qpid.apache.org/proton/index.html) et effectuez son extraction, par exemple :
   
    ```shell
    wget https://archive.apache.org/dist/qpid/proton/0.7/qpid-proton-0.7.tar.gz
    tar xvfz qpid-proton-0.7.tar.gz
    ```
4. Créez un répertoire de build, compilez-le et installez-le :
   
    ```shell
    cd qpid-proton-0.7
    mkdir build
    cd build
    cmake -DCMAKE_INSTALL_PREFIX=/usr ..
    sudo make install
    ```
5. Dans le répertoire de travail, créez un fichier nommé **sender.c** avec le code suivant. N’oubliez pas de remplacer les valeurs pour votre nom/clé SAS, votre nom de hub d’événements et votre espace de noms. Vous devez également remplacer une version codée URL de la clé pour le **SendRule** précédemment créé. Vous pouvez la coder par URL [ici](https://www.w3schools.com/tags/ref_urlencode.asp).
   
    ```c
    #include "proton/message.h"
    #include "proton/messenger.h"
   
    #include <getopt.h>
    #include <proton/util.h>
    #include <sys/time.h>
    #include <stddef.h>
    #include <stdio.h>
    #include <string.h>
    #include <unistd.h>
    #include <stdlib.h>
   
    #define check(messenger)                                                     \
      {                                                                          \
        if(pn_messenger_errno(messenger))                                        \
        {                                                                        \
          printf("check\n");                                                     \
          die(__FILE__, __LINE__, pn_error_text(pn_messenger_error(messenger))); \
        }                                                                        \
      }
   
    pn_timestamp_t time_now(void)
    {
      struct timeval now;
      if (gettimeofday(&now, NULL)) pn_fatal("gettimeofday failed\n");
      return ((pn_timestamp_t)now.tv_sec) * 1000 + (now.tv_usec / 1000);
    }  
   
    void die(const char *file, int line, const char *message)
    {
      printf("Dead\n");
      fprintf(stderr, "%s:%i: %s\n", file, line, message);
      exit(1);
    }
   
    int sendMessage(pn_messenger_t * messenger) {
        char * address = (char *) "amqps://{SAS Key Name}:{SAS key}@{namespace name}.servicebus.windows.net/{event hub name}";
        char * msgtext = (char *) "Hello from C!";
   
        pn_message_t * message;
        pn_data_t * body;
        message = pn_message();
   
        pn_message_set_address(message, address);
        pn_message_set_content_type(message, (char*) "application/octect-stream");
        pn_message_set_inferred(message, true);
   
        body = pn_message_body(message);
        pn_data_put_binary(body, pn_bytes(strlen(msgtext), msgtext));
   
        pn_messenger_put(messenger, message);
        check(messenger);
        pn_messenger_send(messenger, 1);
        check(messenger);
   
        pn_message_free(message);
    }
   
    int main(int argc, char** argv) {
        printf("Press Ctrl-C to stop the sender process\n");
   
        pn_messenger_t *messenger = pn_messenger(NULL);
        pn_messenger_set_outgoing_window(messenger, 1);
        pn_messenger_start(messenger);
   
        while(true) {
            sendMessage(messenger);
            printf("Sent message\n");
            sleep(1);
        }
   
        // release messenger resources
        pn_messenger_stop(messenger);
        pn_messenger_free(messenger);
   
        return 0;
    }
    ```
6. Compilez le fichier (en supposant que **gcc**est défini) :
   
    ```
    gcc sender.c -o sender -lqpid-proton
    ```

    > [!NOTE]
    > Ce code utilise une fenêtre sortante de 1 pour forcer l’envoi des messages dès que possible. Il est recommandé que votre application essaie d’envoyer les messages par lot pour augmenter le débit. Consultez la [page Qpid AMQP Messenger](https://qpid.apache.org/proton/messenger.html) pour plus d’informations sur l’utilisation de la bibliothèque Qpid Proton dans l’ensemble des environnements et à partir des plateformes pour lesquelles des liaisons sont fournies (actuellement Perl, PHP, Python et Ruby).

Utiliser l'application pour envoyer des messages à l’Event Hub 

Félicitations ! Vous venez d’envoyer des messages à un concentrateur d’événements.

## <a name="next-steps"></a>Étapes suivantes
Consultez les articles suivants :

- [EventProcessorHost](event-hubs-event-processor-host.md)
- [Fonctionnalités et terminologie dans Azure Event Hubs](event-hubs-features.md).


<!-- Images. -->
[21]: ./media/event-hubs-c-ephcs-getstarted/run-csharp-ephcs1.png
[24]: ./media/event-hubs-c-ephcs-getstarted/receive-eph-c.png
