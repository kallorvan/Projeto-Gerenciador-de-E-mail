# 3. Fluxo 2 — Marcar como respondido

Quando você envia qualquer e-mail numa conversa que tem itens pendentes, eles passam
para `Respondido` automaticamente.

Em https://make.powerautomate.com → **Criar** → **Fluxo da nuvem automatizado**.
Nome sugerido: `Email - Marcar respondido`.

```
Gatilho: Quando um novo email chegar (V3)  — pasta Itens Enviados
├─ PendentesDaConversa  (SharePoint: Obter itens)
└─ Aplicar a cada
   └─ MarcarRespondido  (SharePoint: Atualizar item)
```

### Gatilho — Office 365 Outlook: *Quando um novo email chegar (V3)*
- Pasta: `Sent Items` (Itens Enviados)
- Incluir Anexos: `Não`

### **PendentesDaConversa** — SharePoint: *Obter itens*
- Lista: `EmailsEmEvidencia`
- Consulta de Filtro (cole como texto e insira a expressão no meio):
  ```
  IdConversa eq '@{triggerOutputs()?['body/conversationId']}' and Status eq 'Pendente'
  ```

### *Aplicar a cada*
- Selecionar saída (expressão): `outputs('PendentesDaConversa')?['body/value']`

### **MarcarRespondido** — SharePoint: *Atualizar item* (dentro do loop)

| Campo | Expressão |
|---|---|
| Id | `items('Aplicar_a_cada')?['ID']` |
| Título | `items('Aplicar_a_cada')?['Title']` (obrigatório repetir) |
| Status Value | `Respondido` |
| RespondidoEm | `utcNow()` |

> Se o loop tiver outro nome (ex.: `Apply_to_each`), ajuste dentro de `items('...')` —
> espaços viram `_`.

### *(Opcional)* Remover o sinalizador no Outlook
Dentro do loop, adicione Office 365 Outlook: *Sinalizar email (V2)* com
Id da Mensagem `items('Aplicar_a_cada')?['IdMensagem']` e Status `complete`.

## Testar

Responda o e-mail de teste do Fluxo 1. Em até alguns minutos o item deve aparecer
como `Respondido` com a data em `RespondidoEm`.

> Para e-mails que não precisam de resposta, mude o `Status` manualmente para `Ignorado`.
