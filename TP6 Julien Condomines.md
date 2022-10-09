# TP-6 Julien CONDOMINES

## Exercice 1

#### Vous administrez le réseau interne 172.16.0.0/23 d’une entreprise, et devez gérer un parc de 254 machines réparties en 7 sous-réseaux. La répartition des machines est la suivante : - Sous-réseau 1 : 38 machines - Sous-réseau 2 : 33 machines - Sous-réseau 3 : 52 machines - Sous-réseau 4 : 35 machines - Sous-réseau 5 : 34 machines - Sous-réseau 6 : 37 machines - Sous-réseau 7 : 25 machines Donnez, pour chaque sous-réseau, l’adresse de sous-réseau, l’adresse de broadcast (multidiffusion) ainsi que les adresses de la première et dernière machine configurées (précisez si vous utilisez du VLSM ou pas).

Nombre de sous-réseaux : 7

Nombre de bits nécessaires : 3 bits (8 sous-réseaux potentiels)

Nombre maximum de machines : 52

Nombre de bits nécessaires : 6 bits (62 machines potentielles par sous-réseau)

Nombre de bits nécessaire pour ID sous-réseau et ID hôte : 3 + 6 = 9

**ID sous-réseau / Première machine / Dernière machine configurée / Broadcast**

172.16.0.0 / 172.16.0.1 / 172.16.0.62 / 172.16.0.63

172.16.0.64 / 172.16.0.65 / 172.16.0.126 / 172.16.0.127

172.16.0.128 / 172.16.0.129 / 172.16.0.190 / 172.16.0.191

172.16.0.192 / 172.16.0.193 / 172.16.0.254 / 172.16.0.255

172.16.1.0 / 172.16.1.1 / 172.16.1.62 / 172.16.1.63

172.16.1.64 / 172.16.1.65 / 172.16.1.126 / 172.16.1.127

172.16.1.128 / 172.16.1.129 / 172.16.1.159 / 172.16.7.160

## Exercice 2. Préparation de l’environnement

#### 1) VM éteintes, utilisez les outils de configuration de VirtualBox pour mettre en place l’environnement décrit ci-dessus.

Sur vsphere, on modifie les paramètres de la VM, on lui ajoute un nouvel adaptateur réseau: ICS_E07_2014. Sur la deuxième VM, on lui ajoute ce même adaptateur réseau. De ce fait, le serveur a accès à Internet et le client à accès à Internet via le serveur.

#### 2) Démarrez le serveur et vérifiez que les interfaces réseau sont bien présentes. A quoi correspond l’interface appelée lo ?

Avec la commande ip adrr on peut vérifier que les interfaces réseaux sont bien présentes. L'interface lo permet de contacter la machine locale sans passer par une interface qui serait accessible de l'extérieur. En d'autres termes, elle permet à la machine de se connecter à elle même sans passer par le réseau. Elle représente la machine elle-même. On peut parler d'adresse de bouclage.

#### 3) Désinstallez complètement ce paquet (il faudra penser à le faire également sur le client ensuite.)

Pour désinstaller le paquet cloud-init, on va utiliser la commande sudo apt-get purge cloud-init.

#### 4) Les deux machines serveur et client se trouveront sur le domaine tpadmin.local. A l’aide de la commande hostnamectl renommez le serveur (le changement doit persister après redémarrage, donc cherchez les bonnes options dans le manuel!

On va tout d'abord utiliser la commande `sudo hostnamectl set-hostname tpadmin.local` qui va permettre de renommer le serveur. Ensuite, `sudo nano /etc/hosts` pour remplacer l'occurence de l'ancien nom par le nouveau. Enfin, on reboot avec `sudo reboot` pour prendre en compte le changement.

## Exercice 3. Installation du serveur DHCP

