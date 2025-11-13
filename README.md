# PowerPlatform

### 🚀 Kanban Dinâmico no Power BI (Custom Visual HTML/DAX)

Este projeto demonstra como criar um visual Kanban altamente customizável e dinâmico diretamente no Power BI, utilizando medidas DAX para gerar código HTML, CSS e JS (na forma de HTML). Essa técnica permite transformar uma simples tabela de tarefas em um painel de gestão de fluxo de trabalho (Workflow), com cores e métricas em tempo real.

O uso de DAX/HTML permite uma flexibilidade de design que os visuais nativos (como Tabela e Matriz) não oferecem, sendo ideal para representar cards de tarefas com múltiplas informações (Título, Responsável, Prioridade, Progresso).

### 📄 Estrutura do Projeto
O visual é construído em duas medidas DAX principais:

Kanban_Tasks_Header_HTML: Responsável pela estrutura de colunas do Kanban e pelos cabeçalhos (que mostram o Status e a Contagem de Tarefas).

Kanban_Tasks_Cards_HTML: Responsável por gerar o conteúdo rolável da coluna, inserindo os cartões de tarefas com seus detalhes e formatação condicional.

Ambas as medidas devem ser colocadas em um visual de Cartão (Card), Texto Explicativo (TextBox) ou Visual Personalizado HTML/CSS (como o Visual HTML Content) no Power BI.

### 🔒 Anonimização e LGPD
Para respeitar a privacidade dos dados, os códigos abaixo foram anonimizados. Qualquer referência a nomes de tabelas, colunas ou dados sensíveis foi substituída por nomes genéricos, garantindo que a estrutura do código seja funcional, mas não revele a arquitetura real do seu modelo de dados.

## Código do Cabeçalho (Header)

Esta medida cria a estrutura de grid horizontal e os cabeçalhos com fundo amarelo e badges de contagem.

```
Kanban_Tasks_Header_HTML = 
VAR pColMinPX = 260
VAR pGapPX    = 12
VAR pGutterPX = 12 

/* Status dinâmicos do contexto atual */
VAR StatusSet =
    FILTER(
        DISTINCT( ALLSELECTED( '[Tabela_Tarefas]'[Status] ) ); // Tabela anonimizada
        NOT ISBLANK( '[Tabela_Tarefas]'[Status] ) && '[Tabela_Tarefas]'[Status] <> ""
    )

/* Ordenação por regra de negócio */
VAR OrdemStatus =
    ADDCOLUMNS(
        StatusSet;
        "_ordem";
            SWITCH(
                TRUE();
                '[Tabela_Tarefas]'[Status] = "Iniciada";     1;
                '[Tabela_Tarefas]'[Status] = "Em Revisão";   2;
                '[Tabela_Tarefas]'[Status] = "Em Pausa";     3;
                '[Tabela_Tarefas]'[Status] = "Bloqueada";    4;
                '[Tabela_Tarefas]'[Status] = "Concluída";    5;
                '[Tabela_Tarefas]'[Status] = "Finalizada";   6;
                999
            )
    )

VAR vCols = COUNTROWS( OrdemStatus )

VAR CSS =
"
<style>
/* Variáveis de layout do Kanban */
:root{
  --col-count:" & vCols & ";
  --col-min:" & pColMinPX & "px;
  --gap:" & pGapPX & "px;
  --sbw:" & pGutterPX & "px;
}
/* Grade do cabeçalho */
.k-grid{
  display:grid;
  grid-template-columns: repeat(var(--col-count), minmax(var(--col-min), 1fr));
  column-gap: var(--gap);
}
/* Estilo da célula do cabeçalho */
.k-hcell{
  box-sizing:border-box;
  background:#E8CB1B; /* Amarelo Forte */
  color:#0b1324;
  font:600 16px 'Segoe UI', system-ui, sans-serif;
  padding:10px 12px;
  border-radius:8px;
  display:flex; align-items:center; justify-content:space-between;
  min-width: 0;
}
.k-badge{
  display:inline-block; min-width:22px; padding:2px 8px; 
  background:#334155; /* Badge Escuro */
  color:#E2E8F0; border-radius:999px; font-weight:700;
}
</style>
"

VAR Cells_Header =
    CONCATENATEX(
        OrdemStatus;
        VAR s = '[Tabela_Tarefas]'[Status]
        VAR quantidade =
            CALCULATE(
                COUNTROWS( '[Tabela_Tarefas]' ); // Tabela anonimizada
                ALLSELECTED();
                KEEPFILTERS( '[Tabela_Tarefas]'[Status] = s )
            )
        RETURN
            "<div class=""k-hcell"">
               <span>" & s & "</span>
               <span class=""k-badge"">" & FORMAT( quantidade; "#,0" ) & "</span>
             </div>";
        "";
        [_ordem]; ASC
    )


RETURN
    CSS &
    "<div class=""k-grid k-header"">" & Cells_Header & "</div>"
```
## Código dos Cartões (Cards)

