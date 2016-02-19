
# FoSyMA -- Cours 7


## 1. Protocoles d'interactions

On rappelle que les messages sont des actes de communication composés esentiellement:

- d'un **émetteur**
-  d'une liste de **receveurs** 
- d'un **performatif** (le type de message, par ex. *request*)
- d'un **contenu** du message 

Les protocoles d'interactions régulent les échanges de messages. 

> **Rappel:** en JADE, vous disposez de l'agent *sniffer*, qui permet de visualiser simplement les séquences de messages

```sequence
Alice->Bob: Quel est le prix de cette pierre?
Note right of Bob: Bob calcule le prix
Bob-->Alice: Je propose 7 euros
```
## 2. Propriétés des protocoles

Comment mesurer la qualité d'un protocole? 
De fait, il faut souvent parler du couple stratégie-protocole. 
Les propriétés souhaitées sont: 

 
 - la **conformité**:  la stratégie de l'agent est-elle conforme aux règles du protocole. On peut vérifier cette propriété ``à la volée" (on-the-fly), c'est-à-dire pendant l'éxécution (run-time), ou a priori (sur la base des spécifications du protocole et des stratégies des agents). 
 
 - la **terminaison**: la stratégie et le protocole permettent-ils d'atteindre un état final? Si l'on restreint à l'analyse du protocole, on pourra se demander si la terminaison est possible pour une stratégie donnée. 
 
 - le **succès** ou l'**optimalité**: l'état atteint est-il satisfaisant? Permet-il d'atteindre une solution optimale parmi toutes les solutions possibles? (bien sûr, cette notion d'optimailité peut être définit de multiple manière selon les domaines)
 
 - l'**efficacité**: la quantité d'information échangée au cours du protocole est-elle acceptable, minimale? 


### Conformité 

Dans les contextes non-coopératifs, la question est centrale car les agents peuvent avoir intérêt à ne *pas* respecter le protocole. Dans les contextes coopératifs, il s'agit plutôt de vérifier que le comportement des agents suit les spécifications.  

**Quelle sémantique pour les actes de communication?**
La question de vérification de conformité consiste donc à vérifier que les actes de communication sont conformes à leur sémantique. 

Par exemple, les actes de FIPA-ACL sont associés à une sémantique définie en terme d'états mentaux. Ainsi:

$$ inform(x,y,p) : K_xp \wedge \neg K_x (K_y p \vee K_y \neg p)$$

Cela pose un gros problème, car il est difficile, voire impossible,  de supposer que l'on peut accéder aux états mentaux (internes) des agents. 

La vérification de conformité doit donc opérer sur des notions **observables**. De l'intérêt des protocoles. On peut vérifier la conformité d'une séquence de messages avec la spécification proposée. 

**Comment vérifier?**
Soit on suppose que c'est un système capable d'avoir un accès instantané à tous les messages. 
Si ce n'est pas le cas, la vérification on-the-fly peut aussi poser de nombreux problèmes: par exemple l'accès à certains messages qui peuvent être retardés. Comment être sûr que ce message a bien été envoyé avant tel autre? 

**Et si vous deviez faire un agent vérificateur en JADE?**

> Vos réponses (cours 13/03): 
> L'agent peut coder un timestamp qui marque un message pour une séquence de message donnée. 
> Un agent $v$ joue le rôle du vérificateur. Soit on double les messages et on les envoie à $v$, soit $v$ joue un rôle de médiateur. L'agent initiateur de l'échange envoie avec le premier message le protocole utilisé. 


