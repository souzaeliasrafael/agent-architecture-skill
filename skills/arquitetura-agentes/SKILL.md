---
name: arquitetura-agentes
description: Avalia, evolui e conduz de ponta a ponta qualquer projeto (ideia, pesquisa, software, negócio), novo ou já em andamento, com frentes paralelas, fiscal cego, juiz por rubrica, árbitro de portões e tarefas agendadas que executam e vigiam. Use quando o usuário pedir 'usa a arquitetura de agentes'.
---

# Arquitetura de agentes (IEE + V6)

Junta o fluxo do Idea Evolution Engine (estruturar, pesquisar, construir, criticar, julgar, melhorar) com o modelo de execução do projeto de geoprocessamento V6 (especificação escrita, frentes paralelas na nuvem com trava, sinal de vida e vigia, fiscal que valida cada entrega, árbitro que decide portões sem esperar o usuário e registra o motivo). Serve para qualquer projeto: o que muda de um projeto para outro fica na especificação, nunca nesta skill.

Modos de uso:
- **Avaliar** ("usa a arquitetura de agentes para avaliar X"): Fases 0 e 1 nesta conversa, sem agendamento.
- **Conduzir** ("usa a arquitetura de agentes e toca o projeto"): Fases 0 a 3, com tarefas agendadas executando e vigiando até o fim.
- **Assumir** (projeto já em andamento): Fase A primeiro, depois segue como Conduzir.

## Fase A: assumir projeto em andamento

1. Localize o que já existe: repositório (`list_repos`), pasta, documentos, notas e tarefas agendadas do projeto (`list_triggers`).
2. **Não refaça nem sobrescreva nada.** Leia o estado atual e escreva a ESPEC e o STATUS a partir do que já foi decidido, marcando como `[JÁ FEITO]` o que está pronto e como `[DECIDIDO PELO USUÁRIO]` cada decisão já tomada (essas valem sobre o árbitro).
3. Se já houver tarefas agendadas para o projeto, adapte-as (`update_trigger`) em vez de criar duplicadas.
4. Se o projeto ainda não tiver repositório, peça ao usuário para criar um repositório privado vazio (a sessão pode não ter permissão para criar) e siga com os arquivos locais enquanto isso.

## Fase 0: especificação (obrigatória)

Crie `ESPEC.md` com:
1. **Objetivo** em uma frase e **tipo de projeto** (ideia de negócio, pesquisa/artigo, software, análise de dados, outro).
2. **Gabarito**: contra o que o resultado é medido (base oficial, métrica, cliente real, teste). Se não existir, escreva isso explicitamente.
3. **Critérios do Juiz**: de 6 a 12 da lista abaixo, mais os específicos do projeto, cada um com perguntas sim/não.
4. **Portões**: em que pontos o árbitro decide seguir, refazer, pivotar ou parar.
5. **Restrições**: orçamento, prazo, ferramentas proibidas (ex.: sem API paga), o que nunca pode ser alterado.
6. **Limites**: máximo de rodadas por tarefa (padrão 3, teto 5; até 3 fiscalizações) e **critério de término** do projeto inteiro.

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
- **Construtor / implementador** (subagente): gera a nova versão ou o código, com testes. **Versões anteriores nunca são alteradas, movidas ou apagadas.**
- **Crítico**: aponta falhas testáveis, cada uma com a condição que a derrubaria. Nada de crítica vaga.
- **Verificação**: o Pesquisador confere as críticas; o Construtor só incorpora as confirmadas.
- **Juiz**: nota cada critério de 0 a 4 com régua fixa:
  - 0 = sem evidência ou evidência contra; 1 = só suposição; 2 = evidência indireta; 3 = evidência direta de uma fonte; 4 = evidência direta e independente de duas fontes ou teste real.
  - **Teto sem evidência**: critério sem fonte nunca passa de 1.
  - Cada nota responde às perguntas sim/não da ESPEC, não a impressão geral.
  - Nota final (0 a 100) calculada por código Python, não estimada: média ponderada × 25. Registra confiança (proporção de critérios com nota ≥ 3) e bloqueios (critérios com 0 que impedem aprovação).
