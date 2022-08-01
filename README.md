<h1 align="center">
    Atelier - Scénarios de reprise après sinistre (DR) pour les applications IaaS sur Azure
</h1>

</br>

<h2 align="gauche">
Le Design d'Architecture 
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
<p>Les deux machines SQL sont standalones. Un cluster de basculement « AlwaysON » sera configuré entre eux afin que toutes les machines aient une copie de disque. Si une machine tombe en panne, une autre la recevra automatiquement. </p>
<p>Cloud Witness est un compte de stockage. C'est lui qui décidera qui est la machine active. Dans le cadre des meilleures pratiques, physiquement, ce compte sera créé dans une troisième région, qui n'est ni l'EAST US ni le SOUTH CENTRAL US.</p>
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
