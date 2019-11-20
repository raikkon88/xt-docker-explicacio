# Objectius d'aquest document 

- Presentar docker Swarm
- Generar un swarm amb 2 màquines físiques. 
- Posar en marxa un servei. 
- Fer un desplegament d'un contenidor en un servei. 

# Docker Swarm [1]

Docker és un servei que posa l'escalabilitat a disposició de qui l'utilitza. Permet generar contenidors que puguin ser fàcilment replicats o distribuïts. Docker, utilitzant conceptes de xarxa que es van parlar en la sessió anterior ens dóna feta la comunicació entre diferents hosts que volguem utiltizar com a part de la infraestructura.

## Tipus de nodes

Existeixen dos tipus de node en un docker swarm. Els nodes manager i els nodes worker. Els nodes managers son nodes que s'encarreguen de distribuïr i enviar les ordres als nodes worker, els nodes worker compleixen amb les ordres enviades per els nodes manager. Exemples de les funcions d'un node master poden ser el balanceig de càrrega, l'ordre de rèplica d'un contenidor o fins i tot tasques concretes que puguin ser delegades en els altres nodes. 

La mínima unitat atòmica programada en l'scheduling del docker son les tasks. Els nodes manager assignen les tasques als nodes worker en funció del nombre de repliques que s'hagi definit a l'hora d'estipular l'escalabilitat d'un servei. 

# Posada en marxa

En aquest apartat explicarem els passos a seguir per poder muntar una infraestructura swarm amb 2 hosts diferents. 

**Descripció del context :**

Tota la infraestructura es monta dins d'una xarxa NAT el router principal de la qual és calxitu.ddns.net. Que té dues interfícies de xarxa, una interfície a una xarxa externa que reb la ip de l'ISP mitjançant dhcp i una xarxa interna en el rang d'adreces: 192.168.8.0/24.

- Domini: calxitu.ddns.net
- Màquina 1 : Intel xeon 8 cores, 8 Gb Ramm, ip interna 192.168.8.250. 
- Màquina 2 : AMD FX6 6 Cores, 6Gb Ramm, ip interna 192.168.8.249.

S'utilitzen ip's fixes per a la simplicitat de la gestió de la connectivitat i també per què no té sentit que tinguin ip's assignades dinàmicament. Son màquines que actúen de servidors i requereixen de configuracions específiques per ells. 

## Instal·lació swarm

Un cop docker ja ha sigut instalat en una màquina, la generació d'un node manager i la creació del swarm passa per la instrucció swarm init : 

```
marc@calxitu:~$ docker swarm init
Swarm initialized: current node (uyxm5kxbdjrcx4sdi5iakgel6) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-39ipyw2nf3fprsh65vauwwzg5fjvm34l9cwus5dtnhjah3r81r-amoanlnzp1ipob5yb3sfyobr 192.168.8.250:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
```

A partir d'aquest moment la màquina 1 és un node manager de un swarm de dockers. 

Recordem com es va explicar en l'apartat de networking, docker utilitzava el seu protocol de comunicació per parlar amb els altres nodes. En aquest cas docker ens inicialitza el swarm i ens proporciona un token per poder afegir-hi nodes.

Per afegir una altre màquina com a node del swarm, en aquest cas com a worker utilitxem la instrucció que es presenta anteriorment. 

```
marc@calxitu2:~$ docker swarm join --token SWMTKN-1-39ipyw2nf3fprsh65vauwwzg5fjvm34l9cwus5dtnhjah3r81r-amoanlnzp1ipob5yb3sfymobr 192.168.8.250:2377
This node joined a swarm as a worker.
```

Verifiquem l'estat dels nodes del swarm. 

```
$ docker node ls
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
uyxm5kxbdjrcx4sdi5iakgel6 *   calxitu             Ready               Active              Leader              19.03.5
nl6doh4795o86wu84bemesn58     calxitu2            Ready               Active                                  19.03.5
```

Veiem que ambdós estan connectats, amb l'status ready i l'availability active. El manager Status ens proporciona la informació per poder discernir entre el node manager i el node worker, en aquest cas el node manager és l'anomenat leader.

Podem veure la representació del swarm des del portainer. 

![Portainer swarm](./img/ca.png)


# Referències 

[1] - Docker Swarm : [https://docs.docker.com/engine/swarm/key-concepts/](https://docs.docker.com/engine/swarm/key-concepts/)
[2] - Docker swarm init : [https://docs.docker.com/engine/reference/commandline/swarm_init/](https://docs.docker.com/engine/reference/commandline/swarm_init/)