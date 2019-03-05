# Python tester son code pour le rendre plus fiable

## Implementation - solution

Voici une proposition d'implémentation.

Implementation du test `unittest` dans le script `test_System.py`:

```python
#!/usr/bin/env python3.7

import unittest
import doctest

from System import System

class TestSystem(unittest.TestCase):

    def setUp(self):

#        print("Initializing data for tests")
        self.sys1 =  System(("reac1","reac2"))
        self.sys2 =  System(("reac1","reac3"))

    def tearDown(self):

#        print("tests finished")
        pass

    def test_merge_systems(self):

        expected_sys_merged = System(("reac1","reac2","reac3"))
        get_sys_merged = System.merge_systems([self.sys1, self.sys2])
        self.assertEqual(expected_sys_merged,get_sys_merged)

    def test_count_reactions(self):

        self.assertEqual(2,System.count_reactions([self.sys1]))
        self.assertEqual(4,System.count_reactions([self.sys1, self.sys2]))

    def load_tests(loader, tests, ignore):
        """run doctest tests"""
        tests.addTests(doctest.DocTestSuite("System"))
        return tests


if __name__ == "__main__":
    # add all test from TestCase (starting with test_)
    suite = unittest.TestLoader().loadTestsFromTestCase(TestSystem)
    # add tests from load_tests
    suite.addTests(unittest.TestLoader().loadTestsFromModule(TestSystem))
    # run the test suite
    unittest.TextTestRunner(verbosity=2).run(suite)
```

Modification de la Classe `System` pour intégrer les cas de tests sous forme de `doctest` plutôt que directement dans le `__main__`:

```python
#!/usr/bin/env python3

import doctest

class System(object):

    nbSystems = 0

    def __init__(self, reactions=()):
        """Initialize a new System object.
        >>> syst = System(("reac1", "reac2"))
        >>> print(System.nbSystems)
        1
        >>> print(sorted(syst.reactions))
        ['reac1', 'reac2']
        >>> syst2 = System(("reac1", "reac3"))
        >>> print(System.nbSystems)
        2
        >>> print(sorted(syst2.reactions))
        ['reac1', 'reac3']
        >>> syst3 = System.merge_systems([syst,syst2])
        >>> print(System.nbSystems)
        3
        >>> print(sorted(syst3.reactions))
        ['reac1', 'reac2', 'reac3']
        >>> print(System.count_reactions([syst,syst2,syst3]))
        7
        >>> System.merge_systems([System(("r1","r2")),System(("r1","r3"))]) == System(("r1","r2","r3"))
        True
        """
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
        """Merge system from a list of Systems
        """

        lreactions = []
        for sys in systems:
            lreactions.extend(sys.reactions)

        return cls(set(lreactions))

    @staticmethod
    def count_reactions(systems=[]):

        return sum([len(syst.reactions) for syst in systems])

    def __eq__(self, other):

        return self.__reactions == other.__reactions

    def __repr__(self):

        return repr(self.reactions)

if __name__ == "__main__":

   doctest.testmod()
```

Pour tester les doctest:

```
python3.7 -m doctest System.py -v ou python3.7 System.py -v
```

Pour tester les tests `unittest` et la `doctest`:

```
python3.7 test_System.py -v

test_count_reactions (__main__.TestSystem) ... ok
test_merge_systems (__main__.TestSystem) ... ok
__init__ (System.System)
Doctest: System.System.__init__ ... ok

----------------------------------------------------------------------
Ran 3 tests in 0.004s

OK
```