#### 1) Sur le serveur, installez le paquet isc-dhcp-server. La commande systemctl status isc-dhcp-server devrait vous indiquer que le serveur n’a pas réussi à démarrer, ce qui est normal puisqu’il n’est pas encore configuré (en particulier, il n’a pas encore d’adresses IP à distribuer)

Après avoir installé le paquet, on peut se rendre compte que le serveur ne parvient pas à démarrer :
```
isc-dhcp-server.service - ISC DHCP IPv4 server
   Loaded: loaded (/lib/systemd/system/isc-dhcp-server.service; enabled; vendor preset: enabled)
   Active: failed (Result: exit-code)
     Docs: man:dhcpd(8)
 Main PID: 1776 (code=exited, status=1/FAILURE)
```

#### 2) Un serveur DHCP a besoin d’une IP statique. Attribuez de manière permanente l’adresse IP 192.168.100.1 à l’interface réseau du réseau interne. Vérifiez que la configuration est correcte.

Le serveur a accès à internet grâce à sa carte réseau NAT (la carte 3).
Le serveur a également accès à un réseau interne (tpadmin.local) avec le client grâce à sa carte de réseau interne (carte 8). Dans le fichier /etc/netplan, on fixe l'adresse ip du serveur de manière permanente dans le réseau interne.
Il faut rajouter dans le fichier netplan une attribution d'ip statique pour la carte 8:

```
network :  
  version : 2  
   renderer : networkd  
   ethernets :  
      enp0s8 :  
          addresses :  
              − 10.10.10.2/24  
``` 

Attention: il ne faut pas enlever la config de la carte 3 dans le netplan du serveur : elle sert à la connexion internet !

#### 3) La configuration du serveur DHCP se fait via le fichier /etc/dhcp/dhcpd.conf. Renommez le fichier existant sous le nom dhcpd.conf.bak puis créez en un nouveau avec les informations suivantes, A quoi correspondent les deux premières lignes? :

```
default-lease-time 120;  
max-lease-time 600;  
authoritative; #DHCP officiel pour notre réseau  
option broadcast-address 192.168.100.255; #informe les clients de l'adresse de broadcast  
option domain-name "tpadmin.local"; #tous les hôtes qui se connectent au 
subnet 192.168.100.0 netmask 255.255.255.0 { #configuration du sous-réseau 192.168.100.0  
  range 192.168.100.100 192.168.100.240; #pool d'adresses IP attribuables  
  option routers 192.168.100.1; #le serveur sert de passerelle par défaut  
  option domain-name-servers 192.168.100.1; #le serveur sert aussi de serveur DNS  
}  
``` 
Sur les 2 premières lignes: on peut lire le bail du serveur dhcp. C'est le temps accordé par le serveur à l’existence d’une ip pour un client. Le client conserve l’ip attribuée pendant la durée du bail. A l’issu de celle-ci, il peut demander une extension de bail.

#### 4) Editez le fichier /etc/default/isc-dhcp-server afin de spécifier l’interface sur laquelle le serveur doit écouter.

Afin de spécifier l'interface sur laquelle le serveur doit écouter, on modifie le fichier isc-dhcp-server avec `sudo nano /etc/default/isc-dhcp-server:`
`INTERFACESv4="enp0s8"; INTERFACESv6="enp0s8"`

#### 5) Validez votre fichier de configuration avec la commande dhcpd -t puis redémarrez le serveur DHCP (avec la commande systemctl restart isc-dhcp-server) et vérifiez qu’il est actif.

On utilise : `sudo dhcpd -t` pour valider le fichier de config du serveur dhcp.
Ensuite, `sudo systemctl restart isc-dhcp-server` pour redémarrer le serveur dhcp. A l'issue de ce redémarrage, le serveur est configuré et donc on peut vérifier qu'il soit bien actif avec `sudo systemctl statut isc-dhcp-server`.

