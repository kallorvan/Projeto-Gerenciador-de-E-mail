# 5. Alerta no Teams criado pelo Copilot do Power Automate

Objetivo: sempre que chegar um e-mail com a palavra **"Viagem"** no **assunto ou no corpo**,
receber no Teams uma mensagem com o e-mail que disparou o alerta.

Em https://make.powerautomate.com → **Página inicial** → caixa do Copilot
("Descreva o que você quer automatizar"), cole o prompt principal.

## Prompt principal

```
Crie um fluxo de nuvem automatizado com o nome "Alerta Teams - Viagem".

Gatilho: Office 365 Outlook "Quando um novo email chegar (V3)", na pasta Caixa de Entrada,
sem incluir anexos e sem filtro de assunto no gatilho.

Depois do gatilho, adicione a ação "Html para texto" (Conversão de Conteúdo) usando o
Corpo do e-mail, e renomeie essa ação para "CorpoTexto".

Em seguida, adicione uma Condição que seja verdadeira quando o Assunto OU o texto de
"CorpoTexto" contiver a palavra "viagem" ou "viagens", ignorando maiúsculas e minúsculas.
Use esta expressão no modo avançado da condição:
@or(contains(toLower(triggerOutputs()?['body/subject']), 'viagem'), contains(toLower(triggerOutputs()?['body/subject']), 'viagens'), contains(toLower(body('CorpoTexto')), 'viagem'), contains(toLower(body('CorpoTexto')), 'viagens'))

No ramo "Sim", adicione a ação do Microsoft Teams "Postar mensagem em um chat ou canal",
postando como "Flow bot" em "Chat com o bot de Fluxo", com o destinatário sendo o meu
próprio e-mail. A mensagem deve conter:
- um título "📩 E-mail com o termo Viagem"
- Remetente (De)
- Assunto
- Data de recebimento
- Destinatários (Para) e Cc
- O texto do e-mail vindo de "CorpoTexto", limitado aos primeiros 4000 caracteres
- Um link "Abrir no Outlook" montado com a expressão:
  concat('https://outlook.office365.com/owa/?ItemID=', encodeUriComponent(triggerOutputs()?['body/id']), '&exvsurl=1&viewmodel=ReadMessageItem')

No ramo "Não", não faça nada.
```

## Prompts de ajuste (se o Copilot errar alguma parte)

Use no painel do Copilot **dentro do editor do fluxo**, um de cada vez:

- `Troque o gatilho para "Quando um novo email chegar (V3)" do Office 365 Outlook, pasta Caixa de Entrada.`
- `Na mensagem do Teams, limite o texto do e-mail com a expressão take(body('CorpoTexto'), 4000).`
- `Mude a ação do Teams para postar como Flow bot no Chat com o bot de Fluxo, com destinatário <seu e-mail>.`
- `Adicione também a ação "Sinalizar email (V2)" no ramo Sim, com o Id da mensagem do gatilho.`
- `Adicione mais um termo à condição: "passagem".`

## Conferir antes de salvar

O Copilot às vezes monta a lógica um pouco diferente. Verifique:

1. **Gatilho** é *Quando um novo email chegar (V3)* (e não "...for sinalizado" ou "...mencionar mim").
2. A **Condição** olha **Assunto e Corpo**, usa `toLower(...)` e combina com **OU** (`or`), não **E**.
3. A ação **Html para texto** se chama exatamente `CorpoTexto` (senão a expressão quebra).
4. A ação do Teams está no ramo **Sim** e o destinatário é você.
5. O texto do corpo está limitado (`take(..., 4000)`): mensagens do Teams têm limite de
   tamanho (~28 KB) e e-mails longos fariam a ação falhar.

## Testar

Salve → **Testar → Manualmente** → envie para você mesmo um e-mail com assunto
"Teste viagem". A mensagem deve chegar no Teams, no chat com o **Power Automate / Workflows**.

## Observações

- A busca é por trecho de texto: "viagem" também casa com "viagem-teste", e não casa com
  "viajar". Inclua variações na condição se precisar.
- Para vários termos ou prioridades, prefira a lista `Termos` do [Fluxo 1](02-fluxo-captura.md)
  e coloque a ação do Teams no ramo Sim dele.
