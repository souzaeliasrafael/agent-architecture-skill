---
name: arquitetura-agentes
description: Avalia, evolui e conduz de ponta a ponta qualquer projeto (ideia, pesquisa, software, negócio), novo ou já em andamento, com frentes paralelas, fiscal isolado, juiz por rubrica, árbitro de portões e execuções agendadas. Use quando o usuário pedir 'usa a arquitetura de agentes'.
---

# Arquitetura de agentes (IEE + V6) — versão ChatGPT e Codex

Junta o fluxo do Idea Evolution Engine (estruturar, pesquisar, construir, criticar, julgar, melhorar) com o modelo de execução de um projeto de pesquisa longo (especificação escrita, frentes paralelas com trava e sinal de vida, fiscal que valida cada entrega sem ver o raciocínio de quem fez, árbitro que decide portões sem esperar o usuário e registra o motivo). Serve para qualquer projeto: o que muda de um projeto para outro fica na especificação, nunca nesta skill.

Modos de uso:
- **Avaliar** ("usa a arquitetura de agentes para avaliar X"): Fases 0 e 1 na conversa, sem agendamento.
- **Conduzir** ("usa a arquitetura de agentes e toca o projeto"): Fases 0 a 3, com execuções agendadas até o fim.
- **Assumir** (projeto já em andamento): Fase A primeiro, depois segue como Conduzir.

## Papéis e isolamento

Os papéis abaixo são executados um de cada vez. O **Fiscal** e o **Árbitro** precisam de isolamento: o ChatGPT não tem subagentes numa conversa comum, então o isolamento é feito por execução separada. O Fiscal roda numa conversa nova ou numa tarefa agendada própria, recebendo **somente** a ESPEC e a entrega (nunca o histórico de quem construiu). O Árbitro também roda numa execução separada, lendo só os arquivos de controle. No Codex, se houver subagentes disponíveis, use um subagente para cada um.

## Fase A: assumir projeto em andamento

1. Localize o que já existe: repositório, pastas, documentos, notas e execuções agendadas do projeto.
2. **Não refaça nem sobrescreva nada.** Escreva a ESPEC e o STATUS a partir do que já foi decidido, marcando `[JÁ FEITO]` o que está pronto e `[DECIDIDO PELO USUÁRIO]` cada decisão já tomada (essas valem sobre o árbitro).
3. Se já houver execuções agendadas para o projeto, ajuste-as em vez de criar duplicadas.

## Fase 0: especificação (obrigatória)

Crie `ESPEC.md` com:
1. **Objetivo** em uma frase e **tipo de projeto** (ideia de negócio, pesquisa/artigo, software, análise de dados, outro).
2. **Gabarito**: contra o que o resultado é medido (base oficial, métrica, cliente real, teste). Se não existir, escreva isso explicitamente.
3. **Critérios do Juiz**: de 6 a 12 da lista abaixo, mais os específicos do projeto, cada um com perguntas sim/não.
4. **Portões**: em que pontos o árbitro decide seguir, refazer, testar ou aprovar uma tarefa.
5. **Restrições**: orçamento, prazo, ferramentas proibidas, o que nunca pode ser alterado.
6. **Limites**: máximo de rodadas por tarefa (padrão 3, teto 5; até 3 fiscalizações) e **critério de término** do projeto inteiro.
7. **Critérios de parada**: sinais que indicariam que o projeto não vale a pena (ex.: "se em 4 semanas não houver X"). O agente propõe e o usuário aprova antes de entrarem na ESPEC. Atingir um critério de parada abre um alerta de continuidade; não encerra nada sozinho.

Se faltar informação essencial, pergunte uma vez, de forma objetiva. Sem o usuário presente, use a leitura mais razoável, marque cada suposição como `[SUPOSIÇÃO]` e siga.

### Critérios-base (escolher conforme o tipo)
- Negócio: demanda comprovada, disposição a pagar, concorrência, canal de aquisição, recorrência, margem, tempo até o primeiro retorno, dependência de trabalho humano.
- Pesquisa: pergunta clara, originalidade, validade do método, qualidade dos dados, reprodutibilidade, generalização, risco de viés.
- Software/dados: corretude contra o gabarito, cobertura de testes, simplicidade, manutenção, custo de execução.
- Sempre: qualidade da evidência, encaixe com as restrições do usuário.

## Fase 1: rodada de evolução (uma tarefa)

