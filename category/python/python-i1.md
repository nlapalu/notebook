# Initiation POO Python

## Implementation - solution

Voici une proposition d'implementation:

```python
#!/usr/bin/env python3.7

class Seq(object):

    def __init__(self, seq, seqType):

        self.seq = seq
        if seqType not in ("nuc","prot"):
            raise Exception("Cannot state seqType; nuc or prot")
        self.seqType = seqType

    def __str__(self):

        return self.seq 

    def __eq__(self, other):

        if not isinstance(other, type(self)):
            return NotImplemented
        return (str(self) == str(other))

    def __ne__(self, other):

        return not self == other

    def __gt__(self, other):

        if not isinstance(other, type(self)):
            return NotImplemented

        return (len(self) > len(other))

    def __lt__(self, other):

        return (len(self) < len(other))

    def __len__(self):

        return len(self.seq)

    def rc(self):

        drev = { 'A':'T',
                 'a':'t',
                 'T':'A',
                 't':'a',
                 'G':'C',
                 'g':'c',
                 'C':'G',
                 'c':'g'}

        return ''.join([drev[i] for i in self.seq[::-1]])


class CDS(object):

    def __init__(self,ref,start,stop,name,seq):

        if (stop-start+1)!= len(seq):
            raise Exception("different size for coordinates and seq length")

        self.ref = ref
        self.start = start
        self.stop = stop
        self.name = name
        self.seq = Seq(seq,"nuc")

    def __str__(self):

        return f"CDS: name={self.name},start={self.start},stop={self.stop},ref={self.ref}"

    def __len__(self):

        return len(self.seq)

    def __eq__(self, other):

        return (self.seq == other.seq)

    def __ne__(self, other):

        return not (self == other)

    def __gt__(self, other):

        return (self.seq > other.seq)

    def __lt__(self, other):

        return (self.seq < other.seq)

    def rc(self):

        return self.seq.rc()

    def toBed(self):

        return f"{self.ref}\t{self.start-1}\t{self.stop}" 

    def toFasta(self, size=60):

        return f">{self.name}\n" + "\n".join([str(self.seq)[i:i+size] for i in range(0,len(str(self.seq)),size)])


if  __name__ == '__main__':

    cds1 = CDS("chr1",20,65,"CDS1","ATGAAAAAAAAAATTTTTTTTTTCCCCCCCCCCGGGGGGGGGGGTA")
    print("## cds1 ##")
    print(cds1)
    print("## cds1.rc() ##")
    print(cds1.rc())
    print("## cds1.toBed() ##")
    print(cds1.toBed())
    print("## cds1.toFasta() ##")
    print(cds1.toFasta())
    print("## cds1.toFasta(5) ##")
    print(cds1.toFasta(5))
    print("## len(cds1) ##")
    print(len(cds1))
    cds2 = CDS("chr2",120,155,"CDS2","ATGAAAAAAAAAATTTTTTTTTTCCCCCCCCCCGGG")
    print("## len(cds2) ##")
    print(len(cds2))
    print("## cds1 != cds2 ##")
    print(cds1 != cds2)
    print("## cds1 < cds2 ##")
    print(cds1 < cds2)
    print("## cds1 > cds2 ##")
    print(cds1 > cds2)
    print("## cds1 == cds2 ##")
    print(cds1 == cds2)
```

L'utilisation d'une classe dans une autre classe se nomme en modélisation, une relation de composition. Pour être exacte il y a le lien de composition comme dans le code ci-dessus et il y a aussi le lien d'aggrégation. Dans le premier cas, un object est instancié par un autre objet, et sera détruit en même temps que l'object instanciateur. Dans le deuxième cas, un objet se sert d'un autre objet, mais chaque objet est instancié séparement. Ainsi la destruction de l'objet aggregateur n'a pas d'influence sur l'autre objet. Dans le cas de notre classe `CDS` cela aurait nécessité de construire un objet `Seq` avant et de fournir cet objet à l'objet `CDS`. Pour cela nous aurions pu modifier la méthode instanciatrice de l'objet de cette façon:


Instanciation de l'objet:

```python
class CDS(object):

    def __init__(self,ref,start,stop,name,seq):

        if (stop-start+1)!= len(seq):
            raise Exception("different size for coordinates and seq length")

        self.ref = ref
        self.start = start
        self.stop = stop
        self.name = name
        self.seq = seq

...

seq = Seq("ATGAAAAAAAAAATTTTTTTTTTCCCCCCCCCCGGGGGGGGGGGTA","nuc")
cds1 = CDS("chr1",20,65,"CDS1",seq)
```
au lieu de:

```python
class CDS(object):

    def __init__(self,ref,start,stop,name,seq):

        if (stop-start+1)!= len(seq):
            raise Exception("different size for coordinates and seq length")

        self.ref = ref
        self.start = start
        self.stop = stop
        self.name = name
        self.seq = Seq(seq,"nuc")

...

cds1 = CDS("chr1",20,65,"CDS1","ATGAAAAAAAAAATTTTTTTTTTCCCCCCCCCCGGGGGGGGGGGTA")
```


