# ⚔️ Sorteio Arena

Sorteador de campeões de **League of Legends** para quem está completando o desafio do modo **Arena**: vencer uma partida com cada campeão do jogo.

A aplicação sorteia um campeão entre os que você ainda não venceu, trava a tela até você informar o resultado da partida e remove o campeão da lista quando a vitória é confirmada. Assim não é mais preciso controlar à mão quais campeões faltam.

🔗 **Acesse:** [kerchak26.github.io/sorteio-arena](https://kerchak26.github.io/sorteio-arena/)

<!-- Dica: adicione um print da aplicação aqui -->
<!-- ![Tela do Sorteio Arena](./screenshot.png) -->

---

## 📌 Sumário

- [Motivação](#-motivação)
- [Funcionalidades](#-funcionalidades)
- [Tecnologias](#-tecnologias)
- [Como funciona](#-como-funciona)
- [Estrutura de dados](#-estrutura-de-dados)
- [Como rodar localmente](#-como-rodar-localmente)
- [Deploy no GitHub Pages](#-deploy-no-github-pages)
- [Decisões técnicas](#-decisões-técnicas)
- [Próximos passos](#-próximos-passos)
- [Aviso legal](#%EF%B8%8F-aviso-legal)
- [Autor](#-autor)

---

## 💡 Motivação

No modo Arena do LoL existe o desafio de vencer (ficar em 1º lugar) com campeões diferentes. O título **Deus da Arena** é liberado ao chegar em 60. Eu controlava isso com uma lista manual: sorteava um campeão, jogava e, se ganhasse, apagava o nome da lista. Com mais de 170 campeões, esse processo era lento e fácil de errar.

O **Sorteio Arena** automatiza esse fluxo de ponta a ponta. Ele sorteia, registra o resultado, guarda o histórico e mostra o progresso.

---

## ✨ Funcionalidades

| Funcionalidade | Descrição |
|---|---|
| 🎲 **Sorteio animado** | Os retratos e nomes alternam com desaceleração progressiva até parar no campeão escolhido. |
| 🔒 **Trava pós-sorteio** | Depois do sorteio, a tela fica bloqueada até você responder *Ganhei*, *Não ganhei* ou *Pular*. O bloqueio continua mesmo se a página for recarregada. |
| ✅ **Remoção automática** | Ao confirmar a vitória, o campeão sai do sorteio e vai para a lista de vencidos. |
| ⏭️ **Pular sem jogar** | Para quando você não tem o campeão. Ele volta ao sorteio sem contar como derrota. |
| 🖼️ **Ícones oficiais** | Lista de campeões e retratos carregados direto da Riot (Data Dragon). Campeões novos aparecem sozinhos a cada patch. |
| 📊 **Estatísticas** | Partidas jogadas, vitórias, taxa de vitória e sequência atual. |
| 🏆 **Marco Deus da Arena** | Barra de progresso com o marco de 60 vitórias destacado e a contagem do que falta. |
| 🕓 **Histórico** | Últimas partidas com retrato, data, hora e resultado. |
| ↩️ **Desfazer** | Desfaz o último resultado em caso de clique errado. |
| 🔍 **Busca e filtros** | Abas *Faltam*, *Vencidos* e *Todos*, com busca que ignora acentos e apóstrofos. |
| ✍️ **Edição manual** | Clique em um campeão para marcar ou desmarcar a vitória. |
| 💾 **Backup** | Exporta e importa o progresso em `.json` para trocar de computador ou guardar uma cópia. |
| 📴 **Modo offline** | Se a Riot não responder, a aplicação usa a última lista salva ou uma lista embutida. |
| 📱 **Responsivo e acessível** | Funciona no celular, tem foco visível no teclado e respeita `prefers-reduced-motion`. |

---

## 🛠 Tecnologias

- **HTML5**
- **CSS3**: variáveis CSS (design tokens), Grid, Flexbox, `color-mix()`, `backdrop-filter`, animações
- **JavaScript (ES2017+)**: puro, sem frameworks e sem etapa de build
- **Fetch API** para consumir o [Data Dragon](https://developer.riotgames.com/docs/lol#data-dragon) da Riot Games
- **Web Storage API** (`localStorage`) para persistir os dados
- **Blob / File API** para exportar e importar o backup
- **Google Fonts**: Cinzel (títulos) e Barlow (texto)
- **GitHub Pages** para hospedagem

---

## ⚙️ Como funciona

```
┌──────────────┐     ┌───────────────────┐     ┌─────────────────────┐
│ Abrir página │ ──▶ │ Busca versão atual│ ──▶ │ Busca champion.json │
└──────────────┘     │  (versions.json)  │     │  (pt_BR + ícones)   │
                     └───────────────────┘     └──────────┬──────────┘
                                                          ▼
┌──────────────┐     ┌───────────────────┐     ┌─────────────────────┐
│   Sortear    │ ◀── │ Calcula restantes │ ◀── │  Lê progresso do    │
└──────┬───────┘     │ (todos − vencidos)│     │   localStorage      │
       ▼             └───────────────────┘     └─────────────────────┘
┌──────────────┐
│ Tela travada │──▶ Ganhei ──▶ adiciona aos vencidos + histórico
│  (pending)   │──▶ Não ganhei ──▶ só histórico
└──────────────┘──▶ Pular ──▶ só histórico (não conta como partida)
```

1. **Carregamento dos campeões:** a aplicação consulta `api/versions.json` para descobrir o patch mais recente e depois baixa `champion.json` em português, que traz o nome e o arquivo do ícone de cada campeão. A lista fica em cache para uso offline.
2. **Sorteio:** um campeão é escolhido com `Math.random()` entre os restantes. A animação dura cerca de 2,4 s, e o intervalo entre trocas cresce 7% a cada passo para dar o efeito de roleta desacelerando.
3. **Trava:** o campeão sorteado fica salvo no campo `pending`. Enquanto ele existir, o botão de sortear fica desabilitado e o modal de resultado continua aberto, inclusive depois de recarregar a página.
4. **Resultado:** a resposta é gravada no histórico. Se for vitória, o nome entra em `won` e deixa de participar dos sorteios.

---

## 🗂 Estrutura de dados

O progresso inteiro é um único objeto JSON salvo no `localStorage`, que é o mesmo formato do arquivo de backup:

```json
{
  "won": ["Ahri", "Garen", "Lux"],
  "pending": null,
  "history": [
    { "c": "Ahri",  "r": "win",  "t": 1760918400000 },
    { "c": "Yasuo", "r": "loss", "t": 1760922000000 },
    { "c": "Zed",   "r": "skip", "t": 1760925600000 }
  ]
}
```

| Campo | Tipo | Descrição |
|---|---|---|
| `won` | `string[]` | Campeões já vencidos |
| `pending` | `string \| null` | Campeão sorteado aguardando resultado (controla a trava) |
| `history` | `object[]` | Registro das partidas: `c` = campeão, `r` = resultado (`win` / `loss` / `skip`), `t` = timestamp |

Os campeões restantes não são guardados. Eles são calculados a cada renderização como **todos os campeões − `won`**. Com isso existe uma única fonte de verdade, e campeões lançados depois entram automaticamente no sorteio.

---

## 💻 Como rodar localmente

Não há dependências nem build. Basta clonar e abrir:

```bash
git clone https://github.com/Kerchak26/sorteio-arena.git
cd sorteio-arena
```

Abra o `index.html` no navegador. Se preferir um servidor local:

```bash
# Python
python -m http.server 8000

# ou Node
npx serve .
```

Depois acesse `http://localhost:8000`.

---

## 🚀 Deploy no GitHub Pages

1. Envie o `index.html` para a raiz do repositório.
2. Vá em **Settings → Pages**.
3. Em **Source**, escolha **Deploy from a branch**, depois a branch `main` e a pasta `/ (root)`.
4. Clique em **Save**. O site fica disponível em `https://<usuario>.github.io/<repositorio>/`.

---

## 🧠 Decisões técnicas

- **Sem framework:** a aplicação é pequena e tem um único estado, então JavaScript puro deixa o projeto leve, rápido e simples de hospedar como página estática.
- **Estado único e derivado:** só os vencidos são guardados, e o restante é calculado. Isso evita inconsistências entre duas listas.
- **Trava persistida:** o `pending` fica salvo junto com o progresso, então fechar a página não serve para "fugir" de um sorteio.
- **Dados vivos da Riot:** consumir o Data Dragon em vez de manter uma lista fixa elimina a manutenção a cada patch.
- **Tolerância a falhas:** a lista é carregada em camadas (Data Dragon → cache local → lista embutida), e um retrato que não carrega é trocado pelas iniciais do campeão.
- **Busca normalizada:** com `String.normalize("NFD")` e remoção de símbolos, buscar "kaisa" encontra *Kai'Sa* e "chogath" encontra *Cho'Gath*.
- **Segurança:** os nomes vindos da API são inseridos com `textContent` e nunca com `innerHTML`, o que evita injeção de HTML.
- **Confirmação sem `confirm()`:** o botão "Zerar progresso" pede um segundo clique em até 4 segundos, sem depender de diálogos nativos do navegador.

---

## 🔭 Próximos passos

- [ ] Filtro por classe (lutador, mago, atirador…) usando as `tags` do Data Dragon
- [ ] Sincronização entre dispositivos (Firebase ou Supabase)
- [ ] Gráfico de evolução de vitórias ao longo do tempo
- [ ] Integração com a Riot API para detectar vitórias automaticamente pelo histórico de partidas
- [ ] PWA instalável com service worker

---

## ⚖️ Aviso legal

*Sorteio Arena* não é endossado pela Riot Games e não reflete as opiniões ou visões da Riot Games nem de qualquer pessoa oficialmente envolvida na produção ou no gerenciamento de propriedades da Riot Games. Riot Games e todas as propriedades associadas são marcas comerciais ou marcas registradas da Riot Games, Inc. Os ícones dos campeões são carregados diretamente do Data Dragon oficial da Riot.

---

## 👤 Autor

**Kauã Queiroga Oliveira André**
Estudante de Análise e Desenvolvimento de Sistemas – Faculdade Impacta

[![LinkedIn](https://img.shields.io/badge/LinkedIn-kaua--queiroga-0A66C2?style=flat&logo=linkedin)](https://linkedin.com/in/kaua-queiroga/)
[![GitHub](https://img.shields.io/badge/GitHub-Kerchak26-181717?style=flat&logo=github)](https://github.com/Kerchak26)
