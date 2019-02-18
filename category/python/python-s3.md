---
title: Python, tester son code 
---

# Python tester son code pour le rendre plus fiable 

## Introduction

Afin de rendre son code plus robuste et plus fiable, il est impératif de mettre en place un ensemble de tests permettant de valider le fonctionnement du code. Les tests sont les garant du résultat. Par ailleurs, pour qu'un code soit facilement testable, il faut qu'il soit modulable et atomique. Un code sera d'autant plus facilement testable que l'on pourra isoler de petites parties et controller spécifiquement l'execution. Il y a énormément de choses à dire autour du test du codexi. Cet atelier permet d'aborder les bases et de pouvoir mettre en place un jeu de test autour d'un module. Pour aller plus loin, vous pouvez vous intéresser au TDD (Test Driven Development), dont la mentalité est le test avant tout. 

## Les tests, qu'est ce que c'est ?

Pour appréhender ce qu'est un test, le plus simple est d'aller faire un peu de revue de code sur notre package bioinfo préféré: la biopython. Si vous récupérez le code de la biopython ou que vous naviguez sur sa documentation (ici: https://github.com/biopython/biopython), vous remarquerez qu'il y a un dossier Tests. Ce dossier contient l'ensemble des tests immplémentés pour les modules de la Biopython et comme nous le verrons plus loin en plus d'aider au développement ils pourront être jouer lors de l'installation du package. Les tests ont donc un intérêt lors du développement mais aussi pendant le déploiement du code.

Regardons de plus près le fichier de test: `test_File.py`

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


class UndoHandleTests(unittest.TestCase):

    def test_one(self):
        h = File.UndoHandle(StringIO(data))
        self.assertEqual(h.readline(), "This\n")
        self.assertEqual(h.peekline(), "is\n")
        self.assertEqual(h.readline(), "is\n")
        # TODO - Meaning of saveline lacking \n?
        h.saveline("saved\n")
        self.assertEqual(h.peekline(), "saved\n")
        h.saveline("another\n")
        self.assertEqual(h.readline(), "another\n")
        self.assertEqual(h.readline(), "saved\n")
        # Test readlines after saveline
        h.saveline("saved again\n")
        lines = h.readlines()
        self.assertEqual(len(lines), 3)
        self.assertEqual(lines[0], "saved again\n")
        self.assertEqual(lines[1], "a multi-line\n")
        self.assertEqual(lines[2], "file")  # no trailing \n
        # should be empty now
        self.assertEqual(h.readline(), "")
        h.saveline("save after empty\n")
        self.assertEqual(h.readline(), "save after empty\n")
        self.assertEqual(h.readline(), "")

    def test_read(self):
        """Test read method"""
        h = File.UndoHandle(StringIO("some text"))
        h.saveline("more text")
        self.assertEqual(h.read(), 'more textsome text')

    def test_undohandle_read_block(self):
        for block in [1, 2, 10]:
            s = StringIO(data)
            h = File.UndoHandle(s)
            h.peekline()
            new = ""
            while True:
                tmp = h.read(block)
                if not tmp:
                    break
                new += tmp
            self.assertEqual(data, new)
            h.close()


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

## Les différents modules de tests

## Intégration dans le packaging de module

## Doctest: le mélange de la documentation et du test

## Implémentation

Nous allons utiliser l'implémentation réalisée dans l'atelier X en 

## Retour sur l'atelier

Nous avons aborder 

Pour compléter cet atelier sur les tests, nous aurions pu aussi aborder la notion de couverture de code. Cette métrique suplémentaire permet de vérifier que l'ensemble du code écrit est utilisé dans au moins un cas et que chacun de ces cas est vérifé par un ou des tests.

## Références


