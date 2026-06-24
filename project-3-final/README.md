# DSL `CookLang` — Fase 3

## Descrição Resumida da DSL

**Contextualização da linguagem.** A CookLang é uma linguagem específica de
domínio (DSL) para descrição de receitas culinárias. Cada programa na linguagem
representa uma receita completa, com metadados (porções e tempo), lista de
ingredientes e passos de preparo. Nas Fases 1 e 2 definimos a gramática e as
primitivas; nesta Fase 3 a linguagem é **implementada de fato**, em Guile
Scheme, usando **macros** para definir sua sintaxe.

**Motivação.** Receitas circulam hoje em formatos livres (blogs, redes sociais,
vídeos), sem padronização. Isso dificulta organizar, validar e reaproveitar essa
informação. A CookLang dá às receitas um formato estruturado, legível por
pessoas e por programas.

**Relevância.** Uma receita escrita em CookLang é um registro estruturado e
validado: o interpretador garante que todo ingrediente usado em um passo foi
declarado, e a mesma estrutura pode ser renderizada, escalada ou consultada.
Isso a torna adequada como base para um **banco de dados público de receitas**.

## Slides

> _Coloque aqui o link para o PDF da apresentação final._
>
> [📄 Apresentação da Fase 3 (PDF)](slides/cooklang-fase3.pdf)

## Sintaxe da Linguagem

Nesta fase a sintaxe da CookLang é expressa em **s-expressões** e definida por
macros Scheme. Uma receita é escrita com a macro `receita`, que recebe quatro
blocos na ordem fixa: título, `porcoes`, `tempo`, `ingredientes` e `passos`.

### Estrutura geral

```scheme
(receita "Título da Receita"
  (porcoes N)
  (tempo N unidade-de-tempo)
  (ingredientes
    (QTD UNIDADE NOME)
    ...)
  (passos
    (VERBO ARG ...)
    ...))
```

### Blocos

| Bloco          | Forma                              | Exemplo                       |
|----------------|------------------------------------|-------------------------------|
| Título         | string literal                     | `"Bolo de Cenoura"`           |
| Porções        | `(porcoes N)`                      | `(porcoes 8)`                 |
| Tempo          | `(tempo N unidade)`                | `(tempo 50 min)`              |
| Ingrediente    | `(QTD UNIDADE NOME)`               | `(2 xicara farinha_trigo)`    |
| Passo          | `(VERBO ARG ...)`                  | `(bater cenoura ovo oleo)`    |

* **Unidades de medida:** `g`, `kg`, `ml`, `l`, `xicara`, `colher_cha`,
  `colher_sopa`, `unidade`, `pitada`, `dente`.
* **Unidades de tempo:** `s`, `min`, `h`.
* Em um passo, todo símbolo que **não** seja unidade de tempo é tratado como
  referência a ingrediente e **deve** ter sido declarado no bloco
  `ingredientes`. Números (ex.: o `40` em `(assar 40 min)`) são ignorados pela
  validação.

### Operações sobre receitas

Uma vez construída, a receita pode ser processada:

```scheme
(servir bolo)            ; renderiza a receita como texto formatado
(escalar bolo 2)         ; devolve uma nova receita com o dobro das porções
(selecionar de bolo onde (eq? unidade 'xicara))  ; consulta os ingredientes
```

A macro de consulta `selecionar ... de ... onde ...` é inspirada no
`select ... from ... where` do material da disciplina e permite filtrar
ingredientes usando diretamente os campos `qtd`, `unidade` e `nome`.

## Gramática da Linguagem

A gramática abaixo (notação BNF/EBNF) descreve a **sintaxe concreta de
s-expressões** aceita pelas macros desta fase. Ela é equivalente à gramática das
fases anteriores, reescrita na forma parentizada do Scheme.

