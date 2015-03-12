<head>
...
    <script type="text/javascript"
            src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML">
    </script>
...
</head>

# FoSyMA -- Cours 9


## 1. Protocoles d'interactions

> **Rappel:** en JADE, vous disposez de l'agent sniffer, qui permet de visualiser simplement les séquences de messages

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
 
 - la **terminaison**: la stratégie et le protocole permettent-ils d'atteindre un état final? 
 
 - le **succès** ou l'**optimalité**: l'état atteint est-il satisfaisant? Permet-il d'atteindre une solution optimale parmi toutes les solutions possibles? (bien sûr, cette notion d'optimailité peut être définit de multiple manière selon les domaines)
 
 - l'**efficacité**: la quantité d'information échangée au cours du protocole est-elle minimale? 

(Verification of Protocols for Automated Negotiation)

### Conformité 
(a priori: diffiicile d'accéder aux stratégies)
(on-the-fly: problème de l'accès aux messages qui peuvent être retardés.) 
(en JADE: sniffer, sniffer étendu?)

### Complexité de communication

Que mesurer? Le nombre de messages? 
Voici un exemple : 

> Les deux agents Alice et Bob souhaitent se coordonner pour donner la priorité à celui ou celle qui a le plus de place disponible dans son sac. Quel protocole employer? 

Protocole 1: Révélation asymétrique
```sequence
Alice->Bob: Il me reste k kilos 
Note right of Bob: Bob calcule 
Bob-->Alice: J'y vais 
Bob-->Alice: Vas-y
```

` 1. Anne envoie sa valuation
2. Bob calcule qui doit aller 
`
 
Protocole 2: Enchère

```sequence
Alice->Bob: J'ai au moins 0 kilo
Note right of Bob: Bob calcule 
Bob-->Alice: J'ai au moins 1 kilo de plus
Bob-->Alice: Vas-y
Alice-->Bob: J'ai au moins 1 kilo de plus
Alice-->Bob: Vas-y
Bob-->Alice: J'ai au moins 1 kilo de plus
Bob-->Alice: Vas-y
```

```python
p = 0; nextSpeaker = Anne; 
while  
Anne 
```

Protocole 3: 

```sequence
Alice->Bob: Il me reste beacoup de place (2 ou 3) 
Alice-->Bob: Il me reste peu de place (0 ou 1)
Note right of Bob: Bob calcule 
Bob-->Alice: J'y vais 
Bob-->Alice: Vas-y
```

```java
if anne.getBackPackFreeSpace()<=1 then anne.hasLowVal=1
anne.sendMessage()
if (alice.hasLowSpace==True) {
	if (bob.getBackPackFreeSpace()==0)
	{sendMessage(youGo)}
	else 
	{sendMessage(IGo)}}
else { //alice has high space
	if (bob.getBackPackFreeSpace()==3)
	{sendMessage(IGo)}
	else
	{sendMessage(youGo)}}

	  
```




## 3. Machines à engagements

Plusieurs approches ont été proposées pour réguler l'interaction non pas sur la base de séquences légales de messages (machines à états finis, réseaux de Petri), mais en s'appuyant sur 


Un engagement peut être: 
1. réalisé
2.   relaché

$$C(x,y)=...$$



> Written with [StackEdit](https://stackedit.io/).
