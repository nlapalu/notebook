---
title: Python, tester son code 
---

# Python tester son code pour le rendre plus fiable 

## Introduction

Afin de rendre son code plus robuste et plus fiable, il est impératif de mettre en place un ensemble de tests permettant de valider le fonctionnement du code. Les tests sont les garant du résultat. Par ailleurs, pour qu'un code soit facilement testable, il faut qu'il soit modulable et atomique. Un code sera d'autant plus facilement testable que l'on pourra isoler de petites parties et controller spécifiquement l'execution. Il y a énormément de choses à dire autour du test du codexi. Cet atelier permet d'aborder les bases et de pouvoir mettre en place un jeu de test autour d'un module. Pour aller plus loin, vous pouvez vous intéresser au TDD (Test Driven Development), dont la mentalité est le test avant tout. 

## Les tests, qu'est ce que c'est ?

Pour appréhender ce qu'est un test, le plus simple est d'aller faire un peu de revue de code sur notre package bioinfo préféré: la biopython. Si vous récupérez le code de la biopython ou que vous naviguez sur sa documentation (ici: https://github.com/biopython/biopython), vous remarquerez qu'il y a un dossier `Tests`. Ce dossier contient l'ensemble des tests immplémentés pour les modules de la Biopython et comme nous le verrons plus loin en plus d'aider au développement ils pourront être jouer lors de l'installation du package. Les tests ont donc un intérêt lors du développement mais aussi pendant le déploiement du code.

Regardons de plus près le fichier de test: `test_File.py` (https://github.com/biopython/biopython/blob/master/Tests/test_File.py)

```python
from __future__ import print_function

import os.path
import shutil
import sys
import tempfile
import unittest

from Bio import bgzf
from Bio import File
from Bio._py3k import StringIO

data = """This
is
a multi-line
file"""

...


class RandomAccess(unittest.TestCase):

    def test_plain(self):
        with File._open_for_random_access("Quality/example.fastq") as handle:
            self.assertTrue("r" in handle.mode)
            self.assertTrue("b" in handle.mode)

    def test_bgzf(self):
        with File._open_for_random_access("Quality/example.fastq.bgz") as handle:
            self.assertIsInstance(handle, bgzf.BgzfReader)

    def test_gzip(self):
        self.assertRaises(ValueError,
                          File._open_for_random_access,
                          "Quality/example.fastq.gz")
```

Le fichier de tests comporte plusieurs classes qui héritent toute de `unittest.TestCase`, classe `TestCase` du module de test `unittest`. Chaque classe implémente plusieurs méthodes commençant par **_test_**, qui correspond plus ou moins à un test. Néanmoins chaque test peut comporter plusieurs valeurs testées. En héritant de `TestCase`, les classes créées vont bénéficier d'un grand ensemble de méthodes de test. Par exemple, la première méthode `test_plain()` va éssayer 2 assertions. **_Quand j'ouvre le fichier fastq (de test) "Quality/example.fastq" via la méthode `_open_for_random_access()`, est ce que le fichier est en mode 'r' et 'b' ?_**


Regardons maintenant la classe de test suivante. 

```python
class AsHandleTestCase(unittest.TestCase):

    def setUp(self):
        # Create a directory to work in
        self.temp_dir = tempfile.mkdtemp(prefix='biopython-test')

    def tearDown(self):
        shutil.rmtree(self.temp_dir)

    def _path(self, *args):
        return os.path.join(self.temp_dir, *args)

    def test_handle(self):
        "Test as_handle with a file-like object argument"
        p = self._path('test_file.fasta')
        with open(p, 'wb') as fp:
            with File.as_handle(fp) as handle:
                self.assertEqual(fp, handle, "as_handle should "
                                 "return argument when given a "
                                 "file-like object")
                self.assertFalse(handle.closed)

            self.assertFalse(handle.closed,
                             "Exiting as_handle given a file-like object "
                             "should not close the file")

    def test_string_path(self):
        "Test as_handle with a string path argument"
        p = self._path('test_file.fasta')
        mode = 'wb'
        with File.as_handle(p, mode=mode) as handle:
            self.assertEqual(p, handle.name)
            self.assertEqual(mode, handle.mode)
            self.assertFalse(handle.closed)
        self.assertTrue(handle.closed)

    @unittest.skipIf(
        sys.version_info < (3, 6),
        'Passing Path objects to File.as_handle requires Python >= 3.6',
    )
    def test_path_object(self):
        "Test as_handle with a pathlib.Path object"
        from pathlib import Path
        p = Path(self._path('test_file.fasta'))
        mode = 'wb'
        with File.as_handle(p, mode=mode) as handle:
            self.assertEqual(str(p.absolute()), handle.name)
            self.assertEqual(mode, handle.mode)
            self.assertFalse(handle.closed)
        self.assertTrue(handle.closed)

    @unittest.skipIf(
        sys.version_info < (3, 6),
        'Passing path-like objects to File.as_handle requires Python >= 3.6',
    )
    def test_custom_path_like_object(self):
        "Test as_handle with a custom path-like object"
        class CustomPathLike:
            def __init__(self, path):
                self.path = path

            def __fspath__(self):
                return self.path

        p = CustomPathLike(self._path('test_file.fasta'))
        mode = 'wb'
        with File.as_handle(p, mode=mode) as handle:
            self.assertEqual(p.path, handle.name)
            self.assertEqual(mode, handle.mode)
            self.assertFalse(handle.closed)
        self.assertTrue(handle.closed)

    def test_stringio(self):
        s = StringIO()
        with File.as_handle(s) as handle:
            self.assertIs(s, handle)


if __name__ == "__main__":
    runner = unittest.TextTestRunner(verbosity=2)
    unittest.main(testRunner=runner)
```

Plusieurs choses intéressantes sont à noter. L'héritage de `TestCase` vous donne la possibilité de surcharger les méthodes `setUp()` et `tearDown()` qui seront utilisées respectivement avant les tests et après les tests. Vous pouvez donc via ces méthodes, configurer plus facilement votre environnement de test en spécifiant des atributs partagés. 

Nous pouvons voir aussi que certaines méthodes comportent des décorateurs (`@unittest.skipIf`). Plusieurs décorateurs existent permettant pas exemple de ne pas executer le test, ce qui est utile en phase de développement ou de résolution de bugs, ou encore de spéficier des contraintes pour son execution (module necessaire, version de Python, ...).

Il existe donc un grand nombre de méthodes d'assertion ou de control d'execution des tests. Afin de ne pas reprendre l'ensemble des possibilités ici, le plus simple est d'explorer la documentation de [unittest](https://docs.python.org/3/library/unittest.html). 

Exécutons maintenant les tests. Grâce au `__main__` implémenté dans le fichier nous pouvons lancer la suite de tests:

`python test_File.py`

```
test_custom_path_like_object (__main__.AsHandleTestCase)
Test as_handle with a custom path-like object ... skipped 'Passing path-like objects to File.as_handle requires Python >= 3.6'
test_handle (__main__.AsHandleTestCase)
Test as_handle with a file-like object argument ... ok
test_path_object (__main__.AsHandleTestCase)
Test as_handle with a pathlib.Path object ... skipped 'Passing Path objects to File.as_handle requires Python >= 3.6'
test_string_path (__main__.AsHandleTestCase)
Test as_handle with a string path argument ... ok
test_stringio (__main__.AsHandleTestCase) ... ok
test_bgzf (__main__.RandomAccess) ... ok
test_gzip (__main__.RandomAccess) ... FAIL
test_plain (__main__.RandomAccess) ... ok
test_one (__main__.UndoHandleTests) ... ok
test_read (__main__.UndoHandleTests)
Test read method ... ok
test_undohandle_read_block (__main__.UndoHandleTests) ... ok

======================================================================
FAIL: test_gzip (__main__.RandomAccess)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "test_File.py", line 85, in test_gzip
    "Quality/example.fastq.gz")
AssertionError: ValueError not raised

----------------------------------------------------------------------
Ran 11 tests in 0.002s

FAILED (failures=1, skipped=2)
```

Nous pouvons voir que les tests se sont executés les uns après les autres dans l'orde alphabétique. Un test a échoué, le détail apparaît alors en fin de la suite de test. 

## Doctest: le mélange de la documentation et du test

Une autre façon de faire des tests est l'utilisation du module `doctest` de python. Le but principal de `doctest` est de pouvoir très facilement intégrer des cas de tests du code au plus près de l'implémentation. Cela limite le nombre de fichier crée en revanche cela peut aussi alourdir le code et rendre moins lisible la partie 'vrai code' de votre fichier. L'utilisation de doctest n'est pas incompatible avec d'autre type de module, c'est simplement une philosphie différente. 
Le fonctionnement de cas de test imbriqué au milieu du code, se fait par la recherche de `>>>` au milieu de commentaires. Les tests peuvent ensuite être joués de différentes manières. Pour bien comprendre le lancement des tests via `doctest`, il faut s'imaginer l'ouverture d'un shell Python dans lequel vous lancez des commandes. Chaque retour de vos commandes peut être ensuite testé.
Comme toujours, une revue de code est bien plus efficace que de longues explications. Intéressons nous donc au module [Bio.Seq](https://github.com/biopython/biopython/blob/master/Bio/Seq.py) de la biopython. Voici un extrait de sa méthode `__init()__`:

```python
def __init__(self, data, alphabet=Alphabet.generic_alphabet):
    """Create a Seq object.
    Arguments:
    - seq - Sequence, required (string)
    - alphabet - Optional argument, an Alphabet object from
    Bio.Alphabet
    You will typically use Bio.SeqIO to read in sequences from files as
    SeqRecord objects, whose sequence will be exposed as a Seq object via
    the seq property.
    However, will often want to create your own Seq objects directly:
    >>> from Bio.Seq import Seq
    >>> from Bio.Alphabet import IUPAC
    >>> my_seq = Seq("MKQHKAMIVALIVICITAVVAALVTRKDLCEVHIRTGQTEVAVF",
    ...              IUPAC.protein)
    >>> my_seq
    Seq('MKQHKAMIVALIVICITAVVAALVTRKDLCEVHIRTGQTEVAVF', IUPACProtein())
    >>> print(my_seq)
    MKQHKAMIVALIVICITAVVAALVTRKDLCEVHIRTGQTEVAVF
    >>> my_seq.alphabet
    IUPACProtein()
    """
    # Enforce string storage
    if not isinstance(data, basestring):
        raise TypeError("The sequence data given to a Seq object should "
                        "be a string (not another Seq object etc)")
    self._data = data
    self.alphabet = alphabet # Seq API requirement
```
On peut voir que 3 tests ont été implémentés. Le premier qui test le retour de `my_seq`, le second de `print(my_seq)` et enfin `my_seq.alphabet`. Maintenant, si nous regardons la fin du fichier, le `__main__` de l'objet a été implémenté avec:

```python 
def _test():
    """Run the Bio.Seq module's doctests (PRIVATE)."""
    print("Running doctests...")
    import doctest
    doctest.testmod(optionflags=doctest.IGNORE_EXCEPTION_DETAIL)
    print("Done")

if __name__ == "__main__":
_test()

``` 
Ainsi, l'objet (module) peut-être directement utilisé comme un script si il est appelé directement (via `__main__`). Dans ce cas, il y aura execution des tests `doctest` via la méthode `doctest.testmod()`. C'est ce que nous faisons si dessous:


```
python3.7 -m Bio.Seq
Running doctests...
/home/nlapalu/Workspace/Github/biopython/Bio/Seq.py:441: BiopythonDeprecationWarning: This method is obsolete; please use str(my_seq) instead of my_seq.tostring().
  BiopythonDeprecationWarning)
/home/nlapalu/Workspace/Github/biopython/Bio/Seq.py:2626: BiopythonWarning: This table contains 2 codon(s) which code(s) for both STOP and an amino acid (e.g. 'TGA' -> 'W' or STOP). Such codons will be translated as amino acid.
  BiopythonWarning)
Done
```
Pour avoir plus d'info, il faut demander le mode verbose

```
python3.7 -m Bio.Seq -v 
/home/nlapalu/Workspace/Github/biopython/Bio/Seq.py:441: BiopythonDeprecationWarning: This method is obsolete; please use str(my_seq) instead of my_seq.tostring().
  BiopythonDeprecationWarning)
Running doctests...
Trying:
    from Bio.Seq import MutableSeq
Expecting nothing
ok
Trying:
    from Bio.Alphabet import generic_dna
Expecting nothing
ok
Trying:
    my_seq = MutableSeq("ACTCGTCGTCG", generic_dna)
Expecting nothing
ok
Trying:
    my_seq
Expecting:
    MutableSeq('ACTCGTCGTCG', DNAAlphabet())
ok

...

   6 tests in __main__.UnknownSeq.transcribe
  10 tests in __main__.UnknownSeq.translate
  10 tests in __main__.UnknownSeq.ungap
   7 tests in __main__.UnknownSeq.upper
  10 tests in __main__._translate_str
   1 tests in __main__.back_transcribe
   1 tests in __main__.complement
   1 tests in __main__.reverse_complement
   1 tests in __main__.transcribe
  15 tests in __main__.translate
449 tests in 103 items.
449 passed and 0 failed.
Test passed.
Done
```

Un des problème de doctest est sa sensibilité au retour du test. En effet, ce qui est vraiement testé est une chaine de caractères. Il suffit donc d'un léger écart dans la chaîne de retour pour que le test soit faux. Si vous souhiatez donc utiliser `doctest` il faut essayer de faire des tests qui limiteront ces possibles problèmes.

## Les différents modules de tests

## Intégration dans le packaging de module



## Implémentation

Nous allons utiliser l'implémentation de la class `System` réalisée dans l'atelier [POO Python suite](/category/python/python-s2.html) pour mettre en place une série de tests. Afin de couvrir les notions vues dans l'atelier, nous allons à la fois implémenter des tests unitaires avec Unittest et doctest. Pour rappel, voici l'implémentation de la class `System`: 

```python
class System(object):

    nbSystems = 0

    def __init__(self, reactions=()):
        """Initialize a new System object."""
        self.__reactions = set(reactions)
        self.__class__.nbSystems += 1

    def __del__(self):
        self.__class__.nbSystems -= 1

    @property
    def reactions(self):
        return self.__reactions

    @reactions.setter
    def reactions(self, reactions=()):
        self.__reactions = set(reactions)

    @classmethod
    def merge_systems(cls,systems=[]):

        lreactions = []
        for sys in systems:
            lreactions.extend(sys.reactions)

        return cls(set(lreactions))

    @staticmethod
    def count_reactions(systems=[]):

        return sum([len(syst.reactions) for syst in systems])

if __name__ == "__main__":

   syst = System(("reac1", "reac2"))
   print(System.nbSystems)
   print(syst.reactions)
   syst2 = System(("reac1", "reac3"))
   print(System.nbSystems)
   print(syst2.reactions)
   syst3 = System.merge_systems([syst,syst2])
   print(System.nbSystems)
   print(syst3.reactions)
   print(System.count_reactions([syst,syst2,syst3]))


1
set(['reac1', 'reac2'])
2
set(['reac1', 'reac3'])
3
set(['reac1', 'reac2', 'reac3'])
7
```

Tout d'abord, vous allez implementer une classe de test `TestSystem` qui héritera de `unittest.TestCase`. Cette classe implémentera 2 méthodes qui vous permettront de tester `merge_systems()` et `count_reactions()` de la classe `System`. Point important, il vous sera nécessaire de modifier l'implémentation de la classe `System` pour pouvoir tester l'égalité de 2 instances de `System`. Pour cela souvenez vous ([atelier 1: POO]()) des méthodes dunder qui permettent de comparer des objets et des instances d'objets. 

La solution est [ici](python-i3.html)

## Retour sur l'atelier

Nous avons aborder 

Pour compléter cet atelier sur les tests, nous aurions pu aussi aborder la notion de couverture de code. Cette métrique suplémentaire permet de vérifier que l'ensemble du code écrit est utilisé dans au moins un cas et que chacun de ces cas est vérifé par un ou des tests.

## Références

* [doc officielle - doctest](https://docs.python.org/3.7/library/doctest.html)
* [test-unitaires](http://sametmax.com/un-gros-guide-bien-gras-sur-les-tests-unitaires-en-python-partie-4/)
* [test tous les aspects](https://www.python-course.eu/python3_tests.php)

* [test](https://openclassrooms.com/fr/courses/235344-apprenez-a-programmer-en-python/2235416-creez-des-tests-unitaires-avec-unittest)