#### 6) Passons au client. Si vous avez suivi le sujet du TP1, le client a été créé en clonant la machine virtuelle du serveur. Par conséquent, son nom d’hôte est toujours serveur. Nous allons remédier à cela. Pour l’instant, vérifiez que la carte réseau du client est débranchée, puis démarrez le client.

Premièrement, il faut suivre les mêmes étapes que pour la partie serveur: `sudo hostname set-hostname client.tpadmin.local` (avec un changement à la main dans le fichier /etc/hosts). On supprime également le paquet cloud-init. Deuxièmement, via `hostname` on obtient bien client.tpadmin.local ; via `ip addr`, la carte réseau du client est bien notée DOWN.

#### 7) La commande tail -f /var/log/syslog aﬀiche de manière continue les dernières lignes du fichier de log du système (dès qu’une nouvelle ligne est écrite à la fin du fichier, elle est aﬀichée à l’écran). Lancez cette commande sur le serveur, puis activez la carte réseau du client et observez les logs sur le serveur. Expliquez à quoi correspondent les messages DHCPDISCOVER, DHCPOFFER, DHCPREQUEST, DHCPACK. Vérifiez que le client reçoit bien une adresse IP de la plage spécifiée précédemment.

D'abord on active la carte réseau du client avec `sudo ip link set enp0s8 up`

La commande `tail -f /var/log/syslog` aﬀiche de manière continue les dernières lignes du fichier de log du système. Le journal du système permet de voir les différentes requêtes qui arrivenet et partent du serveur.

    DHCPDISCOVER : requête du client pour découvrir les serveurs dhcp à dispo. S’il n’y en a pas, il s’auto-attribue une ip.
    DHCPOFFER : le serveur propose une ip du réseau au client.
    DHCPREQUEST : le client accepte la proposition.
    DHCPACK : le serveur confirme la proposition et l’attribution.

Avec ip a, on regarde l’ip attribuée à la carte réseau, c’est 192.168.100.100, c’est-à-dire la première adresse attribuable par le serveur dhcp.

#### 8) Que contient le fichier /var/lib/dhcp/dhcpd.leases sur le serveur, et qu’affiche la commande dhcp-lease-list?

Dans /var/lib/dhcp/dhcpd.leases sur le serveur : pour chaque adresse ip attribuée, on trouve l’horaire d’attribution, l’horaire de fin de bail, l’adresse mac de la machine du client, et le nom du client.

La commande dhcp-lease-list permet d’afficher en tableau les infos du fichier dhcpd.leases. Chaque ligne correspond à un client.

#### 9) Vérifiez que les deux machines peuvent communiquer via leur adresse IP, à l’aide de la commande ping.

On ping avec la commande `ping 192.168.100.1` . On peut voir que les deux communique avec leur adresses ip.

#### 10) Modifiez la configuration du serveur pour que l’interface réseau du client reçoive l’IP statique 192.168.100.20: Vérifiez que la nouvelle configuration a bien été appliquée sur le client (éventuellement, désactivez puis réactivez l’interface réseau pour forcer le renouvellement du bail DHCP, ou utilisez la commande dhclient -v).

On modifie en conséquence le fichier de config `dhcpd.conf`. On force le renouvellement de l'ip du client avec `dhclient -v`. On vérifie l'attribution de l'ip avec `ip a`.

## Exercice 4 : Donner un accès à Internet au client

#### 1) La première chose à faire est d’autoriser l’IP forwarding sur le serveur (désactivé par défaut, étant donné que la plupart des utilisateurs n’en ont pas besoin). Pour cela, il suﬀit de décommenter la ligne net.ipv4.ip_forward=1 dans le fichier /etc/sysctl.conf. Pour que les changements soient pris en compte immédiatement, il faut saisir la commande sudo sysctl -p /etc/sysctl.conf.

On va tout d'abord faire `sudo nano /etc/sysctl.conf` pour aller décommenter la ligne `net.ipv4.ip_forward=1`.
On va ensuite saisir la commande `sudo sysctl -p /etc/sysctl.conf.` afin de pouvoir constater que les changements ont été pris en compte immédiatement et que la nouvelle valeur a bien aussi été prise en compte.

