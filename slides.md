# RedBaron<br>une approche bottom-up au refactoring en Python

---

# Moi

* Belgique
* Code beaucoup trop (Python (beaucoup), Django, oueb, Haskell…)
* Tourisme des langages de programmation
* Neutrinet (FFDN, Gitoyen)
* La Quadrature du Net/Nurpa
* UrLab (hackerspace à l’ULB/Bruxelles)/HSBxl (hackerspace à Bruxelles)

En beaucoup trop détaillé :
[http://worlddomination.be/about/about.html](http://worlddomination.be/about/about.html)

---

# Avant de commencer : révisions

---

# Refactoring (Eclipse)

![refactoring.png](refactoring.png)

---

# Abstract Syntax Tree (AST)

![ast.png](ast.png)

---

# Plan

* Pourquoi ?
* Solution au premier problème (baron)
* Solution au deuxième problème (RedBaron)
* Conclusion

---

# Pourquoi ?

---

# Refactoring custom

* J’ai toujours voulu écrire du code pour modifier mon code
* Très difficile : string énorme sans sens, analyse, déplacement, trop de possibilités, syntaxe
* Frustrant, plein de cas où "ah, si seulement je pouvais scripter cette modification !"
* Comme une blessure à la lèvre
* Générer du code aussi
* Seulement une poignée de gens font ça

---

# Ast.py

![ast.py.png](ast.py.png)

---

# Ast.py

Pas lossless !

    ast_to_code(code_to_ast(code_source)) != source_code

(Commentaires, formatting)

(Et ast\_to\_code n’existe même pas de manière standard).

---

# Ast.py

API Sax like

    !python
    class KeyAttributesFinder(ast.NodeVisitor):
        def visit_Assign(self, assign_node):
            # ...

        def visit_FunctionDef(self, function_node):
            # ...

        # visit_...

Super chiant, impossible à utiliser dans IPython efficacement.

---

# Génération de code

Django (memopol et co) :

    donnees.json -> models.py + import.py

Autre projet :

    Générer du boiler plate en lisant des models de db

---

# pythonfmt

Auto-formater du code python

---

# Refactoring en python

* BycleRepairMan
* Rope (ast.py + regexs)
* PyCharm (?)
* Hyper dur : je suis en (x,y) dans un fichier, y a quoi autour de moi ?

---

# Refactoring: top to bottom

![refactoring1.png](refactoring1.png)

---

# Conclusion : 2 problèmes

* Il manque la bonne abstraction
* Il manque la bonne interface

---

# Solution 1 : l’abstraction -> Baron

---

# Baron

* ast lossless ! (FST == Full Syntax Tree)
* source == ast\_to\_code(code\_to\_ast(source))
* transforme un problème d’analyse de code en parcours/modification d'un graphe
* output du json pour compatibilité maximum (+ structure de donnée simple)

---

# Exemple

    !python

    from baron.helpers import show

    print show("1 + 2")

    [
        {
            "first_formatting": [
                {
                    "type": "space", 
                    "value": " "
                }
            ], 
            "value": "+", 
            "second_formatting": [
                {
                    "type": "space", 
                    "value": " "
                }
            ], 
            "second": {
                "section": "number", 
                "type": "int", 
                "value": "2"
            }, 
            "type": "binary_operator", 
            "first": {
                "section": "number", 
                "type": "int", 
                "value": "1"
            }
        }
    ]

---

![refactoring2.png](refactoring2.png)

---

# État du projet

*1 an de boulot (j'ai dû apprendre)*

* +1000 tests (TDD)
* marche sur le top 100 de pypi
* utilities: position\_to\_path, position\_to\_node, boundinx\_box, walker…
* (encore quelques bugs ultra rares)
* entièrement documenté

---

# Solution 2 : l'interface -> RedBaron

---

# Plan

* principe
* exploration (query)
* modification
* abstraction des listes

---

# RedBaron

* API au dessus de Baron
* Comme BeautifulSoup/jQuery : mapping structure de donnée -> objects
* Pour l’humain, user friendly autant que possible
* Pensé, entre autre, pour être utilisé dans IPython (ou bpython)

---

# RedBaron

API super simple :

    !python
    from redbaron import RedBaron

    red = RedBaron("string représentant du code source")
    # ...
    red.dumps()  # code source

---

# Intuitif (autant que possible)

Surcharge de \_\_repr\_\_:

BeautifulSoup :

![soup.png](soup.png)

---

# Intuitif (autant que possible)

Surcharge de \_\_repr\_\_:

BeautifulSoup :

![soup.png](soup.png)

RedBaron:

![repr.png](repr.png)

---

# Auto descriptif

RedBaron :

![at_0.png](at_0.png)

---

# Auto descriptif

RedBaron :

![at_0.png](at_0.png)

".help()"

![help.png](help.png)

---

# Exploration

Comme BeautifulSoup :

    !python
    red = RedBaron("a = 42\ndef test_chocolat(): pass")
    red.find("name")
    red.find("int", value=42)
    red.find("def", name="g:test_*")
    red.find("assignment", lambda x: x.target.dumps() == "INSTALLED_APPS")

    red.find_all("name")
    red.find_all(("name", "int"))
    red.find_all("def", arguments=lambda x: len(x) == 3)
    red.find_all("def", recursive=False)

---

# Exploration

Comme BeautifulSoup :

    !python
    red = RedBaron("a = 42\ndef test_chocolat(): pass")
    red.find("name")
    red.find("int", value=42)
    red.find("def", name="g:test_*")
    red.find("assignment", lambda x: x.target.dumps() == "INSTALLED_APPS")

    red.find_all("name")
    red.find_all(("name", "int"))
    red.find_all("def", arguments=lambda x: len(x) == 3)
    red.find_all("def", recursive=False)

Raccourcis (comme BeautifulSoup) :

    !python
    red = RedBaron("a = 42\ndef test_chocolat(): pass")
    red.name
    red.int
    red.else_

    red("name")
    red(("name", "int"))
    red("def", arguments=lambda x: len(x) == 3)

---

# Modification

Comment modifier une node ?

    !python
    from redbaron import RedBaron, BinaryOperatorNode

    red = RedBaron("a = 'plop'")
    red[O].value  # 'plop'
    red[0].value = BinaryOperatorNode({'first_formatting':[{'type': 'space',
    'value': ' '}], 'value': '+', 'second_formatting': [{'type': 'space',
    'value': ' '}], 'second': {'section': 'number', 'type': 'int', 'value':
    '1'}, 'type': 'binary_operator', 'first': {'section': 'number', 'type':
    'int', 'value': '1'}})

Pas hyper pratique…

---

# Magie de \_\_setattr\_\_

    !python
    from redbaron import RedBaron, BinaryOperatorNode

    red = RedBaron("a = 'plop'")
    red[0].value = "1 + 1"

    # marche aussi avec : nodes redbaron et ast

Marche pour __toutes__ les nodes.

---

# Modifications avancées :

Autre problème : quel est le corps/body de la fonction "bar" ?

    !python
    class Foo():
        def bar(self):
            pass

        def baz(self):
            pass

---

# Modifications avancées :

Autre problème : quel est le corps/body de la fonction "bar" ?

    !python
    class Foo():
        def bar(self):
            pass

        def baz(self):
            pass

Expected:

![expected.png](expected.png)

---

# Modifications avancées :

Autre problème : quel est le corps/body de la fonction "bar" ?

    !python
    class Foo():
        def bar(self):
            pass

        def baz(self):
            pass

Expected:

![expected.png](expected.png)

Reality:

![reality.png](reality.png)

---

# Solution : magie !

![magic.gif](magic.gif)

    !python

    red.find("def", name="bar").value = "pass"
    red.find("def", name="bar").value = "pass\n"
    red.find("def", name="bar").value = "    pass\n"
    red.find("def", name="bar").value = "    pass\n    "
    red.find("def", name="bar").value = "        pass\n        "
    red.find("def", name="bar").value = "\n    pass\n    "
    # etc ..

Pareil pour les : *else*, *exceptions*, *finally*, *elif*…

---

# Listes

Problème: combien d'éléments dans le corps de cette liste ? <code>['a', 'b', 'c']</code>

---

# Listes


Problème: combien d'éléments dans le corps de cette liste ?

**Expected**:

![list_expected.png](list_expected.png)

---

# Listes


Problème : combien d’éléments dans le corps de cette liste ?

**Expected**:

![list_expected.png](list_expected.png)

**Reality**:

![list_reality.png](list_reality.png)

---

# Listes : solutions

Solution : des "proxy" de listes qui donnent la même API que les listes Python et gèrent le formatting pour vous.

**Reality again**:

![list_expected.png](list_expected.png)

Marche pour les :

* "," (avec et sans indentation)
* les ".", par exemple: <code>a.b.c().pouet[stuff]</code>
* les lignes séparées par des retours à la ligne (corps des fonctions, "blocs python")

---

# Quelques exemples

    !python

    # renommer un 'name' (attention : renommera pas tout)
    for i in red('name', value='pouet'): i.value = 'plop'

---

# Quelques exemples

    !python

    # renommer un 'name' (attention : renommera pas tout)
    for i in red('name', value='pouet'): i.value = 'plop'

    # installer une django app
    red.find("assign", target=lambda x: x.dumps() == 'INSTALLED_APPS').\
        value.append("'debug_toolbar.apps.DebugToolbarConfig'")

---

# Quelques exemples

    !python

    # renommer un 'name' (attention : renommera pas tout)
    for i in red('name', value='pouet'): i.value = 'plop'

    # installer une django app
    red.find("assign", target=lambda x: x.dumps() == 'INSTALLED_APPS').\
        value.append("'debug_toolbar.apps.DebugToolbarConfig'")

    # lines_profiler
    red('def', recursive=False).\
        map(lambda x: x.decorators.insert(0, '@profile'))

---

# Quelques exemples

    !python

    # renommer un 'name' (attention : renommera pas tout)
    for i in red('name', value='pouet'): i.value = 'plop'

    # installer une django app
    red.find("assign", target=lambda x: x.dumps() == 'INSTALLED_APPS').\
        value.append("'debug_toolbar.apps.DebugToolbarConfig'")

    # lines_profiler
    red('def', recursive=False).\
        map(lambda x: x.decorators.insert(0, '@profile'))

    # les retirer
    red("decorator", lambda x: x.dumps() == "@decorator").\
        map(lambda x: x.parent.parent.decorators.remove(x))

---

# État

* +1200 tests
* entièrement documenté (plein d’exemples) (bémol)
* librairie de référence pour écrire du code qui modifie du code
* encore un peu rugueux (alpha ?)
* devrait remplir 80% des cas
* **pas** d’analyse statique (pas encore ?)

---

![refactoring3.png](refactoring3.png)

---

# Conclusion

---

![kent.png](kent.png)

---

# « Mec, t’es en train de coder le nouvel 'ed' du 21e siècle avec 4 niveaux d’abstractions en plus »<br>un pote, fin bourré

---

# Infos

RedBaron :

* [https://github.com/psycojoker/redbaron](https://github.com/psycojoker/redbaron)
* [https://baron.readthedocs.org](https://baron.readthedocs.org)
* <code>pip install redbaron</code>

Baron :

* [https://github.com/psycojoker/baron](https://github.com/psycojoker/baron)
* [https://redbaron.readthedocs.org](https://redbaron.readthedocs.org)
* <code>pip install baron</code>

Contacts :

* Moi: cortex@worlddomination.be
* Irc: irc.freenode.net#baron
