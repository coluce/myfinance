# MyFinance

App de controle financeiro pessoal para **Linux**, distribuído como **AppImage**. Os dados
ficam apenas no seu computador: não há conta, servidor próprio, sincronização ou telemetria.
A única consulta de rede é a verificação de atualizações sob demanda.

- Painel do mês com saldo, receitas, despesas, tendência e gasto por categoria.
- Movimentações e orçamentos mensais por categoria.
- Importação e exportação CSV.
- Backup completo em JSON: manual no navegador/desktop e automático no desktop, em uma pasta escolhida (inclusive uma pasta sincronizada com a nuvem).
- Tour de boas-vindas na primeira abertura, explicando as telas e o backup, com opção de reabrir em Ajustes. Após limpar os dados locais, o tour volta a aparecer.
- Aba **Sobre** com versão, plataforma, armazenamento, links do site/repositório e verificação de atualizações sob demanda.
- Tema claro e escuro, seguindo (e memorizando) a sua escolha.
- Sons discretos na navegação e nas ações principais, sintetizados, ligados por padrão e com opção de desligar em Ajustes.
- Funciona offline depois de baixado; só a verificação de atualizações usa a internet, quando você pede.

## Baixar

A versão pronta está em [Releases](https://github.com/coluce/financeiro.github.io/releases).

```bash
chmod +x MyFinance-1.1.0-x86_64.AppImage
./MyFinance-1.1.0-x86_64.AppImage
```

Requisitos: Linux x86_64 com glibc 2.39+. Em algumas distribuições é preciso ter
`libfuse2` (`libfuse2t64` no Ubuntu 24.04+); sem ele, use
`./MyFinance-1.1.0-x86_64.AppImage --appimage-extract-and-run`.

### Ícone na barra de tarefas (GNOME/Wayland)

Abrir o AppImage diretamente não registra um atalho no sistema. No GNOME, a janela fica
sem associação e mostra o ícone genérico na barra de tarefas (o ícone do explorador de
arquivos vem embutido no AppImage e não muda isso). A partir do repositório, rode uma vez:

```bash
./app/desktop/install-desktop-entry.sh /caminho/para/MyFinance-1.1.0-x86_64.AppImage
```

Sem argumento, o script usa o AppImage mais recente em `app/desktop/release/`. Ele cria
`~/.local/share/applications/myfinance.desktop`, instala os ícones em
`~/.local/share/icons/hicolor/` e não usa `sudo`. Se mover o AppImage, rode de novo;
para desfazer, use `--remove`. Alternativa: usar o AppImageLauncher.

## Desenvolvimento

O app é feito em React Native Web (Expo) e o AppImage empacota esse build dentro de um shell
Electron. Pré-requisitos: Node 20+, pnpm 9 e ~2 GB de RAM livre.

```bash
cd app
pnpm install

pnpm dev          # Expo web em http://localhost:8081 (desenvolvimento)
pnpm check        # tsc --noEmit
pnpm lint         # expo lint
pnpm test         # testes de versão/onboarding (Node 24; type stripping)
```

Para gerar o AppImage:

```bash
./scripts/build-appimage.sh
# saída: dist/MyFinance-1.1.0-x86_64.AppImage
# e app/desktop/release/MyFinance-1.1.0-x86_64.AppImage
```

O script gera o build web, sincroniza o renderer, empacota com o
`electron-builder` e copia o AppImage para `dist/` (ignorado pelo Git). Para
escolher outro destino, use `--out PASTA`.

Passos manuais equivalentes:

```bash
cd app
pnpm build:web        # expo export -p web --output-dir web-build
pnpm desktop:sync     # copia o build para desktop/renderer
npm --prefix desktop install
npm --prefix desktop run pack
```

Guia detalhado, portas, problemas comuns e decisões de arquitetura:
[`doc/execucao.md`](doc/execucao.md) e
[`doc/analises/2026-11-07-appimage-orcar.md`](doc/analises/2026-11-07-appimage-orcar.md).

## Estrutura do repositório

```text
.
├── index.html          # landing page publicada no GitHub Pages
├── img/                # ícone, favicon e screenshot usados na landing
├── app/                # app Expo (React Native Web) + shell desktop
│   ├── app/            # rotas (abas: visão geral, movimentações, orçamentos, ajustes, sobre)
│   ├── components/     # componentes de UI
│   ├── lib/            # estado, persistência local (localStorage), tema
│   ├── desktop/        # Electron + electron-builder (gera o AppImage)
│   └── assets/         # ícones
├── scripts/            # script único de build do AppImage
├── doc/                # documentação obrigatória (ver AGENTS.md)
└── AGENTS.md           # regras para pessoas e agentes de IA
```

## Dados e privacidade

- Os dados ficam em `~/.config/myfinance/` (armazenamento local do Chromium), em uma base
  LevelDB — não em um arquivo JSON simples. Na primeira abertura após a renomeação, o
  perfil antigo de `~/.config/orcar/` é migrado automaticamente; feche o app antigo antes.
- A chave interna `orcar-budget-v1` e a origem `app://orcar` são mantidas como legado para
  preservar os dados; elas não aparecem na interface.
- **Não são criptografados**: quem tiver acesso à sua pasta de usuário consegue lê-los.
- **Backup:** exportação/importação manual de um JSON completo em Ajustes; no desktop, é
  possível escolher uma pasta e gravar um backup automático por dia (retenção de 7, 30 ou
  90 arquivos). O CSV continua disponível para planilhas. O app não envia nada para a
  nuvem: se a pasta escolhida for sincronizada, quem envia é o cliente de sincronização.
- O backup também é texto puro; use uma pasta segura.
- O app não faz chamadas automáticas para servidores. A verificação de atualizações, na aba
  Sobre, consulta a API pública do GitHub somente quando você clica; nenhum dado financeiro
  é enviado (o GitHub recebe a requisição como qualquer acesso à internet).

## Site (GitHub Pages)

A raiz do repositório é publicada como página estática: `index.html` descreve o projeto e
aponta para o Release mais recente. Não há build nem JavaScript nessa página.

## Segurança

- **Análise de segurança (2026-11-07):**
  [`doc/seguranca/2026-11-07-analise-de-seguranca.md`](doc/seguranca/2026-11-07-analise-de-seguranca.md)
- **Verificação de atualizações (2026-11-10):**
  [`doc/seguranca/2026-11-10-verificacao-atualizacoes.md`](doc/seguranca/2026-11-10-verificacao-atualizacoes.md)
- Nenhum código injetado, backdoor ou exfiltração foi encontrado.
- O backend herdado do template (Express/tRPC/OAuth/MySQL) foi **removido** do repositório;
  os achados que dependiam dele (A1, A3, B3, B4) deixaram de existir com a remoção.
- Regra permanente: nunca commitar segredos (`.env`, chaves, tokens).

## Documentação

- Regra obrigatória: **toda mudança precisa de registro em [`doc/`](doc/README.md)** —
  veja [`AGENTS.md`](AGENTS.md).
- Índice geral: [`doc/README.md`](doc/README.md).
- Registros de mudança: [`doc/mudancas/`](doc/mudancas/README.md).

## Contribuindo

1. Leia o `AGENTS.md`.
2. Apresente o plano e aguarde aprovação (Regra número 2).
3. Faça a alteração e crie o registro em `doc/` na mesma entrega.
4. Confira que nenhum segredo entrou no diff.

## Licença

Projeto pessoal, sem licença definida. Todos os direitos reservados ao autor.
