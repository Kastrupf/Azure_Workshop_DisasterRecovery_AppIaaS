<h1 align="center">
    Atelier - Scénarios de reprise après sinistre (DR) pour les applications IaaS sur Azure
</h1>

</br>

<h2 align="gauche">
Le Design de l'Architecture 
</h2>

![design](https://user-images.githubusercontent.com/43493818/182216706-1a02e473-08c3-4c63-a429-cd143f3bc2d4.png)

<br>

<h2 align="gauche">
Le Projet
</h2>

<p> Le projet couvre deux scénarios : haute disponibilité et DR (Disaster Recovery). </p>

<h4 align="center">
Haute Disponibilité
</h4>

<p> Dans la région USA EST, nous avons une disponibilité de zone élevée (côté gauche du dessin). Nous avons 6 machines dans la même région, soit 3 dans chaque zone. Il s'agit d'atteindre une disponibilité des applications de 99,99 %. </p>
<p> La haute disponibilité sera assurée par des équilibreurs de charge, avec presque aucun temps d'arrêt des applications. </p>
<p> L'application utilisée dans le projet est muti tier, ce qui signifie que nous avons d'abord un niveau Web et plus tard un niveau de données (SQL Server). </p>
<p> L'équilibreur de charge LBWeb01 est public et son rôle est de répartir la charge entre WebVM1 et WebVM2. Il a une adresse IP publique et fait face à la rue (Internet). 
L'équilibreur de charge LBSQL01 est interne, ce qui signifie que la communication s'effectue uniquement en interne dans l'environnement. L'adresse frontale est une adresse IP interne. </p>
<p>Les deux machines SQL sont standalones. Un basculement « AlwaysON » sera configuré entre eux afin que toutes les machines aient une copie de disque. Si une machine tombe en panne, une autre la recevra automatiquement. </p>
<p>Cloud Witness est un type de témoin de quorum de cluster de basculement qui utilise Microsoft Azure et il sera dans configuré dans un blob dans une compte de stockage. C'est lui qui décidera qui est la machine active. Dans le cadre des meilleures pratiques, physiquement, ce compte sera créé dans une troisième région, qui n'est ni l'EAST US ni le SOUTH CENTRAL US [https://docs.microsoft.com/fr-fr/windows-server/failover-clustering/deploy-cloud-witness](url).</p>
<p>Nous avons toujours deux contrôleurs de domaine avec Active Directory Domain Service en réplication standard dans le même domaine.</p>

<br>

<h4 align="center">
Disaster Recovery
</h4>

<p>Le site de récupération est situé dans une autre zone : SOUTH CENTRAL US (côté droit). 
Il sert à affronter le pire. perte de tout ce que j'avais. Comment remonter la structure dans le second scénario ? </p>
<p>Nous laisserons une structure similaire à celle que nous avons pré-assemblée. Il doit être suffisant pour gérer la charge de travail au cas où il devrait être utilisé, afin de ne pas avoir d'impact négatif sur l'entreprise. </p>
<p>LBWeb02 n'aura pas de pool principal dans un premier temps, car les machines WebVM1 et WebVM2 seront répliquées par Azure Site Recovery, c'est-à-dire qu'elles ne s'allumeront qu'en cas de basculement. Ce sont de simples copies, c'est pourquoi elles portent le même nom. </p>
<p>En interne, LBSQL2 équilibrera la charge vers SQLVM3 et plus que cela, il vous permettra de configurer l'IPVIP de l'écouteur selon les recommandations des meilleures pratiques. </p>
<p>La machine SQLVM3 est répliquée de manière asynchrone. Le côté positif est que comme cette base de données est déjà prête, c'est-à-dire qu'elle est déjà à l'intérieur du cluster avec le listenner configuré, il n'est pas nécessaire de modifier quoi que ce soit dans la base de données lorsque le commutateur est activé.
Le contrôleur de domaine DVVM3, tel qu'il se trouve dans la même forêt, dans le même domaine, est déjà en cours de réplication. Il n'est pas nécessaire de faire quoi que ce soit. </p>
<p>Toutes les machines de dessin s'authentifient auprès d'AD. Les connexions externes doivent passer par le Front Door qui se trouve sur le bord et qui communique avec des équilibreurs de charge externes, en utilisant des critères de priorité. </p>

<br>

<h2 align="gauche">
Quelques etapes du projet 
</h2>

<h4 align="gauche">
Implantation et configuration de l'infra du site principal avec de l'haute disponnibilité dans la région EAST US sur Azure  
</h4>

<h6 align="gauche">
Availability Zone 1 : WebVM1 + SQLVM1 + DCVM1   
</h6>
<img width="842" alt="deployEASTUS" src="https://user-images.githubusercontent.com/43493818/182875558-9647b10b-0def-48fb-9791-b8e886a4aa68.png">

<h6 align="gauche">
Availability Zone 2 : WebVM2 + SQLVM2 + DCVM2   
</h6>
<img width="619" alt="image" src="https://user-images.githubusercontent.com/43493818/182890441-30632c75-c222-4e63-a8f6-a051dda8f8bb.png">

<h4 align="gauche">
Configuration du Cluster Quorum Cloud Witness sur SQLVM1
</h4>
<img width="607" alt="image" src="https://user-images.githubusercontent.com/43493818/182900231-a0299960-3334-42ce-beff-4f5f8406bd9d.png">

<h4 align="gauche">
Configuration des services sur les deux serveurs SQL
</h4>
<img width="566" alt="image" src="https://user-images.githubusercontent.com/43493818/182905571-466a51b1-b8e8-4598-9c65-f515ed5abc63.png">
<br>
<img width="501" alt="image" src="https://user-images.githubusercontent.com/43493818/182907174-7f6ddd8f-5627-4a1a-8c27-e31a28412913.png">

<h6 align="gauche">
Test de basculement réalisé avec succèss 
</h6>
<img width="581" alt="image" src="https://user-images.githubusercontent.com/43493818/182912140-e10a77c4-a950-4f6d-9766-19844c8681d3.png">

<br>

<h4 align="gauche">
Implantation de la structure de Disaster Recovery dans la région SOUTH CENTRAL US sur Azure  
</h4>

<h6 align="gauche">
Importation du Runbook d'automatisation du failover du cluster SQL
</h6>
<img width="746" alt="image" src="https://user-images.githubusercontent.com/43493818/182917627-11bd0ca7-6061-4547-a4d1-64c0e328a3e2.png">

<h6 align="gauche">
Validation de l'état de la réplication des VMs
</h6>
<img width="746" alt="image" src="https://user-images.githubusercontent.com/43493818/182927428-70a11fb7-8951-4a71-bf45-5288eeafb17c.png">

<br>

<h4 align="gauche">
Configuration du Front Door
</h4>
<img width="616" alt="image" src="https://user-images.githubusercontent.com/43493818/182922066-f5aaadc9-7aad-4c4c-8b00-709a38e42a4a.png">

<h6 align="gauche">
Fonctionnement du Front Door : OK 
</h6>
<img width="424" alt="image" src="https://user-images.githubusercontent.com/43493818/182926348-3efc654f-0b0f-4782-bbbd-c48636e858dd.png">

<br>

<h4 align="gauche">
Fin du Lab
</h4>
<p>Suppression des ressources</p>
<img width="249" alt="image" src="https://user-images.githubusercontent.com/43493818/182931533-26c2f5a1-c474-411f-9fbd-30cf671e3da3.png">
<br>

<p>Atelier du samedi 30/07/2022, promu par TFTEC Prime, Brésil. </p>
