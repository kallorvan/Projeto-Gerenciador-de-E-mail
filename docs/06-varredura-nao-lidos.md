# 6. Varredura de e-mails não lidos (fluxo manual)

O alerta do [guia 5](05-copilot-alerta-teams.md) só reage a e-mails **novos**. Para procurar o
termo nos e-mails **não lidos que já estão** na Caixa de Entrada, crie um fluxo manual que manda
**uma única mensagem no Teams** com a lista do que encontrou.

> Sobre cópia (Cc): o gatilho "Quando um novo email é recebido (V3)" dispara para todo e-mail que
> chega na pasta, seja você Para, Cc ou Cco — desde que os filtros Para/Cc do gatilho estejam vazios.
> Se uma regra do Outlook move e-mails em cópia para outra pasta, aponte o gatilho para essa pasta
> (ou duplique o fluxo).

Criar → **Fluxo da nuvem instantâneo** → nome `Varredura - Viagem nao lidos` →
gatilho **Disparar um fluxo manualmente**.

| # | Ação (nome) | Configuração |
|---|---|---|
| 1 | Office 365 Outlook: *Obter emails (V3)* → `BuscarEmails` | Pasta: Caixa de Entrada · Buscar Somente Mensagens Não Lidas: **Sim** · Incluir Anexos: Não · Superior (Top): `250` |
| 2 | *Filtrar matriz* → `ComViagem` | De: `body('BuscarEmails')?['value']` · modo avançado: `@or(contains(toLower(item()?['subject']), 'viage'), contains(toLower(item()?['body']), 'viage'))` |
| 3 | *Selecionar* → `Linhas` | De: `body('ComViagem')` · Mapear (modo texto), expressão abaixo |
| 4 | Teams: *Postar mensagem em um chat ou canal* | Bot do Flow · Conversar com o bot do Flow · você · mensagem abaixo |

Expressão do **Mapear** (`Linhas`):
```
concat('• ', formatDateTime(convertFromUtc(item()?['receivedDateTime'], 'E. South America Standard Time'), 'dd/MM HH:mm'), ' — ', item()?['from'], ' — <b>', item()?['subject'], '</b> — <a href="https://outlook.office365.com/owa/?ItemID=', encodeUriComponent(item()?['id']), '&exvsurl=1&viewmodel=ReadMessageItem">Abrir no Outlook</a>')
```

Mensagem do Teams:
```
📋 Não lidos com o termo Viagem: [fx] length(body('ComViagem'))
[fx] join(take(body('Linhas'), 40), '<br>')
```

- `take(..., 40)` evita estourar o limite de tamanho da mensagem do Teams (~28 KB). Se houver
  mais de 40, leia/marque como lidos e rode de novo.
- `Superior` define quantos não lidos (os mais recentes) são verificados. Se der erro, reduza.