- **Fiscal** (subagente separado, Sonnet, sem acesso ao raciocínio de quem fez): recebe só a ESPEC e a entrega e responde, item a item: cumpre / não cumpre / não verificável, com motivo; confere também se alguma versão anterior foi alterada; salva `relatorios/FISCAL_<tarefa>_<n>.md`. Até 3 fiscalizações; na terceira reprovação, registra o motivo e entrega como "com ressalvas".
- **Árbitro** (subagente separado, Opus, esforço alto): nos PORTÕES, lê nota, parecer do fiscal e decisões já registradas do usuário e decide `SEGUIR`, `REFAZER`, `TESTAR`, `PIVOTAR`, `APROVAR` ou `DESCARTAR`. Não espera o usuário. Registra o motivo detalhado em `DECISOES_ARBITRO.md` e anota no STATUS "PORTÃO decidido pelo árbitro, aguardando revisão do usuário". Decisões do usuário valem sobre o árbitro. `APROVAR`, `PIVOTAR` e `DESCARTAR` do projeto inteiro são recomendações até o usuário confirmar.

## Fase 2: execução paralela e agendamento (modos Conduzir e Assumir)

### 2.1 Base compartilhada
Cada execução agendada começa numa sessão nova, sem memória desta conversa. Todo o estado vive num **repositório GitHub privado** do projeto (anexar com `add_repo`, acesso `push`; clonar com `--no-single-branch`). Arquivos de controle:

```
COMECE_AQUI.md       ponto de entrada para qualquer sessão nova
ESPEC.md             especificação (fonte da verdade do fiscal)
STATUS.md            tarefas por frente, aprovações, pedidos do usuário
EXECUCAO_<F>.md      trava de cada frente (no ramo da frente)
versoes/V0..Vn       versões imutáveis
HIPOTESES.md         hipóteses com status (aberta, confirmada, refutada)
EVIDENCIAS.md        evidências com fonte e tipo (fato/suposição/inferência)
FONTES.md            fontes com link, data e o que foi conferido
NOTAS.md             notas do Juiz por rodada + script do cálculo
DECISOES.md          decisões do usuário e motivos de cada escolha
DECISOES_ARBITRO.md  decisões do árbitro com motivo
ABLACAO.md           tabela de ablação
relatorios/          relatórios de tarefa e do fiscal
PENDENTE_USUARIO.md  decisões irreversíveis aguardando o usuário
```

No modo Avaliar, os mesmos arquivos ficam na pasta de trabalho, sem os de trava.

### 2.2 Frentes paralelas
- Quebre o projeto em **frentes** (A, B, C...) que podem andar ao mesmo tempo (ex.: código, dados, texto, validação). Cada frente tem **ramo próprio** (`frente-a`, `frente-b`...), lista ordenada de tarefas no STATUS e critério de pronto de cada tarefa.
- **Convivência**: cada frente só cria arquivos novos ou edita seções com o nome dela; nunca altera código, configuração ou saídas de outra frente.
- **Dependências**: quando uma tarefa precisa de uma etapa de outra frente, a frente libera a trava com `LIVRE` e `quem: frente X aguardando <ETAPA>` e encerra. Ela só volta quando o STATUS contiver a frase exata `<ETAPA>: aprovada`.
- **Integração**: tarefa aprovada pelo fiscal → merge no ramo principal (fetch, merge, testes de novo, push; se recusado, refaz o fetch e tenta de novo) e registro no STATUS com `<ETAPA>: aprovada`.

### 2.3 Trava e sinal de vida (cada frente)
- `EXECUCAO_<F>.md`, primeira linha: `LIVRE`, `OCUPADO`, `PAUSADO` ou `CONCLUIDO`; depois `inicio`, `ultima_atualizacao` (horário de São Paulo) e `quem`.
- Ao disparar, **ler a trava antes de qualquer outra coisa**:
  - `CONCLUIDO` ou `PAUSADO` → encerra em silêncio;
  - `OCUPADO` com sinal há menos de 20 min (o mais recente entre `ultima_atualizacao` e o último commit do ramo) → encerra em silêncio;
  - `LIVRE` aguardando etapa ainda não aprovada → encerra sem alterar nada;
  - senão, pega a trava (OCUPADO, hora, id da sessão), commit e push **antes** de trabalhar; se o push for recusado, encerra.
- **Sinal de vida a cada 10 min** (commit + push da trava com o progresso parcial), inclusive durante cálculos longos, que rodam em segundo plano. Antes de cada push, `git pull`; se a trava remota mostrar outro `quem`, para na hora.
- Ao terminar ou perto do limite de contexto: libera a trava, commit, push e resumo curto ao usuário.

### 2.4 Tarefas agendadas
Antes de criar, veja com `list_triggers` o que já existe (deste e de outros projetos), para não duplicar e para dividir o limite de uso. Crie com `create_trigger` (nuvem, `requires_local_device` falso salvo necessidade real):

