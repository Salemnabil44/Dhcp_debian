# Installation et Configuration d'un Serveur DHCP sur Debian

## Prérequis
- Une image ISO de Debian 
- VirtualBox ou tout autre logiciel de virtualisation.
- Une machine virtuelle Debian pour le serveur DHCP.
- Une ou plusieurs machines virtuelles clients.

## Étape 1 : Installation de Debian

1. **Télécharger l'image ISO de Debian** 

2. **Créer une nouvelle machine virtuelle dans VirtualBox** :
    - **Nom** : Debian-DHCP
    - **Type** : Linux
    - **Version** : Debian (64-bit)
    - **Mémoire RAM** : 1024 MB (1 GB) recommandé
    - **Disque dur** : Créer un disque virtuel (20 GB recommandé)

3. **Configurer les paramètres réseau** :
    - **Adapter 1** : Choisir "Internal Network" dans le menu déroulant "Attached to".

4. **Démarrer la machine virtuelle** et sélectionner l'image ISO de Debian comme disque de démarrage.

5. **Suivre les étapes de l'installateur Debian** :
    - **Choisir la langue, le pays et la disposition du clavier**.
    - **Configurer le réseau** : Si demandé, configurez avec une adresse IP statique temporaire pour l'installation.
    - **Créer un utilisateur root et un utilisateur standard**.
    - **Partitionner le disque** (choisir les options par défaut ou personnaliser selon vos besoins).
    - **Installer le système de base**.
    - **Installer le gestionnaire de paquet** : Choisir les miroirs de téléchargement.
    - **Sélectionner et installer les logiciels** : Inclure "Serveur SSH" pour une gestion à distance.
    - **Installer le chargeur de démarrage GRUB**.

6. **Redémarrer la machine** après l'installation.

## Étape 2 : Configuration de l'Interface Réseau du Serveur DHCP

1. **Ouvrir un terminal** et se connecter en tant que root ou utiliser `sudo`.

2. **Configurer une adresse IP statique** pour l'interface réseau :
    ```bash
    sudo nano /etc/network/interfaces
    ```
    Ajouter les lignes suivantes pour `eth0` :
    ```plaintext
    auto eth0
    iface eth0 inet static
        address 172.20.0.1
        netmask 255.255.255.0
        network 172.20.0.0
        broadcast 172.20.0.255
    ```

3. **Redémarrer l'interface réseau** pour appliquer les modifications :
    ```bash
    sudo systemctl restart networking
    ```

## Étape 3 : Installation et Configuration du Service DHCP

1. **Installer le serveur DHCP** :
    ```bash
    sudo apt-get update
    sudo apt-get install isc-dhcp-server
    ```

2. **Configurer les interfaces sur lesquelles le serveur DHCP doit écouter** :
    ```bash
    sudo nano /etc/default/isc-dhcp-server
    ```
    Modifier la ligne INTERFACESv4 pour écouter sur `eth0` :
    ```plaintext
    INTERFACESv4="eth0"
    ```

3. **Configurer le fichier de configuration DHCP** :
    ```bash
    sudo nano /etc/dhcp/dhcpd.conf
    ```
    Ajouter les lignes suivantes pour configurer le pool d'adresses et l'attribution statique :
    ```plaintext
    subnet 172.20.0.0 netmask 255.255.255.0 {
        range 172.20.0.100 172.20.0.200;
        option routers 172.20.0.1;
        option subnet-mask 255.255.255.0;
        option broadcast-address 172.20.0.255;
        option domain-name-servers 8.8.8.8, 8.8.4.4;
    }

    host client1 {
        hardware ethernet XX:XX:XX:XX:XX:XX;  # Remplacer par l'adresse MAC du client
        fixed-address 172.20.0.10;
    }
    ```

4. **Redémarrer le service DHCP** pour appliquer les modifications :
    ```bash
    sudo systemctl restart isc-dhcp-server
    ```

## Étape 4 : Test du Bon Fonctionnement du Serveur DHCP

### Test avec un Client Classique

1. **Configurer la carte réseau de la machine cliente** en Réseau Interne.

2. **Démarrer la machine virtuelle cliente**.

3. **Configurer la carte réseau de la machine cliente en DHCP** :
    ```bash
    sudo nano /etc/network/interfaces
    ```
    Ajouter les lignes suivantes pour `eth0` :
    ```plaintext
    auto eth0
    iface eth0 inet dhcp
    ```

4. **Redémarrer l'interface réseau** sur le client :
    ```bash
    sudo systemctl restart networking
    ```

5. **Vérifier que le client obtient une adresse IP dans la plage définie (172.20.0.100 - 172.20.0.200)** :
    ```bash
    ip a
    ```

### Test avec un Client ayant une Adresse MAC Spécifique

1. **Assigner l'adresse MAC spécifique à la machine cliente** (changer l'adresse MAC pour qu'elle corresponde à celle définie dans le fichier de configuration DHCP).

2. **Configurer la carte réseau de la machine cliente en DHCP** (comme ci-dessus).

3. **Redémarrer l'interface réseau** sur le client :
    ```bash
    sudo systemctl restart networking
    ```

4. **Vérifier que le client obtient l'adresse IP statique 172.20.0.10** :
    ```bash
    ip a
    ```

## Conclusion

En suivant ces étapes, vous avez installé et configuré un serveur Debian, configuré un serveur DHCP avec une adresse IP statique basée sur l'adresse MAC d'un client, et vérifié le bon fonctionnement du serveur DHCP avec un client classique et un client avec une adresse IP statique.




