# Rendu — Atelier Kubernetes

> Complétez ce fichier au fil de l'atelier, puis faites un commit et un push (voir la section « Rendu » du README).

**Nom / Prénom :** GUN Louis
**Groupe :** RS2

---

## Exercice 1 — Cluster

1. Combien de nœuds affiche `kubectl get nodes` ? Quel est le rôle de chacun ?
   > Selon la configuration de création du cluster, `kubectl get nodes` affiche **3 nœuds** (1 `server` et 2 `agent`, ou 2 nœuds s'il n'y a qu'un seul agent) :
   > - **Le nœud `server` (`control-plane`)** : pilote le cluster (API Server, scheduler, gestion de l'état) et peut aussi héberger des pods.
   > - **Les nœuds `agent` (`worker`)** : exécutent uniquement les conteneurs applicatifs (pods)
2. Que montre `docker ps` ? Qu'est-ce qu'un « nœud » dans k3d, concrètement ?
   >`docker ps` liste les **conteneurs Docker en cours d'exécution** sur la machine hôte (un conteneur par nœud k3d + un conteneur load-balancer `serverlb`). Concrètement, dans **k3d** (*k3s in Docker*), un nœud Kubernetes est un simple **conteneur Docker** exécutant une instance légère de Kubernetes (k3s).

## Exercice 2 — Premier déploiement

3. Quels objets Kubernetes ont été créés par `kubectl create deployment` ? (indice : `kubectl get all`)
   >La commande a créé trois objets : un **`Deployment`**, un **`ReplicaSet`** et un **`Pod`**.
4. Vous avez supprimé le pod `web-...` : que s'est-il passé, et pourquoi ?
   >Le pod a été **immédiatement recréé** avec un nouveau nom. Le **`ReplicaSet`** surveille en permanence l'état du cluster et garantit que le nombre de réplicas défini (1) est toujours actif.

## Exercice 3 — Modifier un conteneur en direct

5. Après suppression du pod, votre message « Bienvenue... » a-t-il survécu ? Pourquoi ?
   >**Non**, le message a disparu. Les modifications effectuées directement dans un conteneur sont éphémères : le nouveau pod a été recréé à partir de l'image Docker d'origine, qui est immuable.
6. Pourquoi le `port-forward` a-t-il cessé de fonctionner après la suppression du pod ?
   >Parce que `kubectl port-forward` établit un tunnel direct vers un pod précis (via son nom/IP). Lorsque ce pod est supprimé, le tunnel est coupé et ne bascule pas automatiquement vers le nouveau pod.

## Exercice 4 — Déclaratif

7. Quelle différence entre `kubectl create` et `kubectl apply` ?
   >- `kubectl create` (**impératif**) : crée la ressource et échoue si elle existe déjà.
   >  `kubectl apply` (**déclaratif**) : crée la ressource si elle n'existe pas, ou applique uniquement les modifications nécessaires si elle existe déjà.
8. À quoi sert le `selector` d'un Service ? Que se passe-t-il s'il ne correspond à aucun pod ?
   >Le `selector` permet au Service d'identifier les pods cibles via leurs **labels** (ex. `app: site`) afin de leur acheminer le trafic réseau. S'il ne correspond à aucun pod, le Service n'a aucun `Endpoint` et le trafic échoue (erreur de connexion / timeout).

## Exercice 5 — Scaling & mises à jour

9. Sur quels nœuds se trouvent vos 3 réplicas ? Collez la sortie de `kubectl get pods -l app=site -o wide`.
   ```
 NODE   READINESS GATES
site-fc465f957-284q2   1/1     Running   0          2m41s   10.42.0.8   k3d-lab-server-0   <none>           <none>
site-fc465f957-j6xr9   1/1     Running   0          2m28s   10.42.3.7   k3d-lab-agent-1    <none>           <none>
site-fc465f957-kr9gg   1/1     Running   0          2m36s   10.42.1.8   k3d-lab-agent-0    <none>           <none>
   ```
10. Avec l'image `nginx:9.99`, quel statut avait le pod ? Quelle commande vous a donné la cause ?
    >Le pod avait le statut **`ErrImagePull`** puis **`ImagePullBackOff`** (tag d'image inexistant). La commande **`kubectl describe pod <nom-du-pod>`** (dans la section `Events`) a donné la cause exacte.
11. Pendant la panne, le site restait-il accessible ? Pourquoi ?
    >**Oui.** Par défaut, un Deployment utilise la stratégie **`RollingUpdate`** : Kubernetes conserve les anciens pods actifs tant que les nouveaux ne sont pas prêts (`Ready`). La mise à jour étant bloquée sur le nouveau pod en erreur, les anciens pods ont continué à servir le site sans interruption.

## Exercice 6 — Portainer & Mario

12. URL publique de votre Mario :
    >https://laughing-system-q77ppx55gqrx34prj-8585.app.github.dev/
13. Retrouvez le Deployment `mario` en ligne de commande. Portainer et `kubectl` parlent-ils au même composant ? Lequel ?
    >**Oui**, Portainer et `kubectl` communiquent exactement avec le même composant central : le **`kube-apiserver`** (l'API Server du Control Plane Kubernetes).

## Bonus (facultatif)

14. Bonus réalisés et remarques :
    >

---

**Score obtenu avec `./scripts/check.sh` :** ... / ...
