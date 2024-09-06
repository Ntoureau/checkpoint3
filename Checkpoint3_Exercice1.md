### **Exercice 1**

## **Partie 1 : Gestion des utilisateurs**

**Q1.1.1**
On peut soit utiliser un script PowerShell pour créer un utilisateur à partir d'un modèle ou le faire par l'interface graphique. J'ai choisi cette seconde hypothèse.

Dans une console Powershell, on identifie l'OU où se trouve "Kelly Rhameur" avec 
```Get-ADUser -Filter "Name -like 'Kelly.Rhameur'" -Properties * ``` 
et on voit qu'elle est directrice des ressources humaines.

Dans le gestionnaire de serveur, sélectionner ```tools```et ```Active Directory Users and Computers```. Naviguer dans l'arborescence jusqu'à l'OU de l'utilisateur recherché et clic droit sur l'utilisateur, ```Copy...```. Modifier les informations pour qu'elles correspondent à celles du nouvel utilisateur puis ```Next```. Définir un mot de passe temporaire et cocher ```The user must change the password on next logon```

![P1copie.png](https://github.com/Ntoureau/checkpoint3/blob/main/P1copie.png?raw=true)

![2.Copiefinale.png](https://github.com/Ntoureau/checkpoint3/blob/main/2.copiefinale.png?raw=true)

**Q1.1.2**

Clic droit sur le domaine puis ```New... ``` puis ```Organisational Unit```

![3.png](https://github.com/Ntoureau/checkpoint3/blob/main/3..png?raw=true)

Pour désactiver et déplacer le compte de Kelly Rhameur, clic droit sur l'utilisateur, ```Move...``` puis sélectionner l'OU que l'on vient de créer.

![4.Move.png](https://github.com/Ntoureau/checkpoint3/blob/main/4.Move.png?raw=true)

Puis clic droit sur l'utilisateur ```Disable Account```

**Q1.1.3**
Clic droit sur le groupe ```GrpUsersDirectionDesRessourcesHumaines```puis ```Properties```, sélectionner l'onglet ```Members```, sélectionner l'utilisateur que l'on souhaite retirer du grouper, puis ```Remove``` puis ```Apply```

![5.Remove.png](https://github.com/Ntoureau/checkpoint3/blob/main/5.Remove.png?raw=true)

**Q1.1.4**
On ouvre l'explorateur de fichiers, les Dossiers individuels sont dans F:\
On crée le dossier ```lionel.marchand```. Puis dans ```Properties```on ouvre l'onglet ```Security``` puis ```Add...```et on ajoute l'utilisateur ```Lionel.Marchand```

![7.Add](https://github.com/Ntoureau/checkpoint3/blob/main/7.Add.png?raw=true)

Dans l'AD, clic droit sur l'utilisateur lionel.marchand puis ```Properties```, Profile et on renseigne le chemin dans ```Local path```: ```F:\lionel.marchand```

![6.dossierperso.png](https://github.com/Ntoureau/checkpoint3/blob/main/6.dossierperso.png?raw=true)

On renomme le dossier "kelly.rhameur" en "kelly.rhameur-ARCHIVE"

## **Partie 2 : Restriction utilisateurs**

**Q1.2.1**
On trouve l'OU de l'utilisateur avec PowerShell 
```Get-ADUser -Filter "Name -like 'Gabriel.Guhl'" -Properties * ```
 , il est dans l'OU Finance.

 On fait clic droit sur l'utilisateur dans l'AD, ```Properties``` puis onglet ```Account```. On sélectionne ```Logon Hours...``` et on définit la plage horaire où il peut se connecter comme suit :

 ![8.horaires.png](https://github.com/Ntoureau/checkpoint3/blob/main/8.horaires.png?raw=true)

 **Q1.2.2**
 Toujours dans les ```Properties```de notre utilisateur, on sélectionne ```Log On To...```, on coche ```The Following computers```et on indique le nom du poste auquel il a le droit de se connecter puis ```ADD```

 ![9.logonto.png](https://github.com/Ntoureau/checkpoint3/blob/main/9.logonto.png?raw=true)

 Pour valider les deux modifications suivantes, on clique sur ```Apply```

 **Q1.2.3**
 Dans le gestionnaire de serveurs, sélectionner ```Tools``` Puis ```Group Policy Management```
 Clic droit sur l'OU ```LabUsers``` puis ```Create a GPO in this domain, and Link it here...``` et lui donner un nom (ici LabUsers-PWPolicy)

![10.CreateGPO.png](https://github.com/Ntoureau/checkpoint3/blob/main/10.CreateGPO.png?raw=true)
 Clic droit sur la GPO nouvellement créée, puis ```Edit```
 On sélectionne en suite : ```Computer Configuration```> ```Policies```> ```Windows Settings```> ```Security Settings``` > ```Account Policies```> ```Password Policy```
 ![11.PWPolicies.png](https://github.com/Ntoureau/checkpoint3/blob/main/11.PWpolicies.png?raw=true)
 On sélectionne ensuite les options que l'on veut appliquer:
 ![12.PWsettings.png](https://github.com/Ntoureau/checkpoint3/blob/main/12.PWsettings.png?raw=true)

 ## **Partie 3 : Lecteurs réseaux**

 **Q1.3.1**
 Les postes clients sont enregistrés dans l'OU ```LabComputers```. Dans le ```Group Policy Management```, on clic droit sur l'OU et ```Create a GPO in this domain and link it here``` que l'on nomme ```Drive Mount```

 ![13.CreateDriveMount.png](https://github.com/Ntoureau/checkpoint3/blob/main/13.CreateDriveMount.png?raw=true)

 On clic droit sur la GPO et on sélectionne ```Edit```
 On navigue dans ```User Configuration```> ```Preferences``` > ```Windows Settings``` > ```Drive Maps```
 On clic droit sur ```Drive Maps```et on sélectionne ```New > Mapped Drive```
 ![14.Mappeddrive.png](https://github.com/Ntoureau/checkpoint3/blob/main/14.Mappeddrive.png?raw=true)

 Dans ```Action```, on sélectionne ```Create```puis le chemin du partage et la lettre du Lecteur comme suit :

 ![15.ShareE.png](https://github.com/Ntoureau/checkpoint3/blob/main/15.ShareE.png?raw=true)
 On refait la même manipulation pour le lecteur F:.

 [16.mappedfinal.png](https://github.com/Ntoureau/checkpoint3/blob/main/16.mappedfinal.png?raw=true)