*COURS* : donner au serveur des caractéristiques de routeur : il peut maintenant transmettre des infos entre internet et le client. Il fait interface en routant les paquets.

#### 2) Ensuite, il faut autoriser la traduction d’adresse source (masquerading) en ajoutant la règle iptables ²²suivante : sudo iptables --table nat --append POSTROUTING --out-interface enp0s3 -j MASQUERADE

On va tout d'abord ajouter la règle suivante sudo iptables --table nat --append POSTROUTING --out-interface enp0s3 -j MASQUERADE pour autoriser la traduction d'adresse source (masquerading) puis on va ensuite efféctuer la commande `ping 1.1.1.1` sur le client pour vérifier que tout fonctionne.

*COURS* :  le serveur est l’unique interface pour internet. Le client est masqué par le serveur. L’ip du client est inconnue d’internet qui ne connait que celle du serveur.

## Exercice 5. Installation du serveur DNS

#### 1) Sur le serveur, commencez par installer bind9, puis assurez-vous que le service est bien actif

On va tout d'abord utiliser la commande `sudo apt install bind9`pour installer le DNS bind9 puis ensuite afin de vérifier l' activité de bind9 : `systemctl status bind9.service`.

#### 2) A ce stade, Bind n’est pas configuré et ne fait donc pas grand chose. L’une des manières les simples de le configurer est d’en faire un serveur cache : il ne fait rien à part mettre en cache les réponses de serveurs externes à qui il transmet la requête de résolution de nom. Nous allons donc modifier son fichier de configuration : /etc/bind/named.conf.options. Dans ce fichier, décommentez la partie forwarders, et à la place de 0.0.0.0, renseignez les IP de DNS publics comme 1.1.1.1 et 8.8.8.8 (en terminant à chaque fois par un point virgule). Redémarrez le serveur bind9.

Afin de modifier le fichier de configuration il faut ces commandes :
```
sudo nano /etc/bind/named.conf.options
options {
        directory "/var/cache/bind";

        // If there is a firewall between you and nameservers you want
        // to talk to, you may need to fix the firewall to allow multiple
        // ports to talk.  See http://www.kb.cert.org/vuls/id/800113

        // If your ISP provided one or more IP addresses for stable
        // nameservers, you probably want to use them as forwarders.
        // Uncomment the following block, and insert the addresses replacing
        // the all-0s placeholder.

        forwarders {
                1.1.1.1;
                8.8.8.8;
        };

        //========================================================================
        // If BIND logs error messages about the root key being expired,
        // you will need to update your keys.  See https://www.isc.org/bind-keys
        //========================================================================
        dnssec-validation auto;

        listen-on-v6 { any; };
};

sudo service bind9 restart
``` 
*COURS* Le but de cet étape est de donner à notre DNS interne un lien vers des DNS public externes (google et cloudflare). Ce seront eux qui feront le lien entre nom de domaine et ip.

#### 3) Sur le client, retentez un ping sur www.google.fr. Cette fois ça devrait marcher ! On valide ainsi la configuration du DHCP effectuée précédemment, puisque c’est grâce à elle que le client a trouvé son serveur DNS

On utilise donc la commande `ping www.google.fr`qui fonctionne désormais. 

*COURS* Lorsque l'on ping un site web, cela interroge le serveur DNS interne bind9 que l'on a configuré. Ce serveur transmet ensuite la requête au serveur DNS de google situé à l'ip 8.8.8.8. Le serveur DNS de google fait le lien entre le nom de domaine et l'adresse ip du domaine (même chose ou non à demander).

#### 4) Sur le client, installez le navigateur en mode texte lynx et essayez de surfer sur fr.wikipedia.org (bienvenue dans le passé...)

