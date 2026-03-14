# DSL `CookLang`

## Descrição Resumida da DSL

### Contextualização da linguagem
Linguagem para produzir receitas de culinária. A ideia é que cada código da linguagem gere uma receita completa com ingredientes e passos.
 
### Motivação
Vimos que a linguagem pode ser útil para padronizar e automatizar geração e leitura de receitas.

### Relevância
Compartilhamento de receitas com mais facilidade e praticidade.

## Slides

> Coloque aqui o link para o PDF da apresentação.

## Sintaxe da Linguagem na Forma de Tutorial

Cada código deve ter uma `recipe`. Dentro de cada bloco de `recipe`, há uma lista de `metadata`, uma lista de `ingredients` e uma lista de `steps`.

Dentro de `metadata`, temos `servings`, que indica quantas porções a receita produz, e `time`, que mostra o tempo de preparo da receita.

Dentro de `ingredients`, temos uma lista de ingredientes, onde cada ingrediente tem uma quantidate, uma unidade de medida e um nome.

Dentro de `steps`, temos uma lista de passos, instruções a serem seguidas para fazer a receita. Cada instrução tem um número e uma descrição.

## Gramática da Linguagem

```antlr
S -> Recipe
Recipe -> recipe String { Metadata IngredientsBlock StepsBlock }

Metadata -> servings : Number time : Number TimeUnit

IngredientsBlock -> ingredients { IngredientsList }
IngredientsList -> Ingredient IngredientsList | #
Ingredient -> Number Unit Word

StepsBlock -> steps { StepsList }
StepsList -> Step StepsList | #
Step -> Number : Instruction
Instruction -> InstructionWord InstructionTail
InstructionTail -> InstructionWord InstructionTail | #
InstructionWord -> Word | Number  // previne instruções vazias

Unit -> g | kg | ml | l | xicara | colher_cha | colher_sopa | unidade
TimeUnit -> s | min | h

String -> " WordList "
WordList -> Word WordList | Word

Number -> Digit Number | Digit
Digit -> 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9

Word -> Letter Word | Letter
Letter -> a | b | c | d | e | f | g | h | i | j | k | l | m | n | o | p | q | r | s | t | u | v | w | x | y | z | _
```

## Exemplos Selecionados

### Bolo de cenoura:
Código em CookLang
```yaml
recipe "bolo de cenoura" {
    servings: 8
    time: 50 min

    ingredients {
        3 unidade cenoura
        4 unidade ovo
        2 xicara farinha_trigo
        2 xicara acucar
        1 colher_cha fermento
        1 xicara oleo
    }

    steps {
        1: descascar cenoura
        2: cortar cenoura
        3: bater cenoura ovo oleo
        4: misturar farinha_trigo acucar
        5: adicionar fermento
        6: assar 40 min
    }
}
```

Resultado
```text
Receita: Bolo de Cenoura

Rendimento: 8 porções
Tempo de preparo: 50 minutos

Ingredientes

3 unidades de cenoura

4 unidades de ovo

2 xícaras de farinha_trigo

2 xícaras de açúcar

1 colher de chá de fermento

1 xícara de óleo

Modo de Preparo

1. Descasque as cenouras.

2. Corte as cenouras em pedaços.

3. Bata as cenouras com os ovos e o óleo.

4. Misture a farinha_trigo com o açúcar.

5. Adicione o fermento à mistura.

6. Asse a massa por aproximadamente 40 minutos.
```

### Omelete:
Código em CookLang
```yaml
recipe "omelete simples" {
    servings: 1
    time: 10 min

    ingredients {
        2 unidade ovo
        1 colher_sopa manteiga
        1 pitada sal
    }

    steps {
        1: quebrar ovo
        2: bater ovo sal
        3: aquecer manteiga
        4: fritar ovo
    }
}
```

Resultado
```text
Receita: Omelete Simples

Rendimento: 1 porção
Tempo de preparo: 10 minutos

Ingredientes

2 unidades de ovo

1 colher de sopa de manteiga

1 pitada de sal

Modo de Preparo

1. Quebre os ovos em um recipiente.

2. Bata os ovos com o sal.

3. Aqueça a manteiga em uma frigideira.

4. Despeje os ovos batidos na frigideira e frite até que a omelete esteja pronta.
```

# Referências Bibliográficas

Fowler, M. Domain Specific Languages. Addison-Wesley, 2010.

Parr, T. Language Implementation Patterns. Pragmatic Bookshelf, 2010.

Aho, A. V.; Lam, M.; Sethi, R.; Ullman, J. Compilers: Principles, Techniques, and Tools.
