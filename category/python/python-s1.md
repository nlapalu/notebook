# Initiation POO Python

# Introduction

Pour cette première approche de la programmation objet en Python, nous allons principalement faire de la revue de code et une implémentation naive d'un petit use case. Le but principal de cette première session est de s'inspirer de code existant dans le but de comprendre la logique de développement et la modularité qu'apporte une conception objet. 

## Revue de code : Bio.Seq.py

Pour rester proche du monde de la bioinformatique, nous allons regarder le package BioPython et plus particulièrement la classe Seq pour commencer. Le code de cette classe est disponible [ici](https://github.com/biopython/biopython/blob/master/Bio/Seq.py). Comme vous allez très vite le constater cette classe est très grande. Son rôle premier est de stocker des informations de séquence, mais elle est capable de faire plus comme par exemple des traitements de type traduction en proteine. Ces traitements peuvent se faire via des appels à ce que l'on nomme des "méthodes", ce qui est l'équivalent des "fonctions" pour les modules. Ci-dessous une première extraction du code de la classe.

```
Class Seq
```

Ce que l'on peut d'ores et déjà remarquer, c'est la facilité de compréhension de chaque méthode de la classe. En effet

 
