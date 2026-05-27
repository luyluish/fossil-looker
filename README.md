# 🦖 Fossil Looker

## Introdução

Fósseis de dinossauros podem ser diferenciados de acordo com características visuais identificadas nos próprios fósseis. Seguindo uma hierarquia taxonômica, o sistema de classificação pode ser definido como um algoritmo que começa coletando informações mais superficiais, como a disposição do fóssil (Bípede/Quadrúpede) e vai afunilando características mais específicas, como presença de apêndices, comprimento dos membros, garras, formatos dos dentes, e etc, em uma ordem que começa lá nos clados e se especifica apenas em diferenças mínimas.

A ideia desse sistema é dar a possibilidade do usuário fornecer todas as características que ele consegue ver no fóssil, independente de ordem ou prioridade, e através de uma série de regras, o sistema percorrer a hierarquia de níveis para inferir qual o espécime (ou gênero, já que as diferenças de espécie são muitas vezes bem mínimas). 

A inferência acontece em 3 níveis:

1. Características visuais
2. Agrupamento taxonômico
3. Classificação de espécie

---

## Regras em Linguagem Natural

### 🔹 Nível 1 — Características visuais

* Regra 1: Se tem dentes afiados → é carnívoro
* Regra 2: Se tem dentes achatados → é herbívoro
* Regra 3: Se a disposição é bípede → locomoção bípede
* Regra 4: Se a disposição é quadrúpede → locomoção quadrúpede

---

### 🔹 Nível 2 — Agrupamento taxonômico

* Regra 5: Se é carnívoro e é bípede, é do grupo Terópode
* Regra 6: Se é carnívoro e quadrúpede, ele não existe, porque não tem registros destes. Logo, o sistema gera um erro
* Regra 7: Se é herbívoro e bípede, é do grupo Ornitópode
* Regra 8: Se é herbívoro e quadrúpede, ele se encaixa em um dos outros grupos de Herbíviros

---

### 🔹 Nível 3 — Classificação de espécie

* Regra 9: Se é terópode e tem chifres, é um Ceratosaurus
* Regra 10: Se é Terópode e possui um apêndice dorsal, é um Spinosaurus
* Regra 11: Se é Terópode e possui uma crista, é um Cryolophosaurus
* Regra 12: Se é Ornitópode e possui um apêndice dorsal, é um Ouranosaurus
* Regra 13: Se é Ornitópode e possui uma crista, é um Parasaurolophus
* Regra 14: Se é Ornitópode e tem chifres, ele não existe, porque não tem registro destes também. O sistema lança um erro
* Regra 15: Se é outro grupo dos herbívoros e tem chifres, é um Triceratops
* Regra 16: Se é de outro grupo dos herbívoros e possui um apêndice dorsal, é um Stegosaurus
* Regra 17: Se é de outro grupo dos herbívoros e possui uma crista, é um Pachyrhinosaurus

---

## Casos de Teste

### ✅ Caso 1 — Spinosaurus

![Spinosaurus](https://upload.wikimedia.org/wikipedia/commons/thumb/e/e4/Spinosaurus_aegyptiacus_life_reconstruction.png/1920px-Spinosaurus_aegyptiacus_life_reconstruction.png)

```python
engine.declare(Dino(
    dentes='afiados',
    bipede=True,
    dorsal=True
))
```

**Resultado esperado:**

```
R1: carnívoro
R3: bípede
R5: harnívoro e bípede => Terópode
R10: terópode e apêndice dorsal => Spinosaurus
```

**Motivo:**
A combinação de dieta carnívora + locomoção bípede leva a terópode, e a presença de apêndice dorsal ativa a regra específica do Spinosaurus.

---

### ✅ Caso 2 — Parasaurolophus

![Parasaurolophus](https://upload.wikimedia.org/wikipedia/commons/3/3b/Parasaurolophus_walkeri.png)

```python
engine.declare(Dino(
    dentes='achatados',
    bipede=True,
    crista=True
))
```

**Resultado esperado:**

```
R2: herbívoro
R3: bípede
R7: herbívoro e bípede => Ornitopode
R13: ornitópode e crista => Parasaurolophus
```

**Motivo:**
Ornitópodes com crista são classificados como Parasaurolophus, e os NOTs evitam conflito com outras regras.

---

### ❌ Caso 3 — Entrada Inválida

![Invalid](https://cdn.drawception.com/drawings/110846/pLSG3336.png)

```python
engine.declare(Dino(
    dentes='afiados',
    quadrupede=True
))
```

**Resultado esperado:**

```
R1: carnívoro
R4: quadrúpede
R6: ERRO - Não existiram dinossauros carnívoros quadrúpedes.
```

**Motivo:**
O sistema define que não existem dinossauros carnívoros quadrúpedes, então a regra R6 detecta a inconsistência.
