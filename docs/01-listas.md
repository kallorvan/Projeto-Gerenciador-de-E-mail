# 1. Criar as listas

Você pode usar um **site do SharePoint** da sua equipe ou o **Microsoft Lists** pessoal
(fica no seu OneDrive). Acesse em https://www.office.com/launch/lists.

> **Dica importante sobre nomes:** crie cada coluna primeiro com o nome **sem acento e
> sem espaço** exatamente como na tabela (ex.: `RecebidoEm`). Isso fixa o *nome interno*,
> que é o que o Power Automate usa nos filtros. Depois, se quiser, renomeie o nome de
> exibição para algo como "Recebido em".

Se usar o Lists pessoal, no Power Automate o "Endereço do Site" será algo como
`https://<empresa>-my.sharepoint.com/personal/<seu_usuario>_<empresa>_com` — escolha
"Inserir valor personalizado" e cole esse endereço.

## Lista `Termos`

| Coluna | Tipo | Observação |
|---|---|---|
| `Title` (Título) | já existe | o termo a buscar, ex.: `contrato`, `urgente`, `nota fiscal` |
| `Prioridade` | Número (0 casas decimais), padrão `2` | 1 = Alta, 2 = Média, 3 = Baixa |
| `Ativo` | Sim/Não, padrão **Sim** | desmarque para pausar um termo |

Exemplos de itens:

| Title | Prioridade | Ativo |
|---|---|---|
| urgente | 1 | Sim |
| prazo | 1 | Sim |
| contrato | 2 | Sim |
| nota fiscal | 2 | Sim |
| reunião | 3 | Sim |
| reuniao | 3 | Sim |

> A busca **não ignora acentos**: cadastre variantes (`reunião` e `reuniao`) quando fizer
> sentido. Maiúsculas/minúsculas são ignoradas. Termos curtos (ex.: `nf`) podem casar
> dentro de outras palavras — prefira termos com 4+ letras ou com espaço (`" nf "`).

## Lista `EmailsEmEvidencia`

| Coluna | Tipo | Observação |
|---|---|---|
| `Title` (Título) | já existe | assunto do e-mail |
| `Remetente` | Linha única de texto | |
| `RecebidoEm` | Data e hora (incluir hora) | |
| `Termos` | Linha única de texto | termos encontrados, separados por vírgula |
| `Prioridade` | Número (0 casas decimais) | 1, 2 ou 3 |
| `Status` | Escolha: `Pendente`, `Respondido`, `Ignorado` — padrão `Pendente` | |
| `RespondidoEm` | Data e hora (incluir hora) | |
| `Trecho` | Várias linhas de texto (**texto sem formatação**) | início do corpo |
| `LinkOutlook` | Várias linhas de texto (**texto sem formatação**) | link para abrir o e-mail |
| `IdConversa` | Linha única de texto | **precisa ser linha única** (usado em filtro) |
| `IdMensagem` | Várias linhas de texto (texto sem formatação) | |

Em *Configurações da lista → Colunas indexadas*, crie índices para `IdConversa` e
`Status` (melhora o desempenho do Fluxo 2 quando a lista crescer).
