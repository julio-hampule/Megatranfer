# MegaTransfer

App Android nativo (Kotlin + Jetpack Compose) para automatizar a transferência
de megabytes entre números via USSD (`*162#`), com histórico offline, limites
diários por cartão e notificações.

## Como compilar (sem Termux, projeto Android Studio normal)

1. Baixe e instale o **Android Studio** (gratuito): https://developer.android.com/studio
2. Abra o Android Studio → **Open** → selecione a pasta `MegaTransfer` (esta pasta).
3. O Android Studio vai baixar o Gradle e as dependências sozinho (precisa de internet
   na primeira vez). Aguarde o "Gradle Sync" terminar.
4. Conecte seu celular via USB com **Depuração USB** ativada (ou use um emulador),
   e clique em **Run ▶**.
5. Para gerar o `.apk` final: **Build → Build Bundle(s)/APK(s) → Build APK(s)**.
   O arquivo fica em `app/build/outputs/apk/debug/app-debug.apk`, pronto para
   instalar em qualquer aparelho (ative "Fontes desconhecidas" no Android).

Não é necessário Termux nem servidor: é um app Android comum.

## Estrutura do projeto

```
app/src/main/java/com/megatransfer/app/
├── MainActivity.kt              # navegação por abas (Compose)
├── MyApplication.kt             # inicialização (banco, notificações)
├── data/                        # Room: CardEntity, TransferEntity, DAOs
├── repository/                  # regras de negócio (limites 100–10240 MB, 10/dia)
├── service/UssdAccessibilityService.kt  # automação do menu USSD
├── util/
│   ├── OperatorDetector.kt      # detecta Vodacom/Movitel/Tmcel pelo prefixo
│   ├── UssdDialer.kt            # disca *162# e inicia a sessão automatizada
│   └── NotificationHelper.kt    # notificações de limite/resultado
├── viewmodel/TransferViewModel.kt
└── ui/                          # telas Compose: Transferir, Cartões, Histórico, Ajustes
```

## Permissões que o usuário precisa ativar manualmente

O app tem uma aba **Ajustes** que verifica e guia cada uma:

1. **Serviço de Acessibilidade** — essencial; permite ler e preencher a caixa
   de diálogo do USSD automaticamente.
2. **Permissão de Chamada (CALL_PHONE)** — para discar `*162#`.
3. **Notificações** — para os avisos de limite atingido.
4. **Otimização de bateria desativada** — evita que o Android mate o app
   durante a automação.

## ⚠️ Limitação técnica importante

A automação do USSD funciona lendo o texto exibido na caixa de diálogo nativa
do discador e comparando com palavras-chave (`AGUARDANDO_MENU_PRINCIPAL`,
`AGUARDANDO_SUBMENU`, etc. em `UssdAccessibilityService.kt`). Isso é
**dependente da redação exata usada pela operadora** no momento em que você
testar. Se a Vodacom/Movitel/Tmcel mudar o texto do menu, ou se o número
`*162#` estiver diferente na prática, as palavras-chave em
`UssdAccessibilityService.onAccessibilityEvent()` precisarão ser ajustadas
— recomendo testar manualmente o fluxo primeiro num número de teste e ajustar
as strings de detecção conforme o texto real que aparecer na tela.

Fabricantes como **Xiaomi, Samsung e Huawei** têm proteções extras de bateria/
segundo-plano que podem interromper serviços de acessibilidade; a aba Ajustes
tem uma nota sobre isso.

## Regras de negócio já implementadas

- Detecção automática de operadora pelo prefixo (84/85 Vodacom, 86/87 Movitel, 82/83 Tmcel)
- Mínimo 100 MB / máximo 10240 MB por transferência
- Máximo 10 transferências por dia, por cartão de origem
- Histórico completo salvo em SQLite local (Room) — funciona 100% offline
- Notificações locais quando um cartão atinge o limite diário e quando uma
  transferência termina (sucesso ou falha)