On va utiliser ces deux commandes  `sudo apt install lynx ` et `lynx fr.wikipedia.org` ce qui va nous amener sur cette page :
```
#                                                                                    Wikipédia, l'encyclopédie libre (p1 of 😎
   #Images du jour de Wikipédia Flux d’articles en vedette de Wikipédia Regards sur l'actualité de la Wikimedia, et
   d'ailleurs Wikimag Wikipédia (fr)

Wikipédia:Accueil principal

   Une page de Wikipédia, l'encyclopédie libre.
   Sauter à la navigation Sauter à la recherche

Wikipédia

   L'encyclopédie libre que chacun peut améliorer
   Version pour appareil mobile
     * Accueil de la communauté
     * Comment contribuer ?
     * Portails thématiques
     * Principes fondateurs
     * Sommaire de l'aide
     * Poser une question

Article labellisé du jour

   Un ancien rouet avec une roue à aubes préservée à proximité du circuit touristique.

   La vallée des Rouets est le nom donné à une partie de la vallée de la Durolle, principalement située sur le
   territoire de la commune de Thiers, dans le département français du Puy-de-Dôme et de la région Auvergne-Rhône-Alpes.
   Elle est connue pour son long passé artisanal car on y a exploité la force motrice de la rivière Durolle dès le Moyen
   Âge. Le début du XX^e siècle marque la fermeture de la plupart des rouets dans la vallée au profit de l'industrie
   coutelière, majoritairement installée dans la vallée des Usines située en aval.

   Après plusieurs années de mobilisations associatives pour la protection du patrimoine bâti de la Vallée des Rouets,
   la mairie de Thiers ouvre au public dès 1998 sous l'impulsion de Maurice Adevah-Pœuf, alors député-maire de Thiers,
   un parcours touristique fusionné avec le musée de la coutellerie présentant l'histoire du lieu, des techniques de
   fabrications de couteaux ainsi que le fonctionnement du dernier rouet en activité de la vallée et de sa roue à aubes.
     * Lire la suite

     * Contenus de qualité
     * Bons contenus
     * Sélection
     * Programme

Actualités

   De pastorie in Nuenen in het voorjaar.
     * 15 avril : élections législatives en Corée du Sud.
     * 4 avril : une attaque au couteau fait deux morts et cinq blessés à Romans-sur-Isère, en France.
     * 30 mars : Le Jardin du presbytère de Nuenen au printemps (tableau), une peinture à l'huile de Vincent van Gogh,
       est volé au musée de Laren (Hollande-Septentrionale).
     * 29 mars : le premier tour des élections législatives a lieu au Mali quatre jours après l'enlèvement de l'opposant
       Soumaïla Cissé par des djihadistes présumés.
     * 27 mars : la Macédoine du Nord devient le trentième État membre de l'Organisation du traité de l'Atlantique nord.
     ______________________________________________________________________________________________________________

   Événements en cours :
   Pandémie de Covid-19 · Invasion de criquets en Afrique de l'Est · Épidémie de dengue · Crise présidentielle au
   Venezuela · Guerre civile yéménite · Guerre civile syrienne
     ______________________________________________________________________________________________________________

   Nécrologie :
   Kerstin Meyer, Markus Raetz (14 avril) · Jacques Blamont, Ryō Kawasaki, Landelino Lavilla, Philippe Lécrivain, Sarah
   Maldoror, Patricia Millardet, Bernard Stalter (13 avril) · Eliahou Bakshi-Doron, Maurice Barrier, Peter Bonetti,
   Daniel Camiade, Jacques De Decker, Sascha Hupmann, Tarvaris Jackson, André Manaranche, Jacques Maury, Charles
   Miossec, Stirling Moss, Doug Sanders, Samuel Wembé (12 avril) · Amanda Baggs, Colby Cave, Hélène Châtelain, John
   Horton Conway, Justus Dahinden, Liu Dehai, Edem Kodjo, Francis Leonard Tombs (11 avril)
     * Avril 2020
     * Wikinews
     * Modifier
``` 

