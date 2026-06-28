# 🦖 Fossil Looker

## Introdução

Fósseis de dinossauros podem ser diferenciados de acordo com características visuais identificadas nos próprios fósseis. Seguindo uma hierarquia taxonômica, o sistema de classificação começa coletando informações mais superficiais — como morfologia dental e modo de locomoção — e vai afunilando em características mais específicas, como tipo de apêndice e tamanho estimado do espécime.

A versão **Mini-Projeto 2** abandona as regras IF–THEN nítidas (*crisp*) do MP1 e adota **inferência fuzzy Mamdani** via *scikit-fuzzy*. Em vez de respostas binárias (`afiados` ou `achatados`), o usuário agora fornece **graus** para cada característica em uma escala de 0 a 10. O sistema tolera incerteza, o que faz muito mais sentido quando se está olhando para um fóssil de 66 milhões de anos.

O elenco também cresceu: saímos de **8 dinossauros** para **15**. Bem-vindo ao Jurássico aumentado.

A inferência ainda acontece em 3 níveis conceituais:

1. Características visuais (graus de pertinência nas variáveis de entrada)
2. Agrupamento taxonômico (terópodes, ornitópodes, outros herbívoros)
3. Classificação de espécie (defuzzificação pelo centroide)

---

## Arquitetura do Sistema

O Fossil Looker Fuzzy é um controlador **Mamdani** com **4 entradas** e **1 saída**.

### Variáveis de Entrada

| Variável | Escala | Termos Linguísticos |
|---|---|---|
| `dentes` | 0–10 | *afiados*, *mistos*, *achatados* |
| `locomocao` | 0–10 | *bipede*, *misto*, *quadrupede* |
| `appendice` | 0–10 | *chifres*, *crista*, *dorsal* |
| `tamanho` | 0–10 | *pequeno*, *medio*, *grande* |

> 0 = extremo esquerdo do termo, 10 = extremo direito. Os valores intermediários ativam múltiplos termos simultaneamente — esse é justamente o ponto.

### Variável de Saída

| Variável | Escala | Termos |
|---|---|---|
| `especie` | 0–14 | Um índice por dinossauro (15 no total) |

Cada dinossauro ocupa uma "fatia" do universo de saída com uma gaussiana. A defuzzificação pelo método **centroide** devolve o índice do espécime mais provável dado o conjunto de entradas.

---

## Regras em Linguagem Natural

### 🔹 Nível 1 — Características visuais

* Regra de dentes: quanto mais próximo de 0, mais afiados (carnívoro); quanto mais próximo de 10, mais achatados (herbívoro); valores intermediários indicam dentição mista
* Regra de locomoção: 0 = bípede puro; 10 = quadrúpede puro; valores em torno de 5 capturam espécimes facultativos como o Iguanodon
* Regra de apêndice: 0–3 = chifres; 3–7 = crista craniana; 7–10 = apêndice dorsal (velas, placas)
* Regra de tamanho: 0–3 = pequeno (<3m); 3–7 = médio (3–8m); 7–10 = grande (>8m)

---

### 🔹 Nível 2 — Agrupamento taxonômico

* Dentes afiados + bípede → **Terópode**
* Dentes achatados + bípede (ou misto) → **Ornitópode**
* Dentes achatados + quadrúpede → **Outro Herbívoro**
* Combinações com alta ambiguidade → o sistema não trava: ele simplesmente não dispara nenhuma regra com força suficiente, retornando um centroide sem classificação confiante

---

### 🔹 Nível 3 — Classificação de espécie

#### Terópodes (carnívoros bípedes)

* **R1:** Afiados + Bípede + Chifres + Pequeno → Velociraptor
* **R2:** Afiados + Bípede + Chifres + Grande → T-Rex
* **R3:** Afiados + Bípede + Dorsal + Grande → Spinosaurus
* **R4:** Afiados + Bípede + Chifres + Médio → Ceratosaurus
* **R5:** Afiados + Bípede + Crista + Pequeno → Cryolophosaurus
* **R6:** Afiados + Bípede + Crista + Médio → Allosaurus
* **R7:** Afiados + Bípede + Dorsal + Médio → Carnotaurus

#### Ornitópodes (herbívoros bípedes ou facultativos)

