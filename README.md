# TP 01 Cyber : Diagnostic réseau Linux sous VMware



## Objectifs
Configurer et diagnostiquer les interfaces IPv4 d'une VM Kali sous VMware, distinguer NAT et Host-only, lire une table de routage et tester ICMP, DNS et HTTPS.

## Environnement
- Hôte : Windows 11
- Hyperviseur : VMware Workstation
- Invité : Kali Linux
- `eth0` : NAT (VMnet8), `192.168.152.152/24`
- `eth1` : Host-only (VMnet1), `192.168.78.129/24`
- Passerelle par défaut : `192.168.152.2` via `eth0`
- Interface hôte VMnet1 : `192.168.78.1`

## Commandes
```bash
ip -br addr
ip route
nmcli device status
ping -c 4 192.168.152.2
ping -c 4 192.168.78.1
ping -c 10 1.1.1.1
getent ahostsv4 example.com
curl -I --max-time 10 https://example.com
ip route get 1.1.1.1
ip route get 192.168.78.1
```

Sous Windows :
```powershell
Get-NetIPConfiguration
ping -n 20 192.168.100.1
netsh wlan show interfaces
tracert -d 1.1.1.1
```

## Résultats constatés
| Test | Résultat |
|---|---|
| Kali → passerelle NAT `192.168.152.2` | 0 % de perte ; RTT moyen 0,243 ms |
| Kali → hôte Host-only `192.168.78.1` | 4/4 reçus ; RTT moyen 0,783 ms |
| Kali → `1.1.1.1` | 10/10 reçus ; RTT moyen 1 589 ms |
| Windows → `1.1.1.1` | 10/10 reçus ; RTT moyen 1 618 ms |
| Windows → passerelle Wi-Fi `192.168.100.1` | 20/20 reçus ; RTT moyen 1 547 ms ; min 804 ms, max 2 214 ms |
| Résolution DNS | Réussie pour `example.com` |
| HTTPS | Réponse `HTTP/2 200` pour `example.com` |
| `eth1` initialement en Bridged | Pas d'adresse IPv4 active |
| `eth1` après passage en Host-only | Adresse `192.168.78.129/24`, interface connectée |

## Analyse
La table de routage contient deux routes directement connectées (`192.168.152.0/24` sur `eth0`, `192.168.78.0/24` sur `eth1`) et une route par défaut via `192.168.152.2` sur `eth0`. La communication Host-only fonctionne avec une faible latence.

Les latences élevées sont visibles sur Windows et Kali, y compris de Windows vers sa passerelle Wi-Fi. Le problème ne peut donc pas être attribué exclusivement à VMware. Le Wi-Fi, le routeur et la congestion locale sont des pistes prioritaires, sans cause racine établie. Le diagnostic Wi-Fi indique 5 GHz, canal 36, signal 80 %, RSSI -57 dBm et débit de réception négocié de 13,5 Mbit/s lors de la mesure. Ce dernier chiffre n'est pas une mesure du débit Internet.

Le test vers `192.168.1.1` n'était pas pertinent : la véritable passerelle Wi-Fi était `192.168.100.1`.

## Évaluation pédagogique
- Configuration pratique et tests : réussis.
- Premier QCM de rattrapage : 4/5.
- Rattrapage ciblé sur le routage : 3/3.
- **TP 01 validé** après correction de la confusion entre interface Host-only et passerelle par défaut.

## Suite
TP 02 : Ethernet, adresses MAC, ARP, commutation L2 et communication entre deux VM sur VMnet1.

> Note : les adresses privées et les résultats correspondent à ce laboratoire. 