```bnf
Receita      ::= "(" "receita" String Porcoes Tempo Ingredientes Passos ")"

Porcoes      ::= "(" "porcoes" Number ")"
Tempo        ::= "(" "tempo" Number UnidadeTempo ")"

Ingredientes ::= "(" "ingredientes" Ingrediente* ")"
Ingrediente  ::= "(" Number Unidade Nome ")"

Passos       ::= "(" "passos" Passo+ ")"
Passo        ::= "(" Verbo Token* ")"
Token        ::= Nome | Number

Unidade      ::= "g" | "kg" | "ml" | "l" | "xicara" | "colher_cha"
               | "colher_sopa" | "unidade" | "pitada" | "dente"
UnidadeTempo ::= "s" | "min" | "h"

Verbo        ::= Symbol
Nome         ::= Symbol
String       ::= '"' Char* '"'
Number       ::= Digit+
Symbol       ::= (Letra | "_") (Letra | Digit | "_" | "-")*
```

> Formalismo: BNF/EBNF. A verificação sintática efetiva é feita pelo próprio
> mecanismo de `syntax-rules` do Guile (os padrões da macro `receita`); a
> verificação semântica (ingrediente declarado) é feita em tempo de construção.

## Notebook

A implementação completa e executável está no notebook:

* [notebook (`cooklang.ipynb`)](cooklang.ipynb) — implementação em Guile Scheme.

O notebook é gerado a partir do script auxiliar
[`build_nb.py`](build_nb.py) (apenas conveniência para montar o `.ipynb`; não é
parte da linguagem).

### Como executar

O notebook usa um **kernel Guile** para Jupyter. Com o Guile instalado:

```bash
# instalar o kernel Guile para Jupyter (uma vez)
pip install jupyter
# (kernel Guile, ex.: guile-jupyter ou similar)

jupyter notebook cooklang.ipynb
```

Em seguida, execute as células de cima para baixo. A célula da Seção 8 **deve
falhar propositalmente** — ela demonstra a validação semântica.

## Exemplos Selecionados

Os exemplos completos estão no notebook. Resumo dos principais:

### Bolo de Cenoura

```scheme
(define bolo
  (receita "Bolo de Cenoura"
    (porcoes 8)
    (tempo 50 min)
    (ingredientes
      (3 unidade     cenoura)
      (4 unidade     ovo)
      (2 xicara      farinha_trigo)
      (2 xicara      acucar)
      (1 colher_cha  fermento)
      (1 xicara      oleo))
    (passos
      (descascar cenoura)
      (cortar    cenoura)
      (bater     cenoura ovo oleo)
      (misturar  farinha_trigo acucar)
      (adicionar fermento)
      (assar     40 min))))

(servir bolo)
```

Saída (gerada por `servir`):

```text
Receita: Bolo de Cenoura

Rendimento: 8 porcao(es)
Tempo de preparo: 50 min

Ingredientes:
  - 3 unidade de cenoura
  - 4 unidade de ovo
  - 2 xicara de farinha_trigo
  - 2 xicara de acucar
  - 1 colher_cha de fermento
  - 1 xicara de oleo

Modo de Preparo:
  1. descascar cenoura
  2. cortar cenoura
  3. bater cenoura ovo oleo
  4. misturar farinha_trigo acucar
  5. adicionar fermento
  6. assar 40 min
```

### Validação semântica (erro proposital)

A receita abaixo usa `farinha_trigo` e `acucar` em um passo sem declará-los:

```scheme
(receita "Bolo com Erro"
  (porcoes 8)
  (tempo 50 min)
  (ingredientes (3 unidade cenoura) (4 unidade ovo) (1 xicara oleo))
  (passos
    (bater    cenoura ovo oleo)
    (misturar farinha_trigo acucar)))
```

Resultado:

```text
ERRO SEMANTICO na receita "Bolo com Erro": ingrediente 'farinha_trigo'
usado no passo 'misturar' nao foi declarado nos ingredientes.
```

### Consulta (DSL tipo banco de dados)

```scheme
(selecionar de bolo onde (eq? unidade 'xicara))
;; => ((2 xicara farinha_trigo) (2 xicara acucar) (1 xicara oleo))

(selecionar de bolo onde (> qtd 1))
;; => ((3 unidade cenoura) (4 unidade ovo) (2 xicara farinha_trigo) (2 xicara acucar))
```

### Escalar porções

```scheme
(servir (escalar bolo 2))   ;; mesma receita, 16 porções, quantidades dobradas
```

## Discussão