* **R8:** Achatados + Bípede + Crista + Grande → Parasaurolophus
* **R9:** Achatados + Misto + Chifres + Médio → Iguanodon
* **R10:** Achatados + Bípede + Dorsal + Médio → Ouranosaurus
* **R11:** Achatados + Misto + Crista + Grande → Corythosaurus

#### Outros Herbívoros (quadrúpedes)

* **R12:** Achatados + Quadrúpede + Chifres + Grande → Triceratops
* **R13:** Achatados + Quadrúpede + Dorsal + Médio → Stegosaurus
* **R14:** Achatados + Quadrúpede + Dorsal + Grande → Ankylosaurus
* **R15:** Achatados + Quadrúpede + Crista + Grande → Brachiosaurus

> Diferente do MP1, não há "regras de erro". Combinações biologicamente improváveis simplesmente produzem uma classificação de baixa confiança — o sistema degrada com graciosidade.

---

## Roster Completo

Referência rápida de como cada espécime se mapeia nas escalas de entrada.

| Dinossauro | Dentes | Locomoção | Apêndice | Tamanho | Grupo |
|---|---|---|---|---|---|
| Velociraptor | 1.0 | 1.0 | 1.0 | 1.0 | Terópode |
| T-Rex | 1.0 | 1.0 | 1.0 | 9.0 | Terópode |
| Spinosaurus | 1.0 | 1.0 | 9.0 | 9.5 | Terópode |
| Ceratosaurus | 1.5 | 1.0 | 1.5 | 5.0 | Terópode |
| Cryolophosaurus | 1.0 | 1.0 | 5.0 | 2.0 | Terópode |
| Allosaurus | 1.0 | 1.0 | 5.0 | 6.0 | Terópode |
| Carnotaurus | 1.0 | 1.0 | 8.0 | 5.0 | Terópode |
| Parasaurolophus | 9.5 | 1.0 | 5.0 | 8.5 | Ornitópode |
| Iguanodon | 8.0 | 5.0 | 1.5 | 5.5 | Ornitópode |
| Ouranosaurus | 9.0 | 1.0 | 9.0 | 5.0 | Ornitópode |
| Corythosaurus | 9.0 | 4.0 | 5.0 | 8.0 | Ornitópode |
| Triceratops | 8.5 | 9.0 | 1.0 | 8.0 | Outro Herbívoro |
| Stegosaurus | 9.0 | 9.0 | 9.0 | 5.0 | Outro Herbívoro |
| Ankylosaurus | 9.0 | 9.0 | 9.0 | 8.0 | Outro Herbívoro |
| Brachiosaurus | 9.0 | 9.0 | 5.0 | 9.5 | Outro Herbívoro |

---

## Casos de Teste

### ✅ Caso 1 — Spinosaurus