(Une idée: on créé un agent avec le rôle de vérificateur. La premier message doit être envoyé à l'agent récepteur et au vérificateur. Puis les messages transitent par lui). Bien, pas bien?  

```sequence
Alice->Bob:bla
Alice->Checker: bla
```



### Complexité de communication


Que mesurer? Le nombre de messages? Le nombre de tours? La taille des messages? 
Voici un exemple : 

> Les deux agents Alice et Bob souhaitent se coordonner pour donner la priorité à celui ou celle qui a le plus de place disponible dans son sac. Quel protocole employer? 

Limitons nous pour la simplicité du propos à $k \in \{0,1,2,3\}$ pour la place disponible dans le sac de chaque agent. 

**Protocole 1: Révélation asymétrique**
```sequence
Alice->Bob: Il me reste k kilos 
Note right of Bob: Bob calcule 
Bob-->Alice: J'y vais 
Bob-->Alice: Vas-y
```

Combien de messages? 	
2
Combien de bits échangés? 
$log(4)=2$+1 = 3 (coder la place disponible pour Alice, répondre j'y vais ou tu y vas pour Bob). 
 
**Protocole 2: Enchère**

```sequence
Alice->Bob: J'ai au moins 0 kilo
Note right of Bob: Bob calcule 
Bob-->Alice: J'ai au moins 1 kilo de plus
Bob-->Alice: Vas-y
Note left of Alice: Alice calcule 
Alice-->Bob: J'ai au moins 1 kilo de plus
Alice-->Bob: Vas-y
Note right of Bob: Bob calcule 
Bob-->Alice: J'ai au moins 1 kilo de plus
Bob-->Alice: Vas-y
```


Combien de messages? 	
2, 3, ou 4
Combien de bits échangés? 
2, 3, ou 4 (chaque message peut être codé sur 1 bit)

Protocole 3: 

```sequence
Note left of Alice: Alice calcule 
Alice->Bob: Il me reste beacoup de place (2 ou 3) 
Alice-->Bob: Il me reste peu de place (0 ou 1)
Note right of Bob: Bob calcule 
Bob-->Alice: J'y vais
Bob-->Alice: Vas-y
```

```java
// Alice calcule
if (alice.getBackPackFreeSpace()<=1) 
	{alice.send(``Peu'');
	}
else	
	{alice.send(``Beaucoup'');
	}
	

// Bob calcule
if (bob.receive()==``Peu'') {
	if (bob.getBackPackFreeSpace()==0)
	{bob.send(``you Go'');
	}
	else 
	{bob.send(``I Go'');
	}}
else { //alice has high space
	if (bob.getBackPackFreeSpace()==3)
	{bob.send(``I Go'');
	}
	else
	{bob.send(``you Go'');
	}}

	  
```

Combien de messages? 	
2
Combien de bits échangés? 
2 (chaque message peut être codé sur 1 bit)


## 3. Analyse d'un protocole de mariage
Nous allons analyser maintenant un protocole permettant de constituer des équipes de deux agents. 

> Les agents chasseurs sont de deux types: les serruriers (ceux qui peuvent ouvrir les coffres), et les glaneurs (ceux qui remplissent leur sac de trésors). Les chasseurs doivent donc opérer en équipe de deux. Chaque serrurier possède des préférences sur les glaneurs, et vice-versa.

Dans ce scenario, qui est proche de la formation de coalition (cf. cours 8), il s'agit donc de former des équipes de deux. 
On suppose que chaque chasseur classe les chasseurs de l'autre type. 
Par exemple: 


|Serruriers------------ | ------------Glaneurs |
|------:| :------|
|$g_4,g_1,g_2g_3|s_1$     | $g_1| s_4,s_1,s_3,s_2$ |
| $g_2,g_3,g_1g_4|s_2$     | $g_2| s_1,s_3,s_2,s_4$ |
| $g_2,g_4,g_3g_1|s_3$     | $g_3| s_1,s_2,s_3,s_4$ |
| $g_3,g_1,g_4g_2|s_4$     | $g_4| s_4,s_1,s_3,s_2$ |

On dit qu'une paire d'agents $x,y$ (serrurier / glaneur) est **bloquante** lorsque $x$ et $y$ préfèreraient chacun être avec l'autre qu'avec leur co-équipier actuel. 

```python
marquer comme libres tous les serruriers et les glaneurs
while un serrurier s est libre: 
begin
  g:= premier glaneur sur la liste de s à qui s n a pas encore proposé 
  if g est libre: 
  	g et s forment équipe 
   else:
   	if g préfère s à son équipier actuel s2:
        g et s forment équipe
        libère s2
end;
```

Dans cette proposition, on note que *seuls les serruriers proposent*. Il s'agit en fait de l'algorithme de Gale-Shapley pour les ``mariages stables''. 

Pour la seule séquence de proposition, on peut modéliser par le diagramme suivant: 
```sequence
Serrurier1->Glaneur: proposition
Note right of Glaneur: glaneur évalue 
Glaneur-->Serrurier1: ok
Glaneur-->Serrurier1: non merci
Glaneur-->SerrurierActuel: ciao
```

Mais on voit également que le protocole nécessite une coordination plus globale. Par exemple, les serruriers peuvent se répartir la parole dans un ordre précis. 
Mais comment être sûrs que tous les serruriers ont trouvé un équipier? 
> Vos réponses (cours 13/03): 
> Un agent peut jouer le rôle de celui qui tient à jour la liste des serruriers en équipe. Quand un serrurier est libéré, il en informe cet agent. Quand tous les serruriers sont en équipe, cet agent diffuse l'information à tout le monde. 


### Propriétés de ce protocole
Examinons à présent les propriétés de ce protocole:

- **Terminaison** garantie---pourquoi? 
- Qualité de l'état final: 
 - **stabilité**: il n'existe pas de paire d'agents bloquante
 - **optimalité**: l'ensemble d'équipes obtenues ne peut pas être amélioré *du point de vue des serruriers*. Note: si les glaneurs effectuaient les propositions, l'état final serait optimal pour ceux-ci, le problème est complètement symétrique. 
- **Communication**: hors coordination globale, le nombre de messages est borné par $n^2$ 



### Protocole sans coordination globale

Si les serruriers (resp. les glaneurs) ne se coordonnent pas, qu'obtient-on? 
Un serrurier contacte un glaneur qu'il préfère à son co-équipier actuel, le récepteur évalue la proposition et accepte ou refuse de former une nouvelle équipe. 

**Problèmes**:
 
 
- un agent peut être soumis à différentes propositions: comment choisir? 
- un agent serrurier peut choisir différent glaneurs, comment choisir? 
-  comment être sûr globalement qu'un état stable a été atteint? 
- est-on seulement sûr que ce protocole va terminer?  

Supposons ici qu'un glaneur n'accepte pas plusieurs requêtes simultanées (les communications sont bilatérales et exclusives). 


|Serruriers------------ | ------------Glaneurs |
|------:| :------|
|$g_2,g_3,g_1|s_1$     | $g_1| s_1,s_3,s_2$ |
|$g_1,g_2,g_3|s_2$     | $g_2| s_2,s_1,s_3$ |
|$g_3,g_1,g_2|s_3$     | $g_3| s_1,s_2,s_3$ |
 
On voit que ce type de dynamique peut cycler! C'est-à-dire que les agents vont toujours trouver des interactions qui seront rationnelles à leur yeux. 
Les travaux dans ce domaine cherchent alors à déterminer les garanties de convergence vers un état stable, en fonction de la "dynamique" étudiée. 

## 4. Machines à engagements

Plusieurs approches ont été proposées pour réguler l'interaction non pas sur la base de séquences légales de messages (machines à états finis, réseaux de Petri), mais en s'appuyant sur des notions d'engagement ou de conventions sociales. Intuitivement, énoncer un message va créer en retour des engagements pour les interlocuteurs de répondre d'une certaine manière, ou de réaliser certaines actions.  

Ce qui suit est tiré de: 
*[P. Yolum and M.P. Singh. Commitment Machines. 2004]*

Un **engagement** est de la forme: 
$$C(x,y,p)$$
où $x$ est l'agent engagé, $y$ le récepteur de l'engagement, et $p$ la condition . 
Autrement dit, ``$x$ est engagé envers $y$ de faire en sorte que $p$''

Un **engagement conditionnel** est de la forme:
$$CC(x,y,p,q)$$
qui signifie: si $p$ est satisfait, $x$ sera engagé envers $y$ à faire en sorte que $q$. 

Un engagement peut être: 
1. actif
2. satisfait
3. violé

Les opérations disponibles pour manipuler les engagements sont les suivantes: 

1. $$Create(x,C(x,y,p))$$ --- création de l'engagement par $x$, de $x$ envers $y$ de faire en sorte que $p$
2. $$Discharge(x,C(x,y,p))$$ --- cette action peut seulement être réalisée par le créateur de l'engagement. Après cette opération, la condition $p$ est vraie. 
3. $$Cancel(x,C(x,y,p))$$ --- annulation de l'engagement par $x$ (typiquement remplacé par un autre engagement) 
4. $$Release(y,C(x,y,p))$$ --- libération par $y$ de l'engagement pour $x$
5. $$Assign(y,z,C(x,y,p))$$ ---élimination par $y$ de l'engagement et réassignement à un autre agent $z$
6. $$Delegate(x,z,C(x,y,p))$$ ---transfert, par $x$ lui-même, de l'engagement à un autre agent $z$

Les règles d'évolution des engagements sont les suivantes: 

1. un engagement $C(x,y,p)$ disparait lorsque la condition $p$ est réalisée
2. un engagement conditionnel $CC(x,y,p,q)$ disparait lorsque la condition $p$ est satisfaite, mais l'engagement $C(x,y,p)$ est crée. 
3. un engagement conditionnel $CC(x,y,p,q)$ diaparait lorsque la condition $q$ est satisfaite. 

###Exemple du Contract-Net 

On note par $M$ le rôle du manager du Contract-Net, où $reply$ est un raccourci pour $accepted \vee rejected$. 

$sendCFP(M,P): Create(M,CC(M,P,proposal,reply))$
$sendProposal(P,M): Create(P,CC(P,M,accepted,result))$
$sendReply(M): (accepted \vee rejected)$
$sendResult(P): Discharge(P, C(P,M,result))$
$transferProposal(P,P'): delegate(P,P',C(P,M,result))$
$cancelProposal(P): Cancel(P, C(P,M,result))$

Et voyons à présent une évolution possible des états correspondant à une séquence d'actes. 
Après que le manager a envoyé le CFP et qu'un proposant a envoyé une proposition, les deux engagements suivants sont créés: 
$$C(M,P,accepted \vee rejected) \wedge CC(P,M,accepted,result))$$
Supposons à présent que le manager accepte la proposition. On a alors: 
$$accepted \wedge C(P,M,result)$$
Par un transfert de la proposition, le proposant $P$ délègue l'engagement à un autre agent $P'$:
$$accepted \wedge C(P',M,result)$$
En envoyant le résultat, $P'$ réalise l'engagement, et on arrive finalement dans l'état: 
$$accepted \wedge result$$

Intuitivement, l'état final est un **succès** s'il ne subsiste plus d'engagement 

###Avantages des approches par engagement

Quels sont les avantages d'une approche basée sur les engagements? 
 - flexibilité des interactions obtenues, sans avoir à représenter toutes les séquences de manière explicite
 - approche déclarative, permettant de spécifier de manière naturelle les contraintes naissant de l'interaction, et aux agents de raisonner sur les engagements si besoin. 

 

### Terminaison

Pour garantir que la terminaison est possible, il faut garantir: 

1. que le protocole permette toujours, quelque soit l'état, de passer dans un autre état, sauf dans le cas d'un état final
2. que le protocole ne contienne pas de cycle forcé, c'est-à-dire de cycle tels que entrer dans un état du cycle 

Ce serait le cas par exemple de deux agents se déléguant indéfiniment le même engagement entre eux. 
De là on peut énoncer des conditions suffisantes simples. 

> A tout engagement $c$ doit être associé au moins une opération de Discharge, Cancel, ou Release, ou bien il doit être possible par un nombre fini d'opération d'atteindre un engagement $c'$ pour lequel c'est le cas. 
### Autres propriétés: robustesse aux fautes

Le protocole peut être bloqué suite à la l'impossibilité pour un agent de réaliser un engagement qui apparait dans le protocole. 
Une notion de robustesse peut alors être: Est-il possible de (toujours) trouver des moyens alternatifs de réaliser les engagements? 

## 5. Désengagement, pénalités

Jusqu'alors on a essentiellement envisagé des applications coopératives. Dans ce cas, le désengagement peut être réalisé sans contrainte. Dans le cas des applications non-coopératives, l'engagement est typiquement rigide (il ne peut pas être levé). 

Si ce n'est pas le cas, la question est plus complexe. 
*[Sandholm-Lesser: Leveled Commitment Contracting. AI Magazine, 2002]*.

On peut associer à chaque engagement un **coût de désengagement** pour l'agent $x$ engagé (*contractor*) et un coût de désengagement pour l'agent $y$ (*contractee*).  
Les coût de désengagement sont versés à l'autre partie. 
Cela permet aux agents de considérer les offres extérieures, qui peuvent apparaitre après qui peuvent être rationnelles si leur bénéfice excède le coût induit par le désengagement. 

Mais un agent stratégique peut même éventuellement aller plus loin: il peut être réticent à se désengager, sur la base du fait que l'autre va peut être également se désengager: auquel cas il récolterait le prix de désengagement de l'autre, et être libéré de son engagement. Comme l'autre peut faire le même raisonnement, il existe la possibilité de rester bloqué dans des situations sous-optimales pour les deux agents. 

Ce cadre a été analysé en étudiant un jeu en trois phases: 

1. les agents choisissent un engagement
2. les agents reçoivent les nouvelles offres
3. les agents décident (ou non) de se désengager (soit en séquence, soit de manière simultané)

Il est alors possible d'étudier les situations d'équilibre (type Nash) de ce type de jeu.  





> Nicolas Maudet, version 13/03/15
> Written with [StackEdit](https://stackedit.io/).