A proposta inicial (Fases 1 e 2) era padronizar receitas em um formato
estruturado, validável e processável. Na Fase 3 isso se concretizou em código
executável:

* **Sintaxe via macros.** A macro `receita` (`syntax-rules` com literais
  `porcoes`/`tempo`/`ingredientes`/`passos` e elipses `...`) transforma a receita
  escrita em s-expressões na estrutura interna, em tempo de expansão. Os nomes de
  ingredientes e unidades são automaticamente quotados, enquanto as quantidades
  permanecem numéricas — exatamente o comportamento desejado, obtido sem código
  de parsing manual.
* **Validação semântica.** A regra "todo ingrediente usado foi declarado",
  central desde a Fase 2, ficou expressa de forma direta e roda na construção da
  receita. O exemplo de erro reproduz fielmente o comportamento da Fase 2.
* **Receita como dado.** Representar a receita como uma *closure* de despacho
  (padrão `create-table` do material da disciplina) permitiu adicionar operações
  novas — `servir`, `escalar`, `selecionar` — sem alterar a definição da
  linguagem.
* **Conexão com a proposta.** A macro de consulta `selecionar` mostra
  concretamente como receitas em CookLang podem alimentar um banco de dados:
  cada receita é um registro consultável.

O resultado foi bom porque o casamento de padrões de `syntax-rules` é uma forma
natural de descrever a estrutura de uma linguagem: o que nas fases anteriores era
gramática em papel virou, quase literalmente, o padrão da macro. A principal
limitação observada está na **higiene** das macros: a consulta `selecionar`
precisou recorrer a `eval` sobre uma `lambda` construída dinamicamente para que a
condição enxergasse os campos `qtd`/`unidade`/`nome` — a mesma técnica usada na
macro `selection` de referência. Isso indica uma direção de estudo: comparar
macros higiênicas (`syntax-rules`) com não higiênicas (`define-macro`) e seus
custos.

## Conclusão

* **Principais conclusões.** É possível implementar uma DSL completa para
  receitas usando apenas macros e funções de Scheme. A sintaxe da linguagem, sua
  validação e suas operações de processamento foram obtidas de forma concisa,
  fechando o ciclo iniciado nas fases anteriores.
* **Principais desafios.** Lidar com a higiene das macros na operação de
  consulta; garantir que a validação distinguisse corretamente referências a
  ingredientes de quantidades de tempo dentro de um passo; e ordenar as
  definições para que os construtores existissem quando as macros fossem usadas.
* **Lições aprendidas.** Macros não são apenas açúcar sintático: elas são uma
  ferramenta de projeto de linguagem. Definir a gramática como padrões de
  `syntax-rules` aproximou muito a especificação da implementação, e o padrão de
  *closure* de despacho manteve a linguagem extensível.

# Trabalhos Futuros

* **Mais validações:** checar tempo total dos passos contra o tempo declarado;
  detectar ingredientes declarados e nunca usados.
* **Recipientes e estado:** reintroduzir as primitivas de ação da Fase 2
  (`misturar`, `assar` sobre recipientes) como parte executável da linguagem.
* **Exportação:** gerar saída em JSON/Markdown a partir da estrutura interna,
  para alimentar diretamente um banco de dados ou um site de receitas.
* **Consultas entre receitas:** estender `selecionar` para operar sobre uma
  coleção de receitas (uma tabela de receitas), aproximando-se de um SGBD de
  domínio culinário.
* **Internacionalização:** permitir verbos e unidades em outros idiomas.

# Referências Bibliográficas

* Fowler, M. *Domain Specific Languages*. Addison-Wesley, 2010.
* Parr, T. *Language Implementation Patterns*. Pragmatic Bookshelf, 2010.
* Aho, A. V.; Lam, M.; Sethi, R.; Ullman, J. *Compilers: Principles, Techniques,
  and Tools*. Addison-Wesley.
* Abelson, H.; Sussman, G. J. *Structure and Interpretation of Computer
  Programs*. MIT Press, 1996.
* *GNU Guile Reference Manual* — `syntax-rules`, `define-syntax`, macros.
  <https://www.gnu.org/software/guile/manual/>
