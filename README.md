# TP Packet Tracer : VLAN et routage inter-VLAN (Router-on-a-Stick)

## Objectifs

- **Partie 1** : Ajouter des VLAN à un commutateur
- **Partie 2** : Configurer des sous-interfaces sur le routeur
- **Partie 3** : Tester la connectivité avec le routage inter-VLAN

## Scénario

Dans cette activité, on configure des VLAN et le routage inter-VLAN (méthode *router-on-a-stick*). On active ensuite le trunking sur le port du commutateur relié au routeur, puis on vérifie la connectivité entre les VLAN.

## Topologie et adressage

![Topologie du réseau](images/topologie.png)

![Table d'adressage](images/table-adressage.png)

## Partie 1 : Ajout des VLAN au commutateur S1

1. Création des VLAN 10 et 30 sur S1.
2. Configuration de F0/11 et F0/6 en ports d'accès :
   - F0/11 (PC1) → VLAN 10
   - F0/6 (PC3) → VLAN 30
3. Vérification de la configuration des VLAN :

![Résultat de la vérification des VLAN sur S1](images/partie1-show-vlan-brief.png)

### Question : le ping PC1 → PC3 a-t-il abouti ?

![Ping PC1 vers PC3 (échec)](images/partie1-ping-echec.png)

**Non.** PC1 (VLAN 10) et PC3 (VLAN 30) appartiennent à deux domaines de diffusion et à deux sous-réseaux différents. Le commutateur isole les VLAN les uns des autres et aucun équipement de couche 3 n'est encore opérationnel pour router le trafic entre eux.

## Partie 2 : Configuration des sous-interfaces sur R1

- Création des sous-interfaces **G0/0.10** et **G0/0.30**.
- Encapsulation **802.1Q** avec le VLAN correspondant sur chacune.
- Attribution des adresses IP selon la table d'adressage (172.17.10.1/24 et 172.17.30.1/24).

![Configuration des sous-interfaces sur R1](images/partie2-sous-interfaces.png)

- Activation de l'interface physique **G0/0** : les sous-interfaces étant virtuelles, elles dépendent de l'état de l'interface physique et passent à l'état *up* une fois celle-ci activée.

![Vérification de l'état des sous-interfaces](images/partie2-verification.png)

## Partie 3 : Test de la connectivité avec le routage inter-VLAN

### Étape 1 : ping PC1 → PC3

![Ping PC1 vers PC3 (échec avant trunk)](images/partie3-ping-echec.png)

**Le ping échoue encore.** Le port G0/1 de S1 n'est pas encore configuré en trunk : il est resté en mode accès dans le VLAN 1. Les trames étiquetées des VLAN 10 et 30 ne peuvent donc pas atteindre les sous-interfaces de R1.

### Étape 2 : activation du trunking

**À quel VLAN G0/1 est-elle attribuée ?**

![Résultat de show vlan avant le trunk](images/partie3-show-vlan-avant.png)

Au **VLAN 1** (VLAN par défaut).

**Comment savoir, avec `show vlan`, que l'interface est un port trunk ?**

![Résultat de show vlan après le trunk](images/partie3-show-vlan-apres.png)

Une fois le trunk activé, G0/1 **n'apparaît plus** dans la liste des ports d'aucun VLAN (notamment plus dans le VLAN 1). Les ports trunk ne sont pas associés à un VLAN en particulier et sont donc absents de cette sortie.

**Vérification** : G0/1 fonctionne bien en mode trunk.

![Vérification du trunk](images/partie3-show-interface-trunk.png)

### Étape 3 : test de connectivité

Avec une configuration correcte, PC1 et PC3 peuvent pinguer leur passerelle par défaut ainsi que l'autre PC.

![Ping PC1 vers sa passerelle et vers PC3 (réussi)](images/partie3-ping-pc1.png)

![Ping PC3 vers sa passerelle et vers PC1 (réussi)](images/partie3-ping-pc3.png)

**Passerelles par défaut utilisées :**

- PC1 (VLAN 10) : **172.17.10.1**
- PC3 (VLAN 30) : **172.17.30.1**

## Conclusion

Le routage inter-VLAN en *router-on-a-stick* repose sur trois éléments :

1. Des VLAN créés et correctement attribués aux ports d'accès du commutateur.
2. Des sous-interfaces 802.1Q sur le routeur, une par VLAN, servant de passerelles.
3. Un port trunk entre le commutateur et le routeur pour transporter le trafic de tous les VLAN.

Sans trunk, le routeur ne reçoit pas les trames étiquetées et la communication entre VLAN reste impossible.
