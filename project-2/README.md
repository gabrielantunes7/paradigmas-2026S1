# DSL `CookLang`

## Primitivas da Linguagem

### Estruturas de Dados Subjacentes

Para a construção de primitivas, precisamos de estruturas de dados subjacentes que servem de apoio para a linguagem.

Isso se dá a partir de primitivas usadas internamente para obter listas personalizadas com os tipos de dados que precisamos.

```scheme
;; 1. Construtor de Ingredientes (Baseado na gramática: Number Unit Word)
(define (cria-ingrediente qtd unidade nome)
  (list 'ingrediente qtd unidade nome))

;; 2. Construtor de Recipiente (ex: panela, liquidificador, forma)
(define (cria-recipiente nome conteudo-inicial)
  (list 'recipiente nome conteudo-inicial))

;; Seletores auxiliares para facilitar a leitura do código
(define (tipo-de obj) (car obj))
(define (nome-recip obj) (cadr obj))
(define (conteudo-recip obj) (caddr obj))
(define (nome-ingrediente obj) (cadddr obj))
```

### Primitivas Declarativas (setup do ambiente)

A linguagem precisa de primitivas declarativas, que servem como setup do ambiente.

Aqui, teremos uma Despensa (onde ficam todos os ingredientes da receita) e, a partir dos metadados da receita, o sistema pode fazer validações semânticas.

```scheme
;; Primitiva que declara as restrições da receita
(define (declarar-metadata porcoes tempo-maximo)
  (list 'metadata porcoes tempo-maximo))

;; Primitiva que cria o "Estado Zero" do mundo (A Despensa).
;; Recebe uma lista de ingredientes e guarda na memória para validação futura.
(define (declarar-despensa . lista-ingredientes)
  (list 'despensa lista-ingredientes))

;; Função auxiliar do interpretador (Linter): 
;; Busca um ingrediente na despensa. Se não existir, acusa erro (Validação Semântica).
(define (pegar-da-despensa nome ambiente-despensa)
  (let loop ((itens (cadr ambiente-despensa)))
    (cond 
      ;; Se a lista acabou e não achou, quebra a compilação da receita
      ((null? itens) (error "ERRO DE COMPILAÇÃO: Ingrediente não declarado na receita!" nome))
      ;; Se achou o nome, retorna o ingrediente
      ((eq? (nome-ingrediente (car itens)) nome) (car itens))
      ;; Senão, continua buscando
      (else (loop (cdr itens))))))
```

### Primitivas de Ação (verbos da linguagem)

A linguagem também conta com primitivas de ação, ou seja, verbos.

Essas primitivas servirão para o interpretador realizar uma execução para checagem de corretude da receita e dos seus passos, para que façam sentido.

```scheme
;; Primitiva MISTURAR (Agregação)
;; Recebe um recipiente alvo e N elementos. Retorna um NOVO recipiente com tudo dentro.
(define (misturar recipiente . elementos)
  (if (not (eq? (tipo-de recipiente) 'recipiente))
      (error "O alvo da mistura deve ser um recipiente!")
      (let ((nome (nome-recip recipiente))
            (conteudo-atual (conteudo-recip recipiente)))
        
        ;; Extrai o conteúdo: se for ingrediente, guarda; se for recipiente, despeja o que tem dentro
        (define (extrair el)
          (cond ((eq? (tipo-de el) 'ingrediente) (list el))
                ((eq? (tipo-de el) 'recipiente)  (conteudo-recip el))
                (else '())))
        
        ;; Retorna o novo estado (Programação Funcional: sem mutação destrutiva)
        (cria-recipiente 
         nome 
         (append conteudo-atual (apply append (map extrair elementos)))))))

;; Primitiva ASSAR (Transformação com restrição de tempo)
(define (assar recipiente tempo-minutos contexto-metadata)
  ;; O interpretador poderia usar essa primitiva para somar o tempo total da receita
  ;; e checar se bate com o tempo declarado no cabeçalho!
  (list 'recipiente-assado (nome-recip recipiente) (conteudo-recip recipiente) tempo-minutos))
```

### Exemplos de uso (avaliando uma receita de bolo)

```scheme
;; 1. O Parser leu o cabeçalho e inicializou o contexto:
(define contexto-bolo (declarar-metadata 8 50))

;; 2. O Parser leu o bloco "ingredients" e gerou a despensa:
(define despensa-receita
  (declarar-despensa
   (cria-ingrediente 3 'unidade 'cenoura)
   (cria-ingrediente 4 'unidade 'ovo)
   (cria-ingrediente 1 'xicara 'oleo)
   (cria-ingrediente 2 'xicara 'farinha_trigo)))

;; Criando os recipientes limpos
(define liquidificador (cria-recipiente 'liquidificador_master '()))
(define tigela (cria-recipiente 'tigela_grande '()))

;; 3. Executando Passo: "bater cenoura ovo oleo"
;; Note que o interpretador exige pegar os itens da despensa primeiro (validação)
(define etapa-1-liquidos
  (misturar liquidificador
            (pegar-da-despensa 'cenoura despensa-receita)
            (pegar-da-despensa 'ovo despensa-receita)
            (pegar-da-despensa 'oleo despensa-receita)))

;; 4. Executando Passo: "misturar farinha_trigo acucar"
;; VAMOS FORÇAR UM ERRO! O açúcar não foi declarado na despensa ali em cima.
(define etapa-2-erro (pegar-da-despensa 'acucar despensa-receita)) 

;; 5. Executando Passo (Corrigido apenas com farinha): "misturar liquidos com secos"
(define etapa-3-massa
  (misturar tigela
            (pegar-da-despensa 'farinha_trigo despensa-receita)
            etapa-1-liquidos)) ;; Despejando o liquidificador na tigela

;; 6. Executando Passo: "assar 40 min"
(define bolo-pronto (assar etapa-3-massa 40 contexto-bolo))

(display "Estado final da receita computada:\n")
bolo-pronto
```

Com a linha de `etapa-2-erro` descomentada, o resultado da execução é:

```text
misc-error
(#f ~A ~S (ERRO DE COMPILAÇÃO: Ingrediente não declarado na receita! acucar) #f)
```

Sem essa linha que força o erro, temos:

```text
Estado final da receita computada:
(recipiente-assado tigela_grande ((ingrediente 2 xicara farinha_trigo) (ingrediente 3 unidade cenoura) (ingrediente 4 unidade ovo) (ingrediente 1 xicara oleo)) 40)
```

## Slides

A apresentação do projeto pode ser acessada aqui:

[Apresentação da Segunda Etapa (Figma)](https://www.figma.com/deck/xuG4SWiVr5eV3489TSuPEx)

# Referências Bibliográficas

Fowler, M. Domain Specific Languages. Addison-Wesley, 2010.

Parr, T. Language Implementation Patterns. Pragmatic Bookshelf, 2010.

Aho, A. V.; Lam, M.; Sethi, R.; Ullman, J. Compilers: Principles, Techniques, and Tools.