![Spinosaurus](https://upload.wikimedia.org/wikipedia/commons/thumb/e/e4/Spinosaurus_aegyptiacus_life_reconstruction.png/1920px-Spinosaurus_aegyptiacus_life_reconstruction.png)

```python
identificar(
    dentes_val=1.0,
    locomocao_val=1.0,
    appendice_val=9.0,
    tamanho_val=9.5
)
```

**Resultado esperado:**

```
Dentes    : 1.0  (afiados)
Locomoção : 1.0  (bípede)
Apêndice  : 9.0  (dorsal)
Tamanho   : 9.5  (grande)
Saída defuzzificada: ~2.0
🦕 DINOSSAURO IDENTIFICADO: SPINOSAURUS
```

**Motivo:**
Dentes cônicos afiados e locomoção bípede levam ao grupo terópode. A vela dorsal proeminente (apêndice=9.0) combinada com o porte gigantesco (tamanho=9.5) ativa a R3 com força máxima, puxando o centroide para o índice do Spinosaurus.

---

### ✅ Caso 2 — Parasaurolophus

![Parasaurolophus](https://upload.wikimedia.org/wikipedia/commons/3/3b/Parasaurolophus_walkeri.png)

```python
identificar(
    dentes_val=9.5,
    locomocao_val=1.0,
    appendice_val=5.0,
    tamanho_val=8.5
)
```

**Resultado esperado:**

```
Dentes    : 9.5  (achatados)
Locomoção : 1.0  (bípede)
Apêndice  : 5.0  (crista)
Tamanho   : 8.5  (grande)
Saída defuzzificada: ~7.0
🦕 DINOSSAURO IDENTIFICADO: PARASAUROLOPHUS
```

**Motivo:**
Dentição altamente achatada + bipedalismo leva ao grupo ornitópode. A crista craniana tubular (apêndice=5.0) e o porte grande (tamanho=8.5) ativam a R8 com força dominante, apontando direto para o Parasaurolophus.

---

### ✅ Caso 3 — Triceratops

![Triceratops](https://cdn.sanity.io/images/wxr9jaaa/production/9fb5058673843f49acb43553180389bfb6b4817e-1376x768.png)

```python
identificar(
    dentes_val=8.5,
    locomocao_val=9.0,
    appendice_val=1.0,
    tamanho_val=8.0
)
```

**Resultado esperado:**

```
Dentes    : 8.5  (achatados)
Locomoção : 9.0  (quadrúpede)
Apêndice  : 1.0  (chifres)
Tamanho   : 8.0  (grande)
Saída defuzzificada: ~11.0
🦕 DINOSSAURO IDENTIFICADO: TRICERATOPS
```

**Motivo:**
Herbívoro quadrúpede robusto com três chifres proeminentes e porte grande. A R12 dispara com força máxima, sem competição de outras regras do grupo.

---

### 🔬 Caso Bônus — Iguanodon

![Iguanodon](https://www.dododex.com/media/creature/iguanodon.png)

O Iguanodon é o dinossauro que mais quebrava o modelo do MP1: ele era facultativamente bípede, o que não se encaixava em nenhuma categoria crisp. Aqui, `locomocao=5.0` captura isso sem exigir uma regra especial.

```python
identificar(
    dentes_val=8.0,
    locomocao_val=5.0,
    appendice_val=1.5,
    tamanho_val=5.5
)
```

**Resultado esperado:**

```
Dentes    : 8.0  (achatados)
Locomoção : 5.0  (misto)
Apêndice  : 1.5  (chifres)
Tamanho   : 5.5  (médio)
🦕 DINOSSAURO IDENTIFICADO: IGUANODON
```

**Motivo:**
Este é o caso que o MP1 nunca conseguiria resolver com elegância. O valor `locomocao=5.0` cai exatamente no pico do termo `misto`, ativando a R9 com pertinência plena. O fuzzy absorve a nuance biológica sem nenhuma gambiarra.

---

## Comparativo: MP1 (Regras Crisp) vs MP2 (Fuzzy Mamdani)

| Critério | MP1 (crisp) | MP2 (fuzzy Mamdani) |
|---|---|---|
| Entradas | Booleanas / enum | Valores contínuos 0–10 |
| Nº de dinossauros | 8 | 15 |
| Nº de regras | 17 (incluindo regras de erro) | 15 (sem regras de erro) |
| Ambiguidade | Sem suporte | Graus de pertinência |
| Iguanodon (facultativo) | Não modelável | `locomocao=5.0` |
| Fóssil incompleto | Trava / lança erro | Inferência parcial com baixa confiança |
| Complexidade de extensão | Alta (novas regras de conflito) | Baixa (nova MF + 1 nova regra) |
| Expressividade | Binária | Gradual / linguística |

**Pontos-chave:**

1. **Tratamento de erro:** O MP1 precisava de regras explícitas para combinações inválidas (carnívoro quadrúpede, ornitópode com chifre). O MP2 simplesmente não dispara nenhuma regra forte para esses casos — menos regras, degradação mais graciosa.

2. **Expressividade:** "Dentes levemente afiados" não existia no MP1. Com `dentes=3.5`, o sistema calcula pertinência simultânea em `afiados` (0.3) e `mistos` (0.6), refletindo a incerteza real de um fóssil fragmentado.

3. **Escalabilidade:** Adicionar um dinossauro no MP1 exigia revisar todas as regras de conflito. No MP2, basta acrescentar uma gaussiana na saída e uma nova regra de antecedente.

4. **Trade-off de auditabilidade:** O fuzzy perdeu a clareza diagnóstica do MP1. Antes, você sabia exatamente qual regra disparou. Agora, múltiplas regras disparam com pesos diferentes — o resultado é correto, mas menos auditável.