```
Estruturador -> Pesquisador -> Construtor -> Crítico -> Verificação -> Juiz -> Fiscal -> Árbitro
```

- **Estruturador** (só na primeira rodada do projeto): transforma o texto cru na V0 com hipóteses numeradas (H1, H2...), cada uma com impacto e como seria refutada.
- **Pesquisador**: pega a hipótese aberta de maior impacto e busca evidência com fonte. Separa sempre **fato**, **suposição** e **inferência**. Evidência sem fonte não conta; referência só entra com link aberto e conferido.
- **Construtor / implementador**: gera a nova versão ou o código, com testes. **Versões anteriores nunca são alteradas, movidas ou apagadas.**
- **Crítico**: aponta falhas testáveis, cada uma com a condição que a derrubaria. Nada de crítica vaga.
- **Verificação**: o Pesquisador confere as críticas; o Construtor só incorpora as confirmadas.
- **Juiz**: nota cada critério de 0 a 4 com régua fixa:
  - 0 = sem evidência ou evidência contra; 1 = só suposição; 2 = evidência indireta; 3 = evidência direta de uma fonte; 4 = evidência direta e independente de duas fontes ou teste real.
  - **Teto sem evidência**: critério sem fonte nunca passa de 1.
  - Cada nota responde às perguntas sim/não da ESPEC, não a impressão geral.
  - Nota final (0 a 100) calculada por código Python, não estimada: média ponderada × 25. Registra confiança (proporção de critérios com nota ≥ 3) e bloqueios (critérios com 0 que impedem aprovação).
- **Fiscal** (isolado): recebe só a ESPEC e a entrega e responde, item a item: cumpre / não cumpre / não verificável, com motivo; confere também se alguma versão anterior foi alterada; salva `relatorios/FISCAL_<tarefa>_<n>.md`. Até 3 fiscalizações; na terceira reprovação, registra o motivo e entrega como "com ressalvas".
- **Árbitro** (isolado, modelo mais forte, raciocínio estendido): nos PORTÕES, lê nota, parecer do fiscal e decisões já registradas do usuário e decide, sobre a tarefa, `SEGUIR`, `REFAZER`, `TESTAR` ou `APROVAR`. Não espera o usuário para isso. Registra o motivo detalhado em `DECISOES_ARBITRO.md` e anota no STATUS "PORTÃO decidido pelo árbitro, aguardando revisão do usuário". Decisões do usuário valem sobre o árbitro. O árbitro **nunca** decide pivotar, pausar ou encerrar o projeto: se achar que é o caso, abre um alerta de continuidade (seção abaixo).

## Alerta de continuidade (qualquer agente, a qualquer momento)

Continuar, pausar, mudar o escopo, pivotar ou encerrar o projeto é **decisão exclusiva do usuário**. Nenhum agente, nem o árbitro, toma essa decisão.

**Quem pode abrir:** qualquer papel, em qualquer etapa (pesquisa, construção, crítica, julgamento, fiscalização, árbitro, relatório), sempre que perceber que o projeto, ou uma frente, pode não valer a pena. Exemplos: a evidência derruba uma hipótese central; aparece um concorrente que já resolve o problema; o custo ou o prazo real ficou muito acima do previsto; o gabarito não existe ou não é confiável; um critério de parada da ESPEC foi atingido; estagnação (três rodadas seguidas com ganho menor que 3 pontos).

**O que o agente faz:**
1. Registra em `ALERTA_CONTINUIDADE.md` (um bloco por alerta, numerado e datado):
   - **O que foi percebido** e em que etapa;
   - **Evidências**, cada uma com fonte e marcada como fato, suposição ou inferência;
   - **Abrangência**: o projeto inteiro ou só uma frente;
   - **Custo até agora** e o que ainda falta, com estimativa;
   - **Opções**: continuar como está, ajustar o escopo, pivotar, pausar ou encerrar, cada uma com prós e contras;
   - **Recomendação do agente** e o motivo;
   - **O que mudaria essa recomendação** (qual evidência faria valer a pena continuar).
