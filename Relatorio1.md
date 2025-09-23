# S03 — Relatório de feedback (1º envio)

**Aluno:** Vinícius Santos Araújo
**Data:** 23/09/2025

## Grupo 1 — Jogadores

**O que achei legal**

* Já pensaram em **cadastro/login** desde o começo (várias coisas dependem disso).
* Ter um **repositório único** pra falar com o banco deixa a casa mais organizada.

**O que pode melhorar**

* **Tokens**: faltou explicar expiração/refresh/revogação.
* **Mistura de camadas**: `UserController` e entidade `User` com responsabilidades a mais.
* **Diagrama** está mais complexo do que precisa pro estágio atual.

**Dicas práticas**

* Usar **JWT** com *access token* curto + **refresh token** e rotas de refresh/logout.
* Separar **Controller → Service → Repository** (cada um no seu quadrado).
* Simplificar o diagrama e documentar dois eventos: `UserCreated` e `UserLoggedIn` (pra outros grupos consumirem).

---

## Grupo 2 — Distribuição de cartas

**O que achei legal**

* Escopo objetivo (**sorteio/distribuição**) e preocupação com **integridade** (não duplicar pro mesmo jogador).
* Ideia de **encapsular integrações externas** (bom pra testes e mocks).

**O que pode melhorar**

* **Acoplamento** com a **PokéAPI** se não houver um serviço próprio no meio.
* Se a distribuição também **registrar posse**, pode virar **gargalo**.
* Falta deixar explícito: só distribui se o jogador **existe e está autenticado** (via Grupo 1).

**Dicas práticas**

* Criar **ServiçoPokéAPI** (adapter) pra evitar dependência direta.
* Centralizar **registro de posse** num **repositório de “Cartas do Jogador”** (consumido por 4, 5 e 6).
* Deixar a distribuição **enxuta**: faz o **sorteio** e **emite evento**; quem persiste é o repositório de posse.

---

## Grupo 3 — Trocas

**O que achei legal**

* Responsabilidade clara: **propostas** e **notificações** de troca.
* Sinalização de que precisa de algo **quase em tempo real**.

**O que pode melhorar**

* Só **REST** tende a virar *polling* caro/lento.
* Risco de **inconsistência** se permitir trocar carta que o jogador **não possui mais**.

**Dicas práticas**

* Usar **eventos/filas** ou **WebSocket** (notificação de propostas/aceites).
* Validar posse consultando o **repositório central** (Grupo 2 → “Cartas do Jogador”).
* Separar **módulo de notificação** do **módulo de registro de trocas**.

---

## Grupo 4 — Visualização de Cartas

**O que achei legal**

* Boa ideia de um **“hub” de visualização** (começa simples e dá pra crescer).
* Tendência a **modularização** nos controladores.

**O que pode melhorar**

* Falta um **Player** como entidade central com **PlayerController** orquestrando.
* **Entidades conversando entre si** podem acoplar demais (prefira services/controladores).
* Atualização **quando chegam novas cartas/trocas** ainda pouco definida.

**Dicas práticas**

* Criar **PlayerController** como *façade* que chama serviços (cartas, stats, histórico).
* Assinar eventos: `CardUpdated`, `TradeCompleted` (atualizar telas).
* Documentar **contratos de API/DTO** e já pensar em **paginação/filtro**.

---

## Grupo 5 — Visualização de Trocas

**O que achei legal**

* Age como **frontend de trocas** (boa separação de papéis).
* Intenção de deixar o **negócio** no Grupo 3 e manter aqui só **UI**.

**O que pode melhorar**

* Dependências fortes com **Grupo 1 (autenticar)** e **Grupo 3 (trocas)** precisam estar bem documentadas.
* Risco de **duplicar regras** se começar a validar lógica de troca na UI.

**Dicas práticas**

* Inscrever-se em **notificações** de trocas (tempo real) e só **espelhar** estados.
* Manter camada de **serviços de UI** fina (sem regras de negócio), reaproveitando **DTOs** do Grupo 3.

---

## Grupo 6 — Painel de Administração

**O que achei legal**

* Proposta de **visão global** (monitoramento/auditoria) é muito útil.
* Preocupação com **perfis de acesso** (admin ≠ jogador).

**O que pode melhorar**

* **Acoplamento** se o painel consultar cada grupo diretamente.
* Pode virar “**super sistema**” se começar a **decidir** sobre posse/execução.

**Dicas práticas**

* Ser **consumidor de eventos/logs** (trocas, partidas, distribuição) em vez de *n* chamadas síncronas.
* Estender autenticação/autorização do **Grupo 1** com papéis de admin.

---

## Grupo 7 — Gestão de Partidas

**O que achei legal**

* Papel de **orquestrador** (sem calcular a batalha) está na direção certa.
* Regras pensadas: evitar **auto-pareamento**, discutir **rematch**, etc.

**O que pode melhorar**

* Pode ocorrer **duplicidade** de partidas simultâneas se não houver bloqueios/estados.
* Integrações com **Grupo 1** (validar jogadores) e **Grupo 8** (executar batalha) precisam de contratos explícitos.

**Dicas práticas**

* Modelar **estados de partida** (criando, agendado, em\_andamento, concluído) e **locks** por jogador.
* Contratos claros: `StartBattleRequest/Result` com o **Grupo 8**.

---

## Grupo 8 — Batalha

**O que achei legal**

* Ideia de **motor de regras** separado (probabilidades, vantagens de tipo).
* Preocupação em **isolar estado de batalha**.

**O que pode melhorar**

* Guardar histórico aqui **mistura papéis** (isso é do Grupo 7).
* Definir **limites de escopo**: recebe entrada completa, **só calcula** e devolve.

**Dicas práticas**

* Manter o motor em **módulo próprio** (facilita evolução/testes).
* Estado por **requisição** ou **repositório temporário**; nada de histórico persistido aqui.

---

## Grupo 9 — Pokédex

**O que achei legal**

* Escopo claro de **catálogo/consulta** e vontade de integrar com trocas pra manter dados atuais.
* Focar em **cartas conhecidas** reduz ambiguidade no começo.

**O que pode melhorar**

* Falta definição única do que é **“conhecido”** (distribuição? troca? batalha?).
* Dependência de **duas fontes** (trocas e distribuição) pode ser redundante.

**Dicas práticas**

* Regra clara: “**conhecido** = entrou via **distribuição** (ou chegou por **troca**)” e registrar isso num campo.
* Escolher **uma fonte de verdade** (sugestão: distribuição) e complementar com eventos de trocas.
* Planejar **índices de busca** (nome, tipo, raridade) pra escalar sem travar.

---