## Exercice 6. Configuration du serveur DNS pour la zone tpadmin.local

#### 1) Modifiez le fichier /etc/bind/named.conf.local et ajoutez les lignes suivantes :

```
sudo nano /etc/bind/named.conf.local
zone "tpadmin.local" IN {
type master;
file "/etc/bind/db.tpadmin.local";
};
```

#### 2) Créez une copie appelée db.tpadmin.local du fichier db.local. Ce fichier est un fichier configuration typique de DNS, constitué d’enregistrements DNS (cf. cours). Dans le nouveau fichier, remplacez toutes les références à localhost par tpadmin local, et l’adresse 127.0.0.1 par l’adresse IP du serveur.

```
;
; BIND data file for local loopback interface
;
$TTL	604800
@	IN	SOA	tpadmin.local. root.tpadmin.local. (
			      2		; Serial
			 604800		; Refresh
			  86400		; Retry
			2419200		; Expire
			 604800 )	; Negative Cache TTL
;
@	IN	NS	tpadmin.local.
serveur.tpadmin.local.	IN	A	192.168.100.1
client.tpadmin.local.	IN	A	192.168.100.20
@	IN	AAAA	::1
```

*COURS* NS signifie name server et IN internet. @ IN NS tpadmin.local. indique que le serveur DNS utilisé par la machine a pour nom tpadmin.local.
Les deux lignes suivantes permettent de faire le lien entre les noms des machines et leur adresse IP.

#### 3) Commencez par rajouter les lignes suivantes à la fin du fichier named.conf.local : zone "100.168.192.in-addr.arpa" { type master; file "/etc/bind/db.192.168.100"; };

```
sudo nano /etc/bind/named.conf.local
zone "100.168.192.in-addr.arpa" {
type master;
file "/etc/bind/db.192.168.100";
};
```

#### 3) Créez ensuite le fichier db.192.168.100 à partir du fichier db.127, et modifiez le de la même manière que le fichier de zone. Sur la dernière ligne, faites correspondre l’adresse IP avec celle du serveur

```
;
; BIND reverse data file for local loopback interface
;
$TTL	604800
@	IN	SOA	tpadmin.local. root.tpadmin.local. (
			      1		; Serial
			 604800		; Refresh
			  86400		; Retry
			2419200		; Expire
			 604800 )	; Negative Cache TTL
;
@	IN	NS	tpadmin.local.
1	IN	PTR	serveur.tpadmin.local.
20	IN	PTR	client.tpadmin.local.

``` 

*COURS* Ici, on n'a pas besoin de préciser les adresses client et serveur en entier. En effet, elles font parties du réseaux 192.168.100, il ne manque que le dernier octet d'adresse. C'est lui qu'on indique.
Les fichiers en reverse permettent de faire le lien ip => nom de domaine.
Les fichiers en direct permettent de faire le lien nom de domaine => ip.

#### 4) Utilisez les utilitaires named-checkconf et named-checkzone pour valider vos fichiers de configuration:

```
$ named-checkconf named.conf.local
$ named-checkzone tpadmin.local /etc/bind/db.tpadmin.local
$ named-checkzone 100.168.192.in-addr.arpa /etc/bind/db.192.168.100
```
#### 5) Redémarrer le serveur Bind9. Vous devriez maintenant être en mesure de ”pinguer” les différentes machines du réseau. redémarrer bind9 : sudo service bind9 restart.

Pour vérifier que la machine est bien connectée au serveur DNS 192.168.100.1: faire `resolvectl status`.
Pour vérifier les liens ip/nom de domaine : faire `host 192.168.100.20` ou `host serveur.tpadmin.local`.
Pour ping les machine avec les noms : faire `ping serveur.tpadmin.local`