```
Kanban_Tasks_Cards_HTML = 
VAR pColMinPX    = 260
VAR pGapPX       = 12
VAR pGutterPX    = 20
VAR pAlturaCards = 620

/* Status dinâmicos do contexto atual (Anonimizado) */
VAR StatusSet =
    FILTER(
        DISTINCT( ALLSELECTED( '[Tabela_Tarefas]'[Status] ) );
        NOT ISBLANK( '[Tabela_Tarefas]'[Status] ) && '[Tabela_Tarefas]'[Status] <> ""
    )

/* Ordenação por regra (Anonimizado) */
VAR OrdemStatus =
    ADDCOLUMNS(
        StatusSet;
        "_ordem";
            SWITCH(
                TRUE();
                '[Tabela_Tarefas]'[Status] = "Iniciada";     1;
                '[Tabela_Tarefas]'[Status] = "Em Revisão";   2;
                '[Tabela_Tarefas]'[Status] = "Em Pausa";     3;
                '[Tabela_Tarefas]'[Status] = "Bloqueada";    4;
                '[Tabela_Tarefas]'[Status] = "Concluída";    5;
                '[Tabela_Tarefas]'[Status] = "Finalizada";   6;
                999
            )
    )

VAR vCols = COUNTROWS( OrdemStatus )

/* ===================== CSS ====================== */
VAR CSS =
"
<style>
/* ... (Restante do CSS de layout e cores) ... */

/* Colunas do Kanban */
.k-cols{
  display: grid;
  grid-template-columns: repeat(var(--col-count), minmax(var(--col-min), 1fr));
  gap: var(--gap);
  align-items: start;
}
.k-col{
  display: flex;
  flex-direction: column;
  gap: var(--gap);
}

/* Cartões */
.k-card{
  box-sizing: border-box;
  border: 1px solid #1f2937;
  border-radius: 10px;
  background: #1f2937; /* Fundo Escuro para Dark Mode (ajuste se necessário) */
  padding: 10px;
  box-shadow: 0 1px 2px rgba(2,6,23,.06);
  font: 14px 'Segoe UI', system-ui, sans-serif;
  position: relative;
  /* Faixa de prioridade na lateral (variável --prio) */
  border-left: 6px solid var(--prio, #0362A2);
}
.k-title{ font-weight: 600; margin-bottom: 6px; color: #E5E7EB; }
.k-meta{  color: #E5E7EB; margin: 2px 0; }
.k-strong{ font-weight: 600; color: #9ca3af; }

/* Barra de progresso */
.k-bar{
  height: 8px; border-radius: 999px; background: #E5E7EB; overflow: hidden; margin-top: 6px;
}
.k-bar > i{
  display: block; height: 100%;
  background: var(--bar, #16A34A); /* Cor da barra (variável --bar) */
}
</style>
"

/* ===================== HTML ===================== */
VAR ColunasHTML =
    CONCATENATEX(
        OrdemStatus;
        VAR s = '[Tabela_Tarefas]'[Status]

        /* Cartões do status 's' */
        VAR Cards_This_Status =
            CONCATENATEX(
                FILTER(
                    ALLSELECTED( '[Tabela_Tarefas]' );
                    '[Tabela_Tarefas]'[Status] = s
                );
                // Colunas anonimizadas
                VAR titulo = COALESCE( '[Tabela_Tarefas]'[Titulo]; "Vazio" )
                VAR resp   = COALESCE( '[Tabela_Tarefas]'[Pessoa_Responsavel]; "Vazio" )
                VAR prazo  = '[Tabela_Tarefas]'[Alterado].[Date]
                VAR prio   = COALESCE( '[Tabela_Tarefas]'[Prioridade]; "Média" )
                VAR pct    = COALESCE( '[Tabela_Tarefas]'[Percentual_Progresso]; 0 )

                /* Cores por prioridade (faixa lateral) */
                VAR corPrio =
                    SWITCH(
                        TRUE();
                        prio = "Alta";   "#E53935";  // Vermelho
                        prio = "Média";  "#FB8C00";  // Laranja
                        prio = "Baixa";  "#43A047";  // Verde
                        "#9E9E9E"
                    )

                /* Cor da barra de progresso */
                VAR corBar =
                    SWITCH(
                        TRUE();
                        pct >= 70; "#43A047"; // Verde (quase concluído)
                        pct >= 30; "#FB8C00"; // Laranja (em andamento)
                        "#E53935"            // Vermelho (baixo progresso)
                    )

                RETURN
                    "<div class=""k-card"" style=""--prio:" & corPrio & ";"">
                       <div class=""k-title"">" & titulo & "</div>
                       <div class=""k-meta""><span class=""k-strong"">Responsável:</span> " & resp & "</div>
                       <div class=""k-meta""><span class=""k-strong"">Prazo:</span> " &
                           IF( ISBLANK( prazo ); "—"; FORMAT( prazo; "dd/mm/yyyy" ) ) & "</div>
                       <div class=""k-meta""><span class=""k-strong"">Prioridade:</span> " & prio & "</div>
                       <div class=""k-meta""><span class=""k-strong"">Progresso:</span> " & FORMAT( DIVIDE( pct; 100 ); "0%" ) & "</div>
                       <div class=""k-bar""><i style=""width:" & FORMAT( pct; "0" ) & "%; --bar:" & corBar & ";""></i></div>
                     </div>";
                "";
                // Ordenação dos cartões dentro da coluna (exemplo: por data de alteração)
                '[Tabela_Tarefas]'[Alterado].[Date]; ASC
            )

        RETURN
            "<div class=""k-col"">" & Cards_This_Status & "</div>";
        "";
        [_ordem]; ASC
    )

RETURN
    CSS &
    "<div class=""k-wrap"">
       <div class=""k-scroll"">
         <div class=""k-cols"">" & ColunasHTML & "</div>
       </div>
     </div>"
```

### ⚠️ Direitos Autorais e Licença

*Copyright (c) Flávio Cantanhede Alves 2025. Todos os Direitos Reservados (All Rights Reserved).*
*Este código-fonte é disponibilizado publicamente apenas para fins de visualização e aprendizado. Não é permitida a cópia, distribuição, modificação ou uso comercial do código sem a autorização expressa do autor. A ausência de um arquivo de licença de código aberto (OSI) indica que este projeto está sob "Todos os Direitos Reservados" (All Rights Reserved) pela lei de direitos autorais padrão.*

