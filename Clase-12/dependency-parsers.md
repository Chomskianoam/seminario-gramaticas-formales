# Parsers de dependencias

## Requerimientos
- Python 3
- Spacy
- NLTK
- MaltParser
- Stanza


```python
import spacy
import nltk
import re
from nltk import Tree
from spacy import displacy 
from nltk.parse import malt
```

No poseen distinción entre símbolos no terminales y terminales. Las estructuras representan relaciones de dependencia entre terminales.
Ejemplos de parsers de dependencias:
* [Projective Dependency Parser de NLTK](https://www.nltk.org/_modules/nltk/parse/projectivedependencyparser.html)
* [Maltparser](http://www.maltparser.org/)
* SyntaxNet (Estaba alojado en https://opensource.google.com/projects/syntaxnet, como parte de los recursos de la librería para Inteligencia Artificial TensorFlow de Google, pero en este momento no está disponible y [se rumorea](https://github.com/tensorflow/models/issues/8411) que se lo va a mover al github de [google-research](https://github.com/google-research/google-research))
* [Dependency parser de Spacy](https://spacy.io/usage/linguistic-features#dependency-parse)

## Projective Dependency Parser NLTK


```python
def dep_parser(sentence, grammar):         # define una función llamada dep_parser con dos argumentos
    sentence = sentence.lower()            # convierte a minúscula la oración
    if sentence.endswith('.'):             # si la oración termina con un punto
        sent = re.sub('\.','',sentence)    # se lo quita
    else:                                  # si no
        sent = sentence                    # la toma como está
    sent = sent.split()                    # divide la oración en palabras
    dep_gram = nltk.data.load(grammar, cache=False) # carga la gramática a nltk
    dep_gram = nltk.DependencyGrammar.fromstring(dep_gram) # parsea la gramática como gramática de dependencias
    pdp = nltk.ProjectiveDependencyParser(dep_gram) # aarga la gramática en el parser
    print(dep_gram)                        # imprime mi gramática
    for tree in pdp.parse(sent):           # para cada árbol posible en mi gramática para esa oración
        print(tree)                        # lo imprime
        return(tree)
```


```python
#Para correr el Proyective Dependency Parser

oracion1 = 'Pablo explotó el globo'    # Define la oración a analizar
grammar = 'gramaticas/DG1.txt'        # establece cuál va a ser mi gramática
dep_parser(oracion1, grammar)          # Para correr la función
```

## Non-Projective Dependency Parser NLTK


```python
def npdep_parser(sentence, grammar):                # define una función llamada dep_parser con dos argumentos
    sentence = sentence.lower()                     # convierte a minúscula la oración
    if sentence.endswith('.'):                      # si la oración termina con un punto
        sent = re.sub('\.',' ',sentence)            # se lo quita
    else:                                           # si no
        sent = sentence                             # la toma como está
    sent = sent.split()                             # divide la oración en palabras
    dep_gram = nltk.data.load(grammar, cache=False) # carga la gramática a nltk
    dep_gram = nltk.DependencyGrammar.fromstring(dep_gram) # parsea la gramática como gramática de dependencias
    pdp = nltk.NonprojectiveDependencyParser(dep_gram) # carga la gramática en el parser
    print(sent)
    print(dep_gram)                                  # imprime mi gramática
    g, = pdp.parse(sent)
    print(g.root['word'])
    structure = g.tree()
    print(structure)
    return(structure)
```


```python
#Para correr el Nonproyective Dependency Parser

oracion1 = 'fede fuma el cigarrillo'    # Define la oración a analizar
grammar1 = 'gramaticas/DG1.txt'        # establece cuál va a ser mi gramática
npdep_parser(oracion1, grammar1)        # Para correr la función
```


```python
#Para correr el Nonproyective Dependency Parser

oracion2 = 'quién fuma el cigarrillo'  # Define la oración a analizar
#oracion2 = 'quién dijo fede que fuma'  # Define la oración a analizar
#oracion2 = 'qué dijo fede que fuma'  # Define la oración a analizar
# Habría que arreglar la función npdep_parser para que pueda tomar estas dos últimas oraciones
grammar2 = 'gramaticas/DG2.txt'       # establece cuál va a ser mi gramática
npdep_parser(oracion2, grammar2)        # Para correr la función
```

## PystanfordDependencies

El parser de Stanford se mudó a un nuevo repositorio y cambió su nombre a Stanza. Se puede encontrar la documentación en [https://stanfordnlp.github.io/stanza/](https://stanfordnlp.github.io/stanza/)


```python
import stanza
from stanza.models.common.doc import Token
```


```python
#stanza.download('es') # Baja el modelo para el españo
```


```python
nlp = stanza.Pipeline('es') # Inicializa el modelo de español (con su pipeline de anotación)
nlp
```


```python
doc = nlp("Pablo Neruda escribe poemas") # Anota una oración
doc
```


```python
print(doc.entities)
```


```python
doc.sentences[0].print_dependencies()
```


```python
doc.sentences[0].dependencies
```


```python
doc = nlp("el poeta chileno escribe poemas y la mujer alemana corta un kuchen")
doc
```


```python
doc.entities
```


```python
doc.sentences[0].print_dependencies()
```


```python
def extraer_entidades(stanza_oracion):
    entidades = [] # Creo una lista vacía donde voy a guardar todas las entidades que encuentre en la oración
    for dependencia in stanza_oracion.dependencies: # Recorro las dependencias
        regidor, relacion, dependiente = dependencia # Las dependencias son tuplas de tres elementos que puedo separar
                                                     # en variables
        if regidor.deprel == "nsubj": # Asumimos que queremos encontrar los sujetos, pero podríamos comparar por PoS
                                      # ¿Cómo?
            entidad = [regidor]       # Creo una lista cuyo miembro inicial es el nucleo de la construccion
            for palabra in stanza_oracion.words: # Vuelvo a recorrer las palabras de la oración para encontrar
                                                 # todos los dependientes del sujeto, sean anteriores o posteriores
                if palabra.head == int(regidor.id): # Si el nucleo/regidor/head de una palabra coincide con el id
                    entidad.append(palabra)         # del núcleo de mi construcción, lo sumo a la lista de la entidad
            entidad = sorted(entidad, key=lambda x: x.id) # Ordeno la lista resultante por su número de id para 
                                                          # mantener la linealidad del texto.
            if entidad not in entidades:            # Chequeo que la entidad ya no exista en mi lista porque
                entidades.append(entidad)           # el regidor puede aparecer en más de una dependencia.
            
    return [" ".join([palabra.text for palabra in entidad]) for entidad in entidades] # Me quedo únicamente con el 
                                                       # texto de cada objeto Word y lo devuelvo unido por entidad
```


```python
extraer_entidades(doc.sentences[0])
```


```python
doc.sentences[0].words[0].head
```

## Freeling

[Demo](https://nlp.lsi.upc.edu/freeling/demo/demo.php)

## Malt Parser

A continuación:

- Descargamos Malt Parser en su versión 1.9.2 desde http://maltparser.org/dist/maltparser-1.9.2.tar.gz y lo descomprimimos con el comando `tar`
- El proceso anterior nos tiene que haber creado una carpeta llamada `maltparser-1.9.2`
- Bajamos modelo entrenado engmalt.poly-1.7 de http://www.maltparser.org/mco/english_parser/engmalt.poly-1.7.mco a la carpeta `maltparser-1.9.2`


```python
# descarga maltparser desde la url indicada en una carpeta llamada maltparser-1.9.2 (la crea)
! wget -qO- http://maltparser.org/dist/maltparser-1.9.2.tar.gz | tar -xvz > /dev/null

# descarga el modelo engmalt.poly-1.7 en la carpeta maltparser-1.9.2
! wget -P maltparser-1.9.2 http://www.maltparser.org/mco/english_parser/engmalt.poly-1.7.mco
```

- obtenemos el path absoluto a donde está ubicada esta notebook (`os.path.abspath('.')`)
- unimos esa ubicación con el nombre de la carpeta `maltparser-1.9.2` y hacemos lo mismo con ese resultado y el archivo del modelo (`engmalt.poly-1.7.mco`)
- alojamos ambos paths como variables de entorno


```python
import os

here = os.path.abspath('.')
print('here es: ', here)
maltparser_folder = 'maltparser-1.9.2'
```


```python
print(os.getenv('MALT_PARSER'))
```


```python
os.environ['MALT_PARSER'] = os.path.join(here, maltparser_folder, '')
os.environ['MALT_MODEL'] = os.path.join(here, maltparser_folder, 'engmalt.poly-1.7.mco')
```


```python
print(os.getenv('MALT_PARSER'))
```


```python
! echo $MALT_PARSER
```


```python
maltParser = nltk.parse.malt.MaltParser(os.getenv('MALT_PARSER'), os.getenv('MALT_MODEL'))
```


```python
oracion8 = input()
stemma = maltParser.parse_one(oracion8.split()).tree()
print(stemma)
```

## Spacy - Dependency parser

### Nota para quien no tenga la MV: 

Antes de correr hay que instalar spacy. Con pip3, eso se puede hacer con el comando 

`pip3 install spacy`

Hay que instalar también es_core_news_sm, un modelo entrenado mediante un corpus del español, con el comando

`python3 -m spacy download es_core_news_sm`

Alternativamente puede probarse de instalar es_core_news_md.

`python3 -m spacy download es_core_news_md`

En ese caso, para correrlo hay que cambiar en el código de abajo `es_core_news_sm` por `es_core_news_md`


```python
def gramaticadependencias(sentence):       #Define la función
    nlp = spacy.load('es_core_news_sm')    #Carga el modelo entrenado
    doc = nlp(sentence)                    #define una variable doc con la oración procesada por el modelo
    #for token in doc:               
        #print(token.text, token.dep_, token.head.text, token.head.pos_,
        #    [child for child in token.children])
    #options = {"compact": True, "bg": "#09a3d5","color": "white", "font": "Source Sans Pro"}
    #displacy.render(doc, style='dep', jupyter=True, options=options)
    displacy.render(doc, style='dep', jupyter=True)
```


```python
oracion5 = input('Escribí una oración\n')
gramaticadependencias(oracion5)
```


```python
nlp = spacy.load('es_core_news_sm')
doc = nlp(oracion5)
root = [token for token in doc if token.head == token][0]
print('| {0:10}| {1:5}| {2:7}| {3:7}| {4:30} |'.format('TEXTO','DEP','N_IZQ','N_DER','ANCESTROS'))
print('|'+'='*69+'|')
for descendant in root.subtree:
    assert root is descendant or root.is_ancestor(descendant)
    print('| {0:10}| {1:5}| {2:7}| {3:7}| {4:30} |'.format(
        descendant.text,
        descendant.dep_,
        descendant.n_lefts,
        descendant.n_rights,
        str([ancestor.text for ancestor in descendant.ancestors])
    )
)
```