2. Copia a pergunta para `PENDENTE_USUARIO.md` e avisa o usuário (resumo na conversa ou notificação), de forma direta: o que foi visto, o que recomenda e o que precisa que ele decida.
3. **Pausa só o que o alerta atinge**: a frente afetada (ou todas, se o alerta for do projeto inteiro) fica `PAUSADO` na trava, com `quem: aguardando decisão do usuário sobre o alerta <n>`. Nada é apagado nem descartado; as frentes não afetadas seguem.
4. Retoma só quando o usuário responder (na conversa ou registrando a resposta no STATUS): a decisão e o motivo dele vão para `DECISOES.md`, o alerta é marcado como respondido e a trava volta a `LIVRE` se ele mandou continuar.

**Revisão de continuidade periódica** (ex.: 1× por semana, numa execução isolada): reavalia o projeto inteiro contra o plano, o custo até agora, o que falta, sinais externos e critérios de parada. Se encontrar motivo, abre um alerta pela regra acima; se não, registra uma linha "sem alerta" com a data no STATUS.

## Fase 2: execução paralela e agendamento (modos Conduzir e Assumir)

### 2.1 Base compartilhada
Cada execução agendada começa sem memória da conversa anterior. Todo o estado vive em arquivos: de preferência um **repositório GitHub privado** do projeto. No Codex, as frentes trabalham direto no repositório (cada uma no seu ramo, `frente-a`, `frente-b`...). Se a execução não tiver permissão de escrita no repositório, ela entrega os arquivos atualizados como saída e registra no STATUS o que precisa ser salvo; nunca finja ter salvo.

```
COMECE_AQUI.md       ponto de entrada para qualquer execução nova
ESPEC.md             especificação (fonte da verdade do fiscal)
STATUS.md            tarefas por frente, aprovações, pedidos do usuário
EXECUCAO_<F>.md      trava de cada frente
versoes/V0..Vn       versões imutáveis
HIPOTESES.md         hipóteses com status (aberta, confirmada, refutada)
EVIDENCIAS.md        evidências com fonte e tipo (fato/suposição/inferência)
FONTES.md            fontes com link, data e o que foi conferido
NOTAS.md             notas do Juiz por rodada + script do cálculo
DECISOES.md          decisões do usuário e motivos de cada escolha
DECISOES_ARBITRO.md  decisões do árbitro com motivo
ABLACAO.md           tabela de ablação
relatorios/          relatórios de tarefa e do fiscal
PENDENTE_USUARIO.md  decisões irreversíveis e alertas aguardando o usuário
ALERTA_CONTINUIDADE.md  alertas de continuidade com evidências e recomendação
```

### 2.2 Frentes paralelas
- Quebre o projeto em **frentes** (A, B, C...) que podem andar ao mesmo tempo (ex.: código, dados, texto, validação). Cada frente tem ramo ou pasta própria, lista ordenada de tarefas no STATUS e critério de pronto de cada tarefa.
- **Convivência**: cada frente só cria arquivos novos ou edita seções com o nome dela; nunca altera código, configuração ou saídas de outra frente.
- **Dependências**: quando uma tarefa precisa de uma etapa de outra frente, a frente libera a trava com `LIVRE` e `quem: frente X aguardando <ETAPA>` e encerra. Ela só volta quando o STATUS contiver a frase exata `<ETAPA>: aprovada`.
- **Integração**: tarefa aprovada pelo fiscal → junta no principal, roda os testes de novo e registra no STATUS `<ETAPA>: aprovada`.

### 2.3 Trava e sinal de vida (cada frente)
- `EXECUCAO_<F>.md`, primeira linha: `LIVRE`, `OCUPADO`, `PAUSADO` ou `CONCLUIDO`; depois `inicio`, `ultima_atualizacao` (horário de São Paulo) e `quem`.
- Ao disparar, **ler a trava antes de qualquer outra coisa**:
  - `CONCLUIDO` ou `PAUSADO` → encerra em silêncio;
  - `OCUPADO` com sinal há menos de 20 min → encerra em silêncio;
  - `LIVRE` aguardando etapa ainda não aprovada → encerra sem alterar nada;
  - senão, pega a trava (OCUPADO, hora, identificador da execução) e salva **antes** de trabalhar.
- **Sinal de vida a cada 10 min**, salvando a trava com o progresso parcial. Se a trava mostrar outro `quem`, para na hora.
- Ao terminar ou perto do limite de contexto: libera a trava, salva e deixa um resumo curto ao usuário.

### 2.4 Execuções agendadas
Use as **tarefas agendadas** do ChatGPT para o que é pesquisa, texto e relatório, e o **Codex** para o que mexe em código. Antes de criar, confira as tarefas que já existem, para não duplicar.

