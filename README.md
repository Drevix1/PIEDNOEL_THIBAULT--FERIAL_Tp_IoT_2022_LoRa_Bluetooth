Explication générale du code

Ce code est destiné à un microcontrôleur ESP32 utilisant FreeRTOS pour gérer des tâches asynchrones. Il met en place une connexion Wi-Fi, un client MQTT pour la communication avec un broker distant, et une communication LoRa pour envoyer des messages à longue distance.

Les fonctionnalités principales du code sont :

    Connexion au Wi-Fi : Se connecte à un réseau Wi-Fi défini par WIFI_SSID et WIFI_PASS.
    Client MQTT : Se connecte à un broker MQTT (mqtt://test.mosquitto.org), publie un message et s'abonne à un topic.
    Réception de messages MQTT et transmission via LoRa : Dès qu'un message MQTT est reçu, il est immédiatement retransmis via LoRa.
    Configuration de LoRa : Initialise LoRa sur la fréquence 868 MHz, active le CRC, et configure les paramètres de transmission (facteur d'étalement, largeur de bande, taux de codage).
    Tâche de réception LoRa : Écoute en continu les messages LoRa reçus.
    Tâche de transmission LoRa (optionnelle) : Peut envoyer un message LoRa périodiquement (désactivée par défaut).

Explication détaillée des fonctions
1. void mqtt_event_handler(void *handler_args, esp_event_base_t base, int32_t event_id, void *event_data)

Cette fonction est un gestionnaire d'événements MQTT. Elle est appelée automatiquement lorsque le client MQTT reçoit un événement.

    MQTT_EVENT_CONNECTED :
        Une fois connecté au broker MQTT, l'ESP32 publie immédiatement un message (MQTT_PWD) sur le topic MQTT_TOPIC.
        Ensuite, il s'abonne au topic MQTT_TOPIC2.

    MQTT_EVENT_DISCONNECTED :
        Affiche un message indiquant la déconnexion.

    MQTT_EVENT_DATA :
        Lorsque l'ESP32 reçoit un message MQTT, il est immédiatement envoyé via LoRa (lora_send_packet()).
        Cela permet de relayer les messages MQTT vers un réseau LoRa.

2. void mqtt_init()

    Configure et initialise le client MQTT (esp_mqtt_client_config_t).
    Enregistre le gestionnaire mqtt_event_handler pour traiter les événements MQTT.
    Démarre le client MQTT (esp_mqtt_client_start()).

3. void wifi_event_handler(void *arg, esp_event_base_t event_base, int32_t event_id, void *event_data)

Gère les événements Wi-Fi :

    WIFI_EVENT_STA_START : Se connecte au Wi-Fi (esp_wifi_connect()).
    WIFI_EVENT_STA_DISCONNECTED : En cas de déconnexion, tente de se reconnecter automatiquement.
    IP_EVENT_STA_GOT_IP : Une fois connecté et une adresse IP obtenue, le client MQTT est démarré (mqtt_init()).

4. void wifi_init()

    Initialise NVS (Non-Volatile Storage) pour stocker les configurations.
    Configure le Wi-Fi en mode station (STA).
    Définit les identifiants Wi-Fi (SSID et password).
    Démarre le Wi-Fi et enregistre les gestionnaires d'événements.

5. void task_rx(void *pvParameters)

Cette tâche tourne en continu et écoute les messages LoRa entrants :

    Active le mode réception LoRa (lora_receive()).
    Vérifie si un message a été reçu (lora_received()).
    Si un message est reçu, il est affiché avec sa longueur et son contenu (ESP_LOGI()).
    Petite pause (vTaskDelay(1)) pour éviter les alertes du Watchdog Timer.

6. void task_tx(void *pvParameters)

    Cette fonction envoie un message LoRa contenant "Hello LoRa".
    Ajoute une pause de 5 secondes (vTaskDelay(pdMS_TO_TICKS(5000))) avant un éventuel prochain envoi.


La fonction principale exécutée au démarrage du programme :

    Initialise le Wi-Fi (wifi_init()).
    Vérifie si LoRa est détecté (lora_init()). Si ce n'est pas le cas, elle entre dans une boucle infinie.
    Configure LoRa :
        Fréquence = 868 MHz.
        Active le CRC.
        Configure le taux de codage, la largeur de bande et le facteur d'étalement.
    Crée la tâche de réception LoRa (task_rx).
    (Optionnel) : La tâche de transmission LoRa (task_tx) est désactivée.
