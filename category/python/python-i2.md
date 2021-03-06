# Suite POO Python

## Implementation - solution

Voici une proposition d'implementation:

```python
#!/usr/bin/env python3.7

class AbstractFeature(object):

    def __init__(self, seqid, start, end, id):

        if self.__class__.__name__ == "AbstractFeature":
            raise Exception("Cannot instanciate AbstractFeature class, need to be sub-classed")
        else:
            self.seqid=seqid
            self.start=start
            self.end=end
            self.id=id

    def __str__(self):

        return f"{self.__class__.__name__}: {self.seqid}-{self.start}-{self.end}-{self.id}"

    def __len__(self):

        return self.end-self.start+1

    def __add__(self, other):

        # prevent against Gene and Transcript add
        if self.__class__ != other.__class__:
            raise Exception("Cannot sum different AbstractFeature sub-classes")

        return len(self) + len(other)

    def __radd__(self, other):

        return len(self)  + other


class Gene(AbstractFeature):

    def __init__(self, seqid, start, end, id):

        self.transcripts = []
        AbstractFeature.__init__(self,seqid, start, end, id)

    def add_transcript(self, mRNA):

        self.transcript.append(mRNA)


class Transcript(AbstractFeature):

     def __init__(self, seqid, start, end, id, gene_id):

        self.gene_id = gene_id
        AbstractFeature.__init__(self,seqid, start, end, id)


class Stats(object):

    @staticmethod
    def mean(l):

        return sum(l)/len(l)


if __name__ == "__main__":

    gene1 = Gene("chr1", 1000, 2000, "gene1")
    mRNA1 = Transcript("chr1", 1000, 2000, "mRNA1", "gene1")
    gene2 = Gene("chr1", 10000, 12000, "gene2")
    mRNA2 = Transcript("chr1", 10000, 12000, "mRNA2", "gene2")
    mRNA3 = Transcript("chr1", 10000, 11400, "mRNA3", "gene2")

    print(gene1+gene2)
    print(sum((gene1,gene2)))
    print(f"Gene mean length: {Stats.mean([gene1,gene2])}")
    print(f"Transcript mean length: {Stats.mean([mRNA1,mRNA2,mRNA3])}")
```

Quelques explications:

* l'implementation de `__add__` est suffisante quand nous sommons 2 objets du même type. Dans le cas de la fonction `sum()`, le premier type à sommer est un `int`, car la première addition est 0 + . Il faut donc implementer "__radd__", pour pouvoir résoudre cette première addition.
* dans la méthode `__init__()` de la classe abstraite, nous implémentons un contôle de type de classe pour éviter l'instanciation de la classe. Cette façon de faire est un peu manuelle, mais nous verrons plus tard qu'il existe des modules dans Python à qui déléguer cette tâche.
* dans la méthode `__add__`, nous implémentons aussi un contrôle de class pour éviter de pouvoir additionner des sous-classes de type différents. 
