# Lista_de_Tarefas

Aplicativo Android para organizar tarefas por dia do mês. Funciona sem internet e guarda tudo no próprio aparelho: não existe servidor, API ou banco de dados na nuvem.

Projeto Integrador Transdisciplinar  
Marcelo dos Santos Capre — RGM 32984561

---

## O problema

Compromissos e atividades do dia a dia se perdem quando não há um registro simples, organizado por data. Anotações espalhadas em papel, bloco de notas e mensagens levam ao esquecimento de prazos e à sensação de desorganização.

## A solução

Um aplicativo de celular que mostra as tarefas de um dia por vez, com um seletor de dias no topo e indicadores do mês. A pessoa se cadastra no próprio aparelho e começa a usar em segundos, sem depender de conexão.

## Funcionalidades

**Conta e acesso**

- Cadastro com nome, e-mail e senha (mínimo de 6 caracteres).
- Login e logout. A sessão continua ativa ao fechar e reabrir o aplicativo.
- Aviso em destaque quando o e-mail já está cadastrado naquele aparelho.
- Cada conta enxerga apenas as próprias tarefas.

**Tarefas**

- Cadastro com título, dia, categoria e prioridade, mais uma anotação opcional.
- Categorias: Trabalho, Pessoal, Estudos, Saúde e Geral.
- Prioridades: Baixa, Média e Alta.
- Atalhos "Hoje" e "Amanhã" na escolha da data.
- Concluir, editar o título e excluir tarefas.
- Botão para limpar de uma vez as tarefas concluídas do dia.

**Visualização**

- Faixa horizontal com os dias do mês, marcando o dia selecionado.
- Aba "Visão por Dia" com barra de progresso e filtros Todas, Pendentes e Concluídas.
- Aba "Todas do Mês" com as tarefas agrupadas por data.
- Indicadores do mês: total, concluídas, pendentes e taxa de sucesso.

**Personalização**

- Avatar com 12 ícones prontos ou uma foto da galeria/câmera do celular.
- A foto é reduzida e recortada em quadrado de 256 pixels antes de ser salva.
- Tema escuro em toda a interface, com fontes embutidas no aplicativo.

## Como os dados são guardados

Toda a persistência está em `src/utils/localStore.ts`, do projeto da interface, e usa o armazenamento local do WebView (`localStorage`), sob a chave `lista_tarefas_dados_v1`. No Android, o Cordova mantém esse armazenamento no diretório privado do aplicativo.

As senhas nunca são gravadas em texto puro: são derivadas com PBKDF2 (SHA-256, 100 mil iterações) usando a Web Crypto API. Se o WebView não oferecer essa API, o aplicativo cai em um hash mais simples, e o algoritmo usado fica registrado junto do hash para que contas antigas continuem validando.

Esse modelo tem consequências que fazem parte do projeto:

- Os dados sobrevivem a fechar e reabrir o aplicativo.
- Desinstalar o aplicativo apaga tudo, e será preciso cadastrar novamente.
- Cada aparelho tem seus próprios usuários, sem sincronização entre eles.
- A verificação de e-mail já cadastrado vale para aquele aparelho.

## Tecnologias

| Camada | Tecnologia |
| --- | --- |
| Interface | React 19, TypeScript, Tailwind CSS 4 |
| Build | Vite 6, pnpm |
| Rotas | React Router 7, em modo `HashRouter` |
| Ícones e animação | lucide-react, motion |
| Empacotamento | Apache Cordova com cordova-android 15 |
| Persistência | `localStorage` do WebView |
| Testes | Playwright, em roteiro próprio |

O `HashRouter` é obrigatório aqui: no Cordova a página abre por `file://`, onde rotas por caminho não funcionam. As fontes Plus Jakarta Sans e Outfit são empacotadas via `@fontsource`, sem chamadas ao Google Fonts, para que o aplicativo não dependa de rede.

## Estrutura do repositório

```
config.xml              Configuração do Cordova: ícones, splash e permissões
package.json            Dependências do Cordova
res/mipmap-*/           Ícones do aplicativo, em todas as densidades
res/splash/             Imagem da tela de abertura
www/                    Interface já compilada, que vai dentro do APK
```

A interface é desenvolvida em um projeto React separado e o resultado do build é copiado para `www/`.

## Permissões no Android

Declaradas em `config.xml` e usadas apenas na escolha do avatar:

- `READ_MEDIA_IMAGES` — abrir a galeria no Android 13 e acima.
- `READ_EXTERNAL_STORAGE` (até a API 32) — mesma finalidade em versões anteriores.
- `CAMERA` — tirar a foto na hora.

Não há permissão de internet, porque o aplicativo não acessa a rede.

## Como gerar o APK

Pré-requisitos: Node.js, pnpm, JDK 17 e o Android SDK configurados.

```bash
# 1. No projeto da interface, gerar o build de produção
pnpm install
pnpm build

# 2. Copiar o conteúdo de dist/ para a pasta www/ deste repositório

# 3. Neste repositório, gerar o APK
pnpm install
npx cordova platform add android   # apenas na primeira vez
npx cordova build android
```

O pacote fica em `platforms/android/app/build/outputs/apk/debug/app-debug.apk`.

Para instalar em um aparelho conectado por USB, com a depuração ativada:

```bash
npx cordova run android
```

## Tela de abertura

A imagem da splash fica em `res/splash/splashscreen.png`, em 288 × 288 pixels, exibida sobre fundo preto. As preferências correspondentes precisam ficar na raiz do `config.xml`, e não dentro de `<platform name="android">`: o `prepare` do Cordova lê apenas as preferências da raiz e, caso contrário, aplica a splash padrão.

## Testes

O projeto da interface tem um roteiro automatizado em `scripts/e2e-check.mjs`, executado com Playwright em uma janela do tamanho de um celular:

```bash
pnpm test:e2e
```

Ele cobre cadastro, aviso de e-mail duplicado, lista vazia para conta nova, criação e conclusão de tarefas, troca de avatar por ícone e por foto, cores do tema e ausência de qualquer requisição externa.

Além disso, cinco pessoas testaram o aplicativo instalado no celular, e o resultado está registrado na documentação da disciplina.

## Limitações conhecidas

- Não há sincronização entre aparelhos: os dados são locais.
- Desinstalar o aplicativo apaga as contas e as tarefas.
- Não existe recuperação de senha esquecida.
- Não há lembrete por notificação nem tarefa que se repete automaticamente.
- Testado apenas em celular Android; não foi validado em tablet.

## Autor

Marcelo dos Santos Capre — RGM 32984561
