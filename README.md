# Gerenciador de E-mail — Termos em Evidência

Plataforma que monitora o e-mail corporativo (Outlook / Microsoft 365), identifica
mensagens que contêm **termos específicos** e as coloca **em evidência** numa lista
de pendências até serem respondidas.

Tudo roda **na nuvem do Microsoft 365**, sem instalar nada na máquina e usando
apenas conectores **padrão** (não premium) do Power Automate.

## Arquitetura

```
 Caixa de Entrada (Outlook)                 Itens Enviados (Outlook)
          │                                          │
          ▼                                          ▼
 ┌──────────────────────────┐            ┌──────────────────────────┐
 │ Fluxo 1 — Captura        │            │ Fluxo 2 — Respondido     │
 │ • lê assunto + corpo     │            │ • pega o IdConversa      │
 │ • compara c/ lista Termos│            │ • marca itens pendentes  │
 │ • cria item + sinaliza   │            │   da conversa como       │
 │   o e-mail no Outlook    │            │   "Respondido"           │
 └────────────┬─────────────┘            └────────────┬─────────────┘
              ▼                                       ▼
      ┌──────────────────────────────────────────────────────┐
      │ Microsoft Lists / SharePoint                         │
      │  • "Termos"              → o que monitorar           │
      │  • "EmailsEmEvidencia"   → painel de pendências      │
      │    (cores por prioridade, horas em aberto, link      │
      │     direto para abrir o e-mail no Outlook Web)       │
      └──────────────────────────────────────────────────────┘
```

- **Termos** são cadastrados numa lista — dá para adicionar/remover sem mexer no fluxo.
- Cada termo tem **prioridade** (1 = Alta, 2 = Média, 3 = Baixa); o e-mail herda a mais alta encontrada.
- Quando você responde (qualquer mensagem enviada na mesma conversa), o item sai de "Pendente" automaticamente.

## Passo a passo

1. [Criar as listas no SharePoint/Microsoft Lists](docs/01-listas.md)
2. [Fluxo 1 — Captura de e-mails](docs/02-fluxo-captura.md)
3. [Fluxo 2 — Marcar como respondido](docs/03-fluxo-respondido.md)
4. [Aplicar a formatação visual do painel](docs/04-painel.md)
5. [Alerta no Teams criado pelo Copilot (ex.: termo "Viagem")](docs/05-copilot-alerta-teams.md)
6. [Varredura de e-mails não lidos já existentes](docs/06-varredura-nao-lidos.md)

Arquivos de formatação JSON prontos para colar ficam em [`formatacao/`](formatacao/).

## Conectores usados (todos padrão)

| Conector | Uso |
|---|---|
| Office 365 Outlook | gatilho de novo e-mail, sinalizar e-mail |
| SharePoint | ler termos, criar/atualizar itens |
| Conversão de Conteúdo (Content Conversion) | HTML → texto |
| Microsoft Teams *(opcional)* | alerta para prioridade Alta |

> Antes de ativar, confirme com o TI se há política de DLP que restrinja algum desses
> conectores. Nenhum dado sai do ambiente Microsoft 365 da empresa.

## Próximos passos (opcionais)

- **Sugestão de resposta com IA**: usar o conector Microsoft 365 no claude.ai (se o
  administrador liberar) para ler os itens pendentes e redigir rascunhos.
- **Carga retroativa**: fluxo manual com "Obter e-mails (V3)" para processar os últimos dias.
- **Painel web próprio**: se houver licença premium (conector HTTP), o fluxo pode enviar
  os dados para uma aplicação externa.