1. **Uma tarefa por frente** (Sonnet), recorrente a cada hora, cada frente num minuto diferente para não colidir. Prompt autossuficiente com: repositório, ramo, arquivo de trava, regras da trava (2.3), arquivos a ler, ordem das tarefas da frente, fluxo implementador → testes → relatório → fiscal (até 3) → árbitro nos portões, frase exata de aprovação, regras de convivência e regras permanentes, e quando escrever CONCLUIDO.
2. **Vigia** (Haiku, tarefa curta e mecânica, várias vezes por hora em minutos intercalados): lê só as travas e o último commit de cada frente. Se uma frente está LIVRE (ou aguardando etapa já aprovada) ou OCUPADA sem sinal há mais de 20 min, libera a trava com um commit `vigia: disparo frente X <hora>` e chama `fire_trigger` na tarefa daquela frente. Não dispara de novo se houve disparo do vigia há menos de 15 min. Termina sem resumo. Converter todos os horários para São Paulo antes de comparar.
3. **Relatório diário** (Sonnet, 1× por dia em horário combinado, notificação push ligada): progresso por frente, notas e variação, decisões do árbitro para revisar, pendências do usuário. Trata: frente travada (mesma tarefa falhando 3 vezes → marca bloqueada com diagnóstico e avisa) e **estagnação** (três rodadas seguidas com ganho menor que 3 pontos → aciona o árbitro).

### 2.5 Confirmação
Mostre a lista de tarefas agendadas (nome, frente, cadência, modelo, o que faz) e crie quando o usuário confirmar; se ele já disse para seguir direto, crie e informe. Diga em uma frase que configuração de aprovação cada tarefa recebeu; se as execuções forem pedir aprovação, avise que ele pode ativar "Aprovar automaticamente" nas configurações da tarefa.

## Fase 3: término

Quando o critério de término da ESPEC for atingido, ou o árbitro recomendar `APROVAR`/`DESCARTAR` para o projeto inteiro:
- cada frente escreve CONCLUIDO na trava e o relatório diário gera o relatório final;
- revisão final com um subagente Fable (recebe ESPEC, versão final, DECISOES, DECISOES_ARBITRO, ABLACAO) e parecer independente salvo em `REVISAO_FINAL.md`;
- as tarefas agendadas do projeto são **pausadas** (`update_trigger` com enabled falso) e o usuário é avisado; excluir só se ele pedir.

## Regras permanentes

1. **Anotar o motivo de cada decisão** (data, decisão, motivo, alternativas descartadas), para o usuário ler depois e dizer se concorda.
2. **Nunca alterar, mover, apagar ou sobrescrever** nada já feito nem arquivos de outras versões ou de outras frentes; cada mudança vira versão nova ou commit novo.
3. **Ablação**: toda feature, regra ou mudança só fica se a comparação com e sem ela mostrar ganho na métrica da ESPEC. Atualizar `ABLACAO.md` durante o desenvolvimento, não só no fim.
4. **Validar no conjunto inteiro**, não só numa amostra, quando o custo permitir.
5. **Sem loops infinitos**: respeitar limites de rodadas e fiscalizações e o critério de término.
6. **Decisões irreversíveis** (apagar dados, tornar o repositório público, publicar, gastar dinheiro, enviar algo a terceiros, mudar a ESPEC): preparar, registrar em PENDENTE_USUARIO.md e parar aquela frente; quem decide é o usuário. As outras frentes seguem.
7. **Nada inventado**: fato, suposição e inferência sempre marcados; dados e referências só com fonte conferida.
8. **Nenhum segredo no repositório** (chaves, tokens); só pacotes de licença permissiva.
9. **Pedidos do usuário registrados no STATUS valem sobre o árbitro e sobre o prompt das tarefas.**

## Modelos e limite de uso

- Frentes, fiscal e relatório: Sonnet. Árbitro: Opus, esforço alto. Vigia: Haiku. Revisão final: Fable (só quando o árbitro recomendar APROVAR, ou quando o usuário pedir).
- Ajustar o número de frentes e a cadência ao limite semanal, considerando as tarefas agendadas de outros projetos. Se o relatório perceber que o ritmo vai estourar, reduz a frequência (`update_trigger`) e avisa.

## Saída ao fim de cada tarefa (e no relatório diário)

- versão atual, nota e confiança (com variação em relação à rodada anterior);
- os 3 pontos mais fracos e o que os derrubaria;
- decisão do árbitro e motivo em uma linha (detalhe em DECISOES_ARBITRO.md);
- o que precisa do usuário, se houver.
