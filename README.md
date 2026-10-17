# Cyber Academy — TP 01 : Diagnostic réseau Linux sous VMware

Pour ce premier TP, j’ai configuré une machine virtuelle Kali Linux avec deux interfaces réseau sous VMware Workstation. L’objectif était de vérifier son adressage IPv4, ses routes, sa résolution DNS et son accès à Internet, puis de comprendre les anomalies rencontrées pendant les tests.

La connectivité fonctionne, mais une latence élevée reste à expliquer. Ce compte rendu présente l’état actuel du diagnostic.

## Mon environnement

- **Système hôte :** Windows 11
- **Hyperviseur :** VMware Workstation
- **Machine virtuelle :** Kali Linux
- **eth0 :** NAT, sur VMnet8
- **eth1 :** Host-only, sur VMnet1

## Configuration réseau obtenue

| Interface | Mode | Adresse IPv4 | Réseau |
| --- | --- | --- | --- |
| eth0 | NAT (VMnet8) | 192.168.152.152/24 | 192.168.152.0/24 |
| eth1 | Host-only (VMnet1) | 192.168.78.129/24 | 192.168.78.0/24 |

La passerelle par défaut est **192.168.152.2**, accessible via **eth0**. Cette interface sert à accéder à Internet. L’interface **eth1** est dédiée au réseau Host-only, pour les échanges avec l’hôte et les machines présentes sur ce réseau virtuel.

## Vérifications effectuées

J’ai commencé par relever les adresses IP, les routes et l’état des connexions :

```bash
ip -br addr
ip route
nmcli device status
nmcli connection show
```

J’ai ensuite testé la passerelle NAT, une adresse IP publique, la résolution de nom et une requête HTTPS :

```bash
ping -c 4 192.168.152.2
ping -c 4 1.1.1.1
getent ahostsv4 example.com
curl -I --max-time 10 https://example.com
```

Ces tests permettent de vérifier séparément la connectivité IP, la résolution DNS et l’accès à un service web en HTTPS.

## Ce que j’ai observé

| Vérification | Résultat |
| --- | --- |
| Accès à la passerelle NAT | 0 % de perte, RTT moyen de 0,243 ms |
| Connectivité IP publique | 0 % de perte lors des tests |
| Résolution DNS de `example.com` | Fonctionnelle |
| Requête HTTPS vers `example.com` | Réponse `HTTP/2 200` |
| État final des interfaces | eth0 et eth1 connectées |

Au départ, **eth1 était configurée en Bridged et n’avait pas d’adresse IPv4 active**. Après son passage en **Host-only**, elle a obtenu l’adresse **192.168.78.129/24**. La configuration correspond alors à l’architecture prévue pour le TP.

## Le point encore à éclaircir : la latence

Malgré des tests de connectivité réussis, les temps de réponse vers `1.1.1.1` sont élevés. Une comparaison sur **10 requêtes par système** a donné les moyennes suivantes :

| Système | Latence moyenne vers 1.1.1.1 |
| --- | --- |
| Kali Linux | 1 589 ms |
| Windows 11 | 1 618 ms |

Ces mesures sur 10 requêtes sont distinctes des commandes de vérification sur 4 requêtes présentées plus haut.

Le traceroute réalisé depuis Windows montre aussi une latence élevée dès le premier saut, vers la passerelle Wi-Fi **192.168.100.1**.

Le fait de retrouver une latence similaire sur l’hôte et dans la VM invite à examiner également la connexion de Windows et le réseau local. Cela ne permet toutefois pas d’identifier la cause exacte, ni de conclure à un problème Wi-Fi à ce stade.

## Prochaines vérifications

- [ ] Effectuer un ping direct vers `192.168.100.1` depuis Windows.
- [ ] Vérifier la qualité de la connexion Wi-Fi.
- [ ] Tester la communication entre Kali et l’interface Host-only de Windows, à l’adresse `192.168.78.1`.

## Ce que ce TP m’a permis de travailler

- Lire un adressage IPv4 et un préfixe CIDR.
- Vérifier une table de routage et identifier la passerelle par défaut.
- Distinguer les modes NAT, Bridged et Host-only de VMware.
- Examiner les interfaces et les connexions réseau sous Linux.
- Croiser des tests ICMP, DNS et HTTPS pour établir un diagnostic.
- Comparer les mesures de l’hôte et de la VM avant de tirer une conclusion.

## Bilan provisoire

L’adressage IPv4 et le routage de Kali sont cohérents avec l’architecture retenue. Les tests de connectivité, de résolution DNS et d’accès HTTPS ont réussi.

Le TP reste néanmoins en cours de validation : je dois encore approfondir le diagnostic de latence avant de considérer les vérifications comme terminées.
