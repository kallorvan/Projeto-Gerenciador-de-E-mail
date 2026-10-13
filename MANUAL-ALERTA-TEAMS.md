# Manual — Alerta no Teams por termos no e-mail (Power Automate)

Este manual monta, do zero, um fluxo que lê cada e-mail novo da **Caixa de Entrada** do Outlook
e, quando encontra os termos configurados no **assunto ou no corpo**, envia uma mensagem no
**Teams** com o e-mail que disparou o alerta e um link **"Abrir no Outlook"**.

> Tudo é feito no navegador, sem instalar nada. Use **a conta corporativa de quem vai receber
> os alertas** — o fluxo lê a caixa de e-mail da conta que estiver conectada a ele.

---

## Sumário

0. [Antes de começar](#0-antes-de-começar)
1. [Criar o fluxo e o gatilho](#1-criar-o-fluxo-e-o-gatilho)
2. [Converter o corpo do e-mail em texto (CorpoTexto)](#2-converter-o-corpo-do-e-mail-em-texto-corpotexto)
3. [Condição com os termos](#3-condição-com-os-termos)
4. [Montar o link "Abrir no Outlook" (LinkOutlook)](#4-montar-o-link-abrir-no-outlook-linkoutlook)
5. [Mensagem no Teams](#5-mensagem-no-teams)
6. [Salvar, ativar e testar](#6-salvar-ativar-e-testar)
7. [Manutenção do dia a dia](#7-manutenção-do-dia-a-dia)
8. [Problemas comuns](#8-problemas-comuns)
9. [Referência rápida — todas as expressões](#9-referência-rápida--todas-as-expressões)

---

## 0. Antes de começar

- [ ] Acessar **https://make.powerautomate.com** e entrar com a **conta corporativa** da pessoa
      que vai usar o fluxo.
- [ ] Conferir quem está logado: clique na foto/iniciais no **canto superior direito**.
      Se for outra pessoa, saia (ou use uma **janela anônima**: `Ctrl+Shift+N`).
- [ ] No canto superior direito, em **Ambiente**, deixe o ambiente padrão da empresa.

**Como inserir valores nos campos** (vale para o manual todo):

| Símbolo no manual | O que fazer |
|---|---|
| **⚡ Nome** | Clique no campo → ícone **⚡** (conteúdo dinâmico) → escolha o item **Nome**. Se o ícone não aparecer, digite `/` no campo → *Inserir conteúdo dinâmico*. |
| **fx** `expressão` | Clique no campo → ícone **fx** → cole a expressão → **Adicionar** (ou **Atualizar**). |
| `texto` | Digite normalmente. |

> **Nomes das ações importam.** As expressões citam as ações pelo nome (`CorpoTexto`,
> `LinkOutlook`). Renomeie exatamente como indicado — com maiúsculas e sem espaço.

---

## 1. Criar o fluxo e o gatilho

1. Menu da esquerda → **+ Criar** → **Fluxo da nuvem automatizado**.
2. **Nome do fluxo:** `Alerta Teams - Termos`
3. **Gatilho:** busque `novo email` e selecione **"Quando um novo email é recebido (V3)"**
   — ícone do **Office 365 Outlook**.
   Não use as versões "compartilhada", "sinalizado" ou "me mencionar".
4. Clique em **Criar**.
5. Clique no bloco do gatilho e configure (alguns campos ficam em **Parâmetros avançados → Mostrar tudo**):

| Campo | Valor |
|---|---|
| Pasta | **Caixa de Entrada** |
| Incluir Anexos | **Não** |
| Importância | **Qualquer** |
| Filtro de Assunto | *(vazio)* |
| Para / CC / Para ou CC | *(vazios)* |

6. No rodapé do painel, confirme **"Conectado a <e-mail da pessoa>"**.
   Se aparecer outra conta, clique em **Alterar conexão**.

> O gatilho cobre e-mails em que a pessoa está em **Para, Cc ou Cco**. Ele só olha a pasta
> escolhida — subpastas **não** entram. Se alguma **regra do Outlook** move e-mails para
> subpastas, veja a seção [7](#7-manutenção-do-dia-a-dia).

---

## 2. Converter o corpo do e-mail em texto (CorpoTexto)

1. Clique no **⊕** abaixo do gatilho → **Adicionar uma ação**.
2. Busque `html para texto` → selecione **"HTML em texto"** (conector *Conversão de Conteúdo* /
   *Content Conversion*).
3. **Conteúdo:** **⚡ Corpo** (do gatilho "Quando um novo email é recebido (V3)").
4. **Renomeie** a ação: clique no título no topo do painel (ou **⋮ → Renomear**) e digite:

   ```
   CorpoTexto
   ```

---

## 3. Condição com os termos

A regra usada é: **alerta somente quando o e-mail tiver pelo menos um termo do Grupo 1
E pelo menos um termo do Grupo 2.**

```
E ⌄
 ├─ Grupo [Ou]  — assunto principal
 │   ├─ contém deadline
 │   ├─ contém instruc
 │   ├─ contém instruç
 │   └─ contém ordem
 └─ Grupo [Ou]  — tipo de mudança
     ├─ contém altera
     ├─ contém modific
     ├─ contém corrig
     ├─ contém correç
     └─ contém correc
```

| Exemplo de e-mail | Alerta? |
|---|---|
| "Deadline do projeto" | ❌ (só Grupo 1) |
| "Documento corrigido" | ❌ (só Grupo 2) |
| "Instrução alterada" | ✅ (Grupo 1 + Grupo 2) |
| "Houve alteração no deadline" | ✅ |

### 3.1 Inserir a Condição

1. **⊕** abaixo de **CorpoTexto** → **Adicionar uma ação** → busque `condição` → **Condição** (*Controle*).

### 3.2 Montar o Grupo 1

1. Clique na setinha ao lado de **+ Adicionar linha** → **Adicionar grupo**.
2. No grupo, mude o seletor para **Ou**.
3. Para cada termo do Grupo 1, preencha uma linha (use **+ Adicionar linha** dentro do grupo):
   - **Valor da esquerda:** **fx**
     ```
     toLower(concat(triggerOutputs()?['body/subject'], ' ', body('CorpoTexto')))
     ```
   - **Operador:** `contém`
   - **Valor da direita:** o termo, **em minúsculas**:
     `deadline` · `instruc` · `instruç` · `ordem`

### 3.3 Montar o Grupo 2

1. Fora do Grupo 1, clique de novo na setinha de **+ Adicionar linha** → **Adicionar grupo**.
2. Seletor do grupo em **Ou**.
3. Uma linha por termo, com a **mesma expressão** à esquerda e operador `contém`:
   `altera` · `modific` · `corrig` · `correç` · `correc`

### 3.4 Ajustar o seletor principal

- O seletor **do topo** (fora dos grupos) deve ficar em **E**.
- Se sobrou alguma linha solta fora dos grupos, exclua (**⋯ → Excluir**).

### 3.5 Conferir no código

O designer mostra uma **linha em branco** no fim de cada grupo ("Escolher um valor / é igual a").
Ela é só visual e não é salva. Para confirmar:

1. Com a Condição selecionada, abra a aba **Exibição de código**.
2. Deve haver `"and"` com dois blocos `"or"`, cada um só com linhas `"contains"`.
3. **Não** pode existir `"equals": [ "", "" ]`. Se existir, preencha essa linha como
   **fx** `false` · `é igual a` · **fx** `true` (linha sempre falsa, não interfere).

### Por que os termos estão "cortados"

O fluxo procura **trechos de texto**, ignorando maiúsculas/minúsculas, mas **não** ignora acentos
nem corrige erros de digitação. Usar a raiz da palavra cobre as variações:

| Cadastrado | Pega | Não pega |
|---|---|---|
| `deadline` | deadline, deadlines, Deadline | dealine (erro de digitação) |
| `instruç` / `instruc` | instrução, instruções / instrucao, instrucoes | instrumento, instrutor |
| `ordem` | ordem, ordens, ordem de serviço | — (palavra comum: pode gerar alertas extras) |
| `altera` | alterado(a/s), alteração, alterar | — |
| `modific` | modificado(a/s), modificação, modificar | — |
| `corrig` | corrigido(a/s), corrigir, corrige | — |
| `correç` / `correc` | correção, correções / correcao, correcoes | — |

---

## 4. Montar o link "Abrir no Outlook" (LinkOutlook)

**4.0 — Perfil de quem usa o fluxo (destinatário automático)**

1. Na Condição, no quadro **Verdadeiro**, clique no **⊕** → **Adicionar uma ação**.
2. Busque `obter meu perfil` → **"Obter meu perfil (V2)"** (*Office 365 Users*). Nenhum campo a preencher.
3. **Renomeie** para:

   ```
   MeuPerfil
   ```

**4.1 — LinkOutlook**

1. No quadro **Verdadeiro**, clique no **⊕ abaixo de MeuPerfil** → **Adicionar uma ação**.
2. Busque `compor` → **Compor** (*Operação de Dados*).
3. **Renomeie** para:

   ```
   LinkOutlook
   ```

4. **Entradas:** **fx**

   ```
   concat('<a href="https://outlook.office365.com/owa/?ItemID=', encodeUriComponent(triggerOutputs()?['body/id']), '&exvsurl=1&viewmodel=ReadMessageItem">Abrir no Outlook</a>')
   ```

5. Deixe o quadro **Falso** vazio.

---

## 5. Mensagem no Teams

1. No quadro **Verdadeiro**, clique no **⊕ abaixo de LinkOutlook** → **Adicionar uma ação**.
2. Busque `postar mensagem` → **"Postar mensagem em um chat ou canal"** (*Microsoft Teams*).
3. Preencha:

| Campo | Valor |
|---|---|
| Postar como | **Bot do Flow** (Flow bot) |
| Postar em | **Conversar com o bot do Flow** (Chat with Flow bot) |
| Recipient | **fx** `coalesce(outputs('MeuPerfil')?['body/mail'], outputs('MeuPerfil')?['body/userPrincipalName'])` — envia sempre para quem está conectado ao fluxo. Se o campo só mostrar a busca de pessoas, clique no ⚙️ ao lado ou em *Inserir valor personalizado* e use o **fx**. |

4. **Mensagem** — monte linha por linha (use **Enter** para quebrar linha):

```
📩 E-mail com termo monitorado        ← digite
De: [⚡ De]                             ← digite "De: " e insira ⚡ De
Assunto: [⚡ Assunto]                   ← ⚡ Assunto
Recebido em: [⚡ Hora de recebimento]   ← ⚡ Hora de recebimento (Received Time)

[fx corpo limpo]                        ← fx, expressão abaixo

[⚡ Saídas de LinkOutlook]              ← ⚡ Saídas (grupo LinkOutlook)
```

Expressão do **corpo limpo** (remove assinatura, links de imagens e histórico de respostas,
e limita o tamanho para o Teams aceitar):

```
take(trim(first(split(first(split(body('CorpoTexto'), '[http')), '________________'))), 4000)
```

> Os colchetes `[ ]` acima só indicam onde inserir cada item — **não** digite colchetes.

---

## 6. Salvar, ativar e testar

1. Clique em **Salvar** (canto superior direito). Se aparecer erro no **Verificador de fluxo**,
   veja a seção [8](#8-problemas-comuns).
2. Volte para a página do fluxo e confirme que está **Ativado** (se não, clique em **Ativar**).
3. Envie três e-mails **para a própria conta**:

| Assunto do teste | Resultado esperado |
|---|---|
| `Deadline do projeto` | ❌ sem alerta |
| `Documento corrigido` | ❌ sem alerta |
| `Instrução alterada` | ✅ alerta no Teams |

4. Aguarde **1 a 3 minutos** (o Power Automate verifica a caixa periodicamente).
5. O alerta chega no Teams, no chat **Power Automate / Fluxos de trabalho**.
6. Clique em **Abrir no Outlook** na mensagem e confira se abre o e-mail correto.

Para ver o que aconteceu em cada e-mail: página do fluxo → **Histórico de execuções**.

---

## 7. Manutenção do dia a dia

### Incluir um termo
Fluxo → **Editar** → **Condição** → no grupo certo, **+ Adicionar linha** → mesma expressão à
esquerda, `contém`, termo em **minúsculas** → **Salvar**.

- Plural/acento: cadastre a **raiz** (`passage` → passagem/passagens).
- Palavra com acento no meio: cadastre **com e sem acento** (`reunião` e `reuniao`).
- Termo composto (`nota fiscal`): só casa se as palavras estiverem **juntas, com um espaço**.
  Para "as duas palavras em qualquer lugar", crie um **grupo E** com `nota` e `fiscal`.
- Evite raízes curtas demais (`via`, `nf`) — casam dentro de outras palavras.

### Mudar a lógica para "qualquer termo dispara"
Troque o seletor **do topo** de **E** para **Ou**.

### Pausar o alerta
**Meus fluxos** → selecione o fluxo → **Desativar**.

### Subpastas
O fluxo olha só a **Caixa de Entrada**. E-mails que **você** move depois já foram verificados.
Se uma **regra do Outlook** move e-mails direto para uma subpasta: **Meus fluxos → ⋮ → Salvar como**,
abra a cópia, troque a **Pasta** do gatilho para a subpasta, salve e **ative**.
(Ao mudar termos, mude em todas as cópias.)

### Anexos em PDF
O fluxo **não lê** o conteúdo de anexos. Ler PDF exige AI Builder (licença premium).

### Usar o fluxo em outra conta
O jeito mais seguro é **montar do zero com este manual**, logado na conta da outra pessoa.
Alternativa: **Meus fluxos → ⋮ → Enviar uma cópia** → a outra pessoa abre a cópia → **Editar** →
em cada bloco, **Alterar conexão** para a conta dela → troca o **Recipient** do Teams para ela →
**Salvar** → **Ativar**.

> ⚠️ Nunca deixe o fluxo de outra pessoa usando a **sua** conexão do Outlook: ele leria o **seu** e-mail.

---

## 8. Problemas comuns

| Sintoma | Causa provável | Solução |
|---|---|---|
| Erro citando `CorpoTexto` ou `LinkOutlook` "not found" | Ação com nome diferente | Renomeie exatamente `CorpoTexto` / `LinkOutlook` |
| Alerta para **todos** os e-mails | Linha vazia contando como verdadeira, ou seletor do topo em **Ou** | Confira a **Exibição de código** (3.5) e o seletor do topo em **E** |
| Nenhum alerta chega | Fluxo desativado, termo com acento diferente, ou Recipient errado | Ative o fluxo; confira os termos; confira o Recipient |
| Alerta chega para outra pessoa | **Recipient** do Teams aponta para outra conta | Troque o Recipient |
| Corpo vem cortado no meio | Havia um link `[http…]` no texto | Limitação da limpeza — use **Abrir no Outlook** |
| Aparece `<a href=…>` escrito | A mensagem não interpretou HTML | Confirme que o item inserido é **Saídas** de `LinkOutlook` |
| Link não abre o e-mail | E-mail movido/excluído, ou outro domínio do Outlook Web | Troque `outlook.office365.com` por `outlook.office.com` em `LinkOutlook` |
| Erro de conexão/permissão ao salvar | Política de dados (DLP) da empresa | Falar com o TI |

---

## 9. Referência rápida — todas as expressões

**Texto pesquisado** (valor da esquerda de todas as linhas da Condição):
```
toLower(concat(triggerOutputs()?['body/subject'], ' ', body('CorpoTexto')))
```

**Versão opcional** que também trata quebras de linha e espaços especiais (útil para termos compostos):
```
toLower(replace(replace(replace(concat(triggerOutputs()?['body/subject'], ' ', body('CorpoTexto')), decodeUriComponent('%0D'), ' '), decodeUriComponent('%0A'), ' '), decodeUriComponent('%C2%A0'), ' '))
```

**LinkOutlook** (ação Compor):
```
concat('<a href="https://outlook.office365.com/owa/?ItemID=', encodeUriComponent(triggerOutputs()?['body/id']), '&exvsurl=1&viewmodel=ReadMessageItem">Abrir no Outlook</a>')
```

**Recipient automático** (ação do Teams):
```
coalesce(outputs('MeuPerfil')?['body/mail'], outputs('MeuPerfil')?['body/userPrincipalName'])
```

**Corpo limpo** (mensagem do Teams):
```
take(trim(first(split(first(split(body('CorpoTexto'), '[http')), '________________'))), 4000)
```

**Estrutura final do fluxo:**
```
Quando um novo email é recebido (V3) — Caixa de Entrada
└─ CorpoTexto (HTML em texto)
   └─ Condição  [E]  Grupo 1 [Ou]  +  Grupo 2 [Ou]
      ├─ Verdadeiro:
      │   ├─ MeuPerfil (Obter meu perfil V2)
      │   ├─ LinkOutlook (Compor)
      │   └─ Postar mensagem em um chat ou canal (Teams)
      └─ Falso: (vazio)
```
