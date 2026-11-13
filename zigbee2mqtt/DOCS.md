# Appairage

Par défaut, l'application a `permit_join` réglé sur `false`. Pour permettre aux appareils de se connecter, vous devez l'activer une fois l'application démarrée. Vous pouvez désormais utiliser l'[interface intégrée](https://www.zigbee2mqtt.io/information/frontend.html) pour cela. Pour les détails sur l'activation de l'interface intégrée, voir la section suivante.

# Activer l'interface intégrée

Activez `ingress` pour disposer de l'interface dans votre UI : **Paramètres → Apps → Zigbee2MQTT → Afficher dans la barre latérale**. Vous trouverez plus de détails sur cette fonctionnalité dans la [documentation de Zigbee2MQTT](https://www.zigbee2mqtt.io/information/frontend.html).

# Configuration

## Onboarding (première configuration)

L'[onboarding](https://www.zigbee2mqtt.io/guide/getting-started/#onboarding) permet de configurer Zigbee2MQTT sans avoir à saisir manuellement les informations dans la page de configuration de l'application. Lorsque l'application démarre avec une installation toute neuve (aucune configuration présente), l'interface affiche une page de configuration rapide qui vous permet de choisir les différents paramètres nécessaires au démarrage de Zigbee2MQTT.

> [!NOTE]
> La détection des adaptateurs proposés dans la liste peut varier selon votre installation/réseau. Il se peut que vous deviez saisir ces [informations manuellement](https://www.zigbee2mqtt.io/guide/configuration/adapter-settings.html#basic-configuration) sur la page à la place.

> [!TIP]
> Vous pouvez forcer le ré-lancement de l'onboarding (par ex. pour changer d'adaptateur) à l'aide de l'interrupteur disponible dans la page de configuration de l'application (visible après avoir coché `Afficher les options de configuration facultatives inutilisées`). Cela forcera l'exécution de l'onboarding même après une première configuration réussie. Pensez à le désactiver une fois terminé.

## Manuelle

La configuration requise pour démarrer Zigbee2MQTT est disponible dans la configuration de l'application. Les autres options peuvent être configurées via l'interface de Zigbee2MQTT.

> [!CAUTION]
> Les paramètres définis via la page de configuration de l'application ont priorité sur ceux du fichier `configuration.yaml` (par ex. si vous définissez `rtscts: false` dans la page de configuration de l'application et `rtscts: true` dans `configuration.yaml`, c'est `rtscts: false` qui sera utilisé). _Si vous voulez contrôler toute la configuration via YAML, retirez-les de la page de configuration de l'application._

#### Exemples pour chaque section de configuration

- socat
  ```yaml
  enabled: false
  master: pty,raw,echo=0,link=/tmp/ttyZ2M,mode=777
  slave: tcp-listen:8485,keepalive,nodelay,reuseaddr,keepidle=1,keepintvl=1,keepcnt=5
  options: "-d -d"
  log: false
  ```
- mqtt
  ```yaml
  server: mqtt://localhost:1883
  user: my_user
  password: "my_password"
  ```
- serial
  ```yaml
  adapter: zstack
  port: /dev/serial/by-id/usb-Texas_Instruments_TI_CC2531_USB_CDC___0X00124B0018ED3DDF-if00
  ```

# Sauvegarde de la configuration

L'application crée une sauvegarde de votre configuration.yml dans votre dossier de données : `$DATA_PATH/configuration.yaml.bk`. Lors d'une mise à jour, utilisez-la pour renseigner les valeurs pertinentes dans votre nouvelle configuration, en particulier la clé réseau, afin d'éviter de casser votre réseau et de devoir ré-appairer tous vos appareils.
La sauvegarde de votre configuration est créée au démarrage de l'application si aucune sauvegarde précédente n'a été trouvée.

# Activer le watchdog

Pour redémarrer automatiquement Zigbee2MQTT en cas de panne logicielle (comme « adapter disconnected »), le watchdog peut être utilisé. Il s'active en ajoutant ceci à la configuration de l'application :

```yaml
watchdog: default
```

Cela utilisera les délais de nouvelle tentative par défaut du watchdog : 1 min, 5 min, 15 min, 30 min, 60 min. Des délais personnalisés sont également possibles, par ex. `watchdog: 5,10,30` démarrera Zigbee2MQTT avec des délais de nouvelle tentative de 5 min, 10 min et 30 min. Pour plus d'informations sur le watchdog, consultez la [documentation](https://www.zigbee2mqtt.io/guide/installation/15_watchdog.html).

# Ajouter la prise en charge de nouveaux appareils

Si vous souhaitez ajouter la prise en charge de nouveaux appareils à Zigbee2MQTT, consultez [Comment prendre en charge de nouveaux appareils](https://www.zigbee2mqtt.io/how_tos/how_to_support_new_devices.html).

# Notes

- Selon votre configuration, la config du serveur MQTT peut devoir inclure le port, généralement `1883` ou `8883` pour les communications SSL. Par exemple, `mqtt://core-mosquitto:1883` pour l'app Mosquitto de Home Assistant.
- Pour savoir quels ports série vous avez exposés, allez dans **Paramètres → Système → Matériel → ⋮ → Matériel du système** et saisissez le nom de l'appareil dans la barre de recherche.
- Le répertoire `/ssl` de Home Assistant est monté en lecture seule, donc les certificats qui s'y trouvent (par exemple ceux gérés par l'addon Let's Encrypt) peuvent être référencés à la fois dans la configuration MQTT (`key: /ssl/privkey.pem`, `cert: /ssl/fullchain.pem`) et dans les paramètres de l'interface (`ssl_key` / `ssl_cert`). Comme il est en lecture seule, les renouvellements effectués par l'addon gestionnaire sont pris en compte sans copier de fichiers.

# Socat

Dans certains cas, il n'est pas possible de transférer un appareil série vers le conteneur dans lequel tourne zigbee2mqtt. Cela peut être dû au fait que l'appareil n'est pas du tout branché physiquement à la machine.

Socat peut être utilisé pour transférer un appareil série via TCP vers zigbee2mqtt. Voir les [pages de manuel de socat](https://linux.die.net/man/1/socat) pour plus d'informations.

Vous pouvez configurer le module socat dans la section socat avec les options suivantes :

- `enabled` true/false pour activer socat (par défaut : false)
- `master` adresse master ou première adresse utilisée dans la ligne de commande socat (obligatoire)
- `slave` adresse slave ou seconde adresse utilisée dans la ligne de commande socat (obligatoire)
- `options` options supplémentaires ajoutées à la ligne de commande socat (facultatif)
- `log` true/false pour enregistrer la sortie stdout/stderr de socat dans data_path/socat.log (par défaut : false)

**NOTE :** Vous devrez modifier à la fois les options `master` et `slave` selon vos besoins. Les valeurs par défaut font en sorte que socat écoute sur le port `8485` et redirige sa sortie vers `/dev/ttyZ2M`. Le paramètre de port série de zigbee2mqtt n'est PAS défini automatiquement et doit être modifié en conséquence.