1. **Uma execução por frente**, recorrente (ex.: a cada hora ou poucas vezes ao dia, conforme o limite do plano), cada frente num horário diferente. Instrução autossuficiente: repositório ou pasta, frente, arquivo de trava, regras da trava (2.3), arquivos a ler, ordem das tarefas, fluxo implementador → testes → relatório → fiscal isolado (até 3) → árbitro nos portões, frase exata de aprovação, regras de convivência e regras permanentes, e quando escrever CONCLUIDO.
2. **Retomada sem vigia**: o ChatGPT não dispara uma tarefa a partir de outra. A recuperação vem da própria regra da trava: na próxima execução agendada, uma trava sem sinal há mais de 20 min é assumida. Por isso a cadência de cada frente define o tempo máximo parado.
3. **Relatório diário** (1× por dia em horário combinado, com notificação): progresso por frente, notas e variação, decisões do árbitro para revisar, pendências do usuário. Trata frente travada (mesma tarefa falhando 3 vezes → marca bloqueada com diagnóstico) e **estagnação** (três rodadas seguidas com ganho menor que 3 pontos → abre alerta de continuidade). Lista os alertas de continuidade abertos no topo do relatório.

### 2.5 Confirmação
Mostre a lista de execuções agendadas (nome, frente, cadência, modelo, o que faz) e crie quando o usuário confirmar; se ele já disse para seguir direto, crie e informe.

## Fase 3: término

Quando o critério de término da ESPEC for atingido, ou o usuário decidir encerrar (por exemplo, respondendo a um alerta de continuidade):
- cada frente escreve CONCLUIDO na trava e o relatório diário gera o relatório final;
- revisão final independente, no modelo mais forte disponível e numa execução isolada (recebe ESPEC, versão final, DECISOES, DECISOES_ARBITRO, ABLACAO), com parecer salvo em `REVISAO_FINAL.md`;
- as execuções agendadas do projeto são **pausadas** e o usuário é avisado; excluir só se ele pedir.

## Regras permanentes

1. **Anotar o motivo de cada decisão** (data, decisão, motivo, alternativas descartadas), para o usuário ler depois e dizer se concorda.
2. **Nunca alterar, mover, apagar ou sobrescrever** nada já feito nem arquivos de outras versões ou de outras frentes; cada mudança vira versão nova.
3. **Ablação**: toda feature, regra ou mudança só fica se a comparação com e sem ela mostrar ganho na métrica da ESPEC. Atualizar `ABLACAO.md` durante o desenvolvimento, não só no fim.
4. **Validar no conjunto inteiro**, não só numa amostra, quando o custo permitir.
5. **Sem loops infinitos**: respeitar limites de rodadas e fiscalizações e o critério de término.
6. **Decisões irreversíveis** (apagar dados, tornar o repositório público, publicar, gastar dinheiro, enviar algo a terceiros, mudar a ESPEC): preparar, registrar em PENDENTE_USUARIO.md e parar aquela frente; quem decide é o usuário. As outras frentes seguem.
7. **Nada inventado**: fato, suposição e inferência sempre marcados; dados e referências só com fonte conferida.
8. **Nenhum segredo nos arquivos do projeto** (chaves, tokens); só pacotes de licença permissiva.
9. **Continuar, pausar, mudar o escopo, pivotar ou encerrar o projeto é decisão só do usuário.** Agentes alertam e recomendam, com motivos e evidências; nunca decidem.
10. **Pedidos do usuário registrados no STATUS valem sobre o árbitro e sobre as instruções das execuções.**

## Modelos e limite de uso

- Frentes, fiscal e relatório: um modelo intermediário, para caber no limite do plano. Árbitro e revisão final: o modelo mais forte disponível, com raciocínio estendido.
- Ajustar o número de frentes e a cadência ao limite de uso do plano; se o relatório perceber que o ritmo vai estourar, reduz a frequência e avisa.

## Saída ao fim de cada tarefa (e no relatório diário)

- versão atual, nota e confiança (com variação em relação à rodada anterior);
- os 3 pontos mais fracos e o que os derrubaria;
- decisão do árbitro e motivo em uma linha (detalhe em DECISOES_ARBITRO.md);
- alertas de continuidade abertos, se houver;
- o que precisa do usuário, se houver.
