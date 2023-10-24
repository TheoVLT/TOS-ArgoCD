# TOS ArgoCD

Bonjour à tous,

Aujourd'hui, nous plongeons dans l'univers fascinant des déploiements continus dans des environnements Kubernetes en explorant un outil open-source essentiel, ArgoCD. L'automatisation des déploiements est devenue une nécessité pour les équipes de développement cherchant à garantir une livraison rapide, cohérente et sans erreurs de leurs applications. C'est là qu'ArgoCD entre en scène.

Dans cette présentation, nous allons découvrir ce que fait ArgoCD, pourquoi il est devenu un acteur majeur dans le domaine de la gestion des déploiements dans Kubernetes, et comment il peut simplifier la vie des équipes de développement et des opérations. Nous allons également voir comment faire le lien avec nos repos Azure DevOps.

Alors, préparez-vous à plonger dans le monde passionnant d'ArgoCD et à comprendre comment cette puissante solution peut révolutionner votre approche des déploiements continus. Sans plus attendre, entrons dans le vif du sujet et découvrons comment ArgoCD peut être votre allié dans la quête de déploiements plus efficaces et plus fiables.
![image](https://github.com/TheoVLT/TOS-ArgoCD/assets/148872577/d439fb3b-c256-481e-a562-1899c1723255)


# Pourquoi ArgoCD ?

Dans un monde où les développements logiciels évoluent rapidement, la nécessité d'automatiser les déploiements est cruciale. Les déploiements manuels peuvent entraîner des erreurs humaines coûteuses, des retards dans la mise en production, et une incohérence dans les environnements de test et de production.

C'est là qu'ArgoCD intervient. ArgoCD offre une solution élégante en automatisant le processus de déploiement, en garantissant la cohérence entre les environnements et en offrant une reproductibilité totale. Plus besoin de dépendre des processus manuels fastidieux qui ralentissent votre cycle de développement.

ArgoCD, avec ses fonctionnalités avancées, peut être la réponse à ces défis en transformant vos déploiements continus en un processus fluide, efficace et exempt d'erreurs. Dans les prochaines diapositives, nous allons plonger plus en profondeur dans les fonctionnalités et les avantages d'ArgoCD.

## Pré-requis

- Un cluster Kubernetes prêt à être utilisé. Cela peut être un cluster local (comme Minikube ou Kind) ou un cluster Kubernetes dans un environnement de production.
- Requiert Kubernetes 1.16.0 ou ultérieur
- Désactiver le swap sur le cluster
- Open-iscsi doit être installé :  [Suivre cette documentation](https://longhorn.io/docs/1.5.1/deploy/install/#installing-open-iscsi)
- Un client NSFv4 doit être installé :  [Documentation](https://longhorn.io/docs/1.5.1/deploy/install/#installing-nfsv4-client)
- MetalLb si vous avez un cluster bare metal :  [Documentation](https://metallb.universe.tf/installation/)
- Espace de stockage : Assurez-vous d'avoir suffisamment d'espace de stockage pour les ressources et les données nécessaires à ArgoCD.

## Installation en Helm
1. On va commencer par créer le namespace qu'on nous utiliserons pour ArgoCD :
```shell
kubectl  create  namespace  argocd
```
2. Ensuite on importe le repo ArgoCD :
```shell
helm  repo  add  argo  https://argoproj.github.io/argo-helm
```
3. On actualise les repos pour avoir les dernières charts :
```shell
helm repo update
```

4. Dans un dossier dédié, on va importer le fichier de configuration que l'on modifiera :
```shell
helm  repo  add  argo  https://argoproj.github.io/argo-helm
mkdir  argocd
cd  argocd
helm  show  values  argo/argo-cd > values.yml```
```
5. On change les valeurs dans le fichier :
```shell
nano  values.yml
on  change  ClusterIP  par  LoadBalancer (dans Server service  type)
on  cherche External  Redis  password  et  on  change  le  username  et  le  password
Si vous utilisez Ingress : on  enable  ingress => à la place de false mettre true 
```
6. Installation avec notre fichier :
```shell
helm  install  argocd  argo/argo-cd  -n  argocd  -f  values.yml
```
7. On vérifie que les pods sont en running (ça peut mettre un certain temps) :
```shell
kubectl  get  pods  -A
kubectl -n argocd get pod
```

```shell
NAME  						READY  STATUS  		RESTARTS  		AGE
argocd-application-controller-0  1/1  Running  0  103m
argocd-applicationset-controller-9d98764f8-lf5wz  1/1  Running  0  103m
argocd-dex-server-555c664ccd-8hqb2  1/1  Running  0  103m
argocd-notifications-controller-b9b9454d4-5hznz  1/1  Running  0  103m
argocd-redis-86588c9cc4-crpcw  1/1  Running  0  103m
argocd-repo-server-7547957d95-5twtd  1/1  Running  0  103m
argocd-server-75cc888f77-8tjpk  1/1  Running  0  103m
```

## Installation de l'outil CLI ArgoCD
ArgoCD est livré avec un outil CLI. Cet outil nous permettra d’interagir avec le serveur ArgoCD.
```shell
curl  -sSL  -o  argocd-linux-amd64  https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
install  -m  555  argocd-linux-amd64  /usr/local/bin/argocd
rm  argocd-linux-amd64
chmod  +x  /usr/local/bin/argocd
```
Pour récupérer le mot de passe admin :
```shell
kubectl  -n  argocd  get  secret  argocd-initial-admin-secret  -o  jsonpath="{.data.password}" | base64  -d
```

## Ajouter le repo Azure DevOps
![image](https://github.com/TheoVLT/TOS-ArgoCD/assets/148872577/fe75a98d-81ce-44ce-9e5d-7643f75ae303)
- Pour ajouter le repo on récupère l'ip du service :
```shell
kubectl -n longhorn-system get svc
```
- On peut alors ajouter notre repo :
```shell
argocd  login  IPDUSERVICE:80
argocd  repo  add  LIEN DU REPO  --username  username  --password  password
```
[Récupération des infos sur DevOps](https://drive.google.com/file/d/1cMbW5Csqaw1JUx7ZwvMPHbCu5J7Ko-gz/view?usp=drive_link)


