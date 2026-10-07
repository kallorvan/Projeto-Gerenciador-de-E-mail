# 4. Painel visual na lista `EmailsEmEvidencia`

## Modo de exibição "Pendentes"

1. Na lista, menu de modos de exibição → **Criar novo modo de exibição** → Lista → nome `Pendentes`.
2. **Filtro**: `Status` é igual a `Pendente`.
3. **Classificar**: `Prioridade` crescente, depois `RecebidoEm` crescente (mais antigos primeiro).
4. Colunas visíveis sugeridas: Prioridade, Título, Remetente, Termos, RecebidoEm, LinkOutlook, Status.
5. Defina como modo de exibição padrão, se quiser.

## Formatação

Abra o menu de modos de exibição → **Formatar o modo de exibição atual** → **Opções avançadas**
e cole o conteúdo de [`formatacao/linhas.json`](../formatacao/linhas.json):

- 🔴 fundo vermelho: pendente de prioridade Alta
- 🟠 fundo laranja: pendente de prioridade Média
- 🟢 fundo verde: respondido

Para cada coluna abaixo: clique no cabeçalho → **Configurações de coluna** → **Formatar esta
coluna** → **Opções avançadas** e cole o JSON correspondente:

| Coluna | Arquivo | Resultado |
|---|---|---|
| Prioridade | [`formatacao/coluna-prioridade.json`](../formatacao/coluna-prioridade.json) | selo "🔴 Alta / 🟠 Média / 🟢 Baixa" |
| RecebidoEm | [`formatacao/coluna-recebido.json`](../formatacao/coluna-recebido.json) | data + horas em aberto (vermelho após 24 h se pendente) |
| LinkOutlook | [`formatacao/coluna-link.json`](../formatacao/coluna-link.json) | botão "Abrir no Outlook" |
| Status | [`formatacao/coluna-status.json`](../formatacao/coluna-status.json) | selo colorido |

## Acesso rápido

- Fixe a lista no Teams (canal ou chat → **+** → *Lists*) ou adicione aos favoritos do navegador.
- No app Microsoft Lists do celular a lista também aparece, com os mesmos filtros.
