# 2. Fluxo 1 — Captura de e-mails

Em https://make.powerautomate.com → **Criar** → **Fluxo da nuvem automatizado**.
Nome sugerido: `Email - Captura de termos`.

> **Renomeie cada ação** com o nome indicado em **negrito** (menu `…` → Renomear)
> *antes* de usar expressões que se referem a ela. As expressões abaixo dependem
> desses nomes. Para colar uma expressão, clique no campo → ícone **fx** (Expressão).

## Visão geral

```
Gatilho: Quando um novo email chegar (V3)
├─ HtmlParaTexto        (Conversão de Conteúdo: Html para texto)
├─ TextoBusca           (Compor)
├─ ObterTermos          (SharePoint: Obter itens)
├─ FiltrarTermos        (Filtrar matriz)
└─ TemTermo?            (Condição)
   └─ Sim:
      ├─ NomesTermos    (Selecionar)
      ├─ Prioridades    (Selecionar)
      ├─ CriarItem      (SharePoint: Criar item)
      ├─ SinalizarEmail (Outlook: Sinalizar email (V2))
      └─ (opcional) AlertaTeams
```

## Passos

### Gatilho — Office 365 Outlook: *Quando um novo email chegar (V3)*
- Pasta: `Inbox` (Caixa de Entrada)
- Incluir Anexos: `Não`
- Importância: `Any`

### **HtmlParaTexto** — Conversão de Conteúdo: *Html para texto*
- Conteúdo (expressão): `triggerOutputs()?['body/body']`

### **TextoBusca** — *Compor*
- Entradas (expressão):
  ```
  toLower(concat(triggerOutputs()?['body/subject'], ' ', body('HtmlParaTexto')))
  ```

  > Quer buscar só no assunto + início do corpo (evita disparar por texto citado de
  > respostas antigas)? Use:
  > `toLower(concat(triggerOutputs()?['body/subject'], ' ', take(body('HtmlParaTexto'), 1500)))`

### **ObterTermos** — SharePoint: *Obter itens*
- Endereço do Site: o site/Lists onde criou as listas
- Nome da Lista: `Termos`
- Consulta de Filtro: `Ativo eq 1`

### **FiltrarTermos** — *Filtrar matriz* (Data Operation / Operação de Dados)
- De (expressão): `outputs('ObterTermos')?['body/value']`
- Clique em **Editar no modo avançado** e cole:
  ```
  @contains(outputs('TextoBusca'), toLower(trim(item()?['Title'])))
  ```

### **TemTermo?** — *Condição*
- Valor à esquerda (expressão): `length(body('FiltrarTermos'))`
- Operador: `é maior que`
- Valor à direita: `0`

### Ramo **Sim**

**NomesTermos** — *Selecionar*
- De: `body('FiltrarTermos')`
- Mapear: clique no ícone **Alternar para modo de texto** e cole a expressão
  `item()?['Title']`

**Prioridades** — *Selecionar*
- De: `body('FiltrarTermos')`
- Mapear (modo de texto, expressão): `int(coalesce(item()?['Prioridade'], 3))`

**CriarItem** — SharePoint: *Criar item* (lista `EmailsEmEvidencia`)

| Campo | Expressão |
|---|---|
| Título | `triggerOutputs()?['body/subject']` |
| Remetente | `triggerOutputs()?['body/from']` |
| RecebidoEm | `triggerOutputs()?['body/receivedDateTime']` |
| Termos | `join(body('NomesTermos'), ', ')` |
| Prioridade | `min(body('Prioridades'))` |
| Status Value | `Pendente` (texto) |
| Trecho | `take(body('HtmlParaTexto'), 500)` |
| LinkOutlook | `concat('https://outlook.office365.com/owa/?ItemID=', encodeUriComponent(triggerOutputs()?['body/id']), '&exvsurl=1&viewmodel=ReadMessageItem')` |
| IdConversa | `triggerOutputs()?['body/conversationId']` |
| IdMensagem | `triggerOutputs()?['body/id']` |

**SinalizarEmail** — Office 365 Outlook: *Sinalizar email (V2)*
- Id da Mensagem: `triggerOutputs()?['body/id']`
- Status do Sinalizador: `flagged`

  Assim o e-mail também fica em evidência dentro do próprio Outlook.

**AlertaTeams** *(opcional)* — coloque dentro de uma *Condição*
`min(body('Prioridades'))` **é igual a** `1`, e use Microsoft Teams:
*Postar mensagem em um chat ou canal* → Postar como `Flow bot`, Postar em `Chat com o bot de Fluxo`:
```
🔴 E-mail prioritário: @{triggerOutputs()?['body/subject']}
De: @{triggerOutputs()?['body/from']}
Termos: @{join(body('NomesTermos'), ', ')}
```

## Testar

1. Salve o fluxo e clique em **Testar → Manualmente**.
2. Envie para você mesmo um e-mail com um termo cadastrado (ex.: "teste urgente").
3. Confira a execução no histórico e o novo item na lista `EmailsEmEvidencia`.

## Problemas comuns

| Sintoma | Causa provável |
|---|---|
| `The template language expression ... 'HtmlParaTexto' ... not found` | a ação não foi renomeada exatamente com esse nome |
| Erro de filtro em `ObterTermos` | a coluna foi criada com outro nome interno (veja dica em [01-listas.md](01-listas.md)) |
| Nenhum termo encontrado mesmo existindo | diferença de acento (`reunião` × `reuniao`) |
| Link não abre o e-mail | o e-mail foi movido/excluído, ou a empresa usa outro domínio do Outlook Web (troque `outlook.office365.com` por `outlook.office.com`) |
