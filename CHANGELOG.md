# Changelog — Dante's Inferno (Hell's Gate Recomp)

Todas as alterações notáveis neste projeto serão documentadas neste arquivo.

O formato é baseado em [Keep a Changelog](https://keepachangelog.com/pt-BR/1.0.0/) e este projeto segue o versionamento semântico.

## [0.7.2-beta-hotfix] — 2026-09-16

Esta versão incorpora as atualizações do upstream oficial (`v0.7.0-beta`, `v0.7.1-beta` e `v0.7.2-beta-hotfix`), trazendo **cache persistente de shaders SPIR-V pré-aquecidos**, **correção cirúrgica de Ultrawide Hor+**, **sincronização de VSync com vblank** e novo HUD de monitoramento de desempenho.

### 🚀 Destaques da Versão
* **Zero Shader Stutter (SPIR-V Vulkan Cache)**: Inclusão do pacote de shaders pré-compilados (`shader_cache/`), semeado automaticamente no arranque em `SeedShaderStorage()`, eliminando travamentos de compilação gráfica durante a gameplay.
* **Correção Definitiva de Proporção Ultrawide**: Novo gancho em `0x824D6B90` (`f0`) que intercepta o registrador antes do armazenamento da razão de aspecto na memória global do jogo, garantindo FOV, projeção e interface 21:9 Hor+ sem distorção.
* **Pacing de VSync e Novo Overlay de FPS**: Pacing sincronizado ao vblank do monitor e overlay ImGui redesenhado com cálculo por janela de 250ms e exibição da taxa em Hz do vblank.
* **Preservação de Todas as Blindagens Linux**: Mantida a remoção de entrypoints TU2 inválidos no manifesto, tolerância a mapeamentos no runtime, fallback dinâmico para Vulkan nativo sem exigência de `DiligentCore` e compatibilidade C++23 no Clang 18.

---

## [0.6.6-beta] — 2026-09-16

Esta versão sincroniza o porte com o upstream oficial (`v0.6.6-beta`), adicionando **suporte nativo automático a DLCs**, blindagens essenciais contra crashes de boot e melhorias nos scripts de empacotamento e instalação.

### 🚀 Destaques da Versão
* **Auto-Instalação e Montagem de DLCs**: O motor detecta e instala pacotes STFS de DLCs colocados na pasta `dlc/` automaticamente em `OnPostSetup`.
* **Resolução de Crash no Boot (`C0000001`)**: Removidos 7 entrypoints de Title Update (TU2) que estavam fora da faixa de endereços do `default.xex` base, e blindada a rejeição de stubs no ReXGlue SDK para emitir `REXSYS_WARN` em vez de abortar o jogo.
* **Instalador Otimizado sem Duplicação**: A criação de atalhos pelo script do jogador agora referencia diretamente o diretório do usuário, sem copiar arquivos desnecessariamente para `~/.local/share`.
* **Compatibilidade Clang 18 / C++23 e Vulkan**: Fallback limpo no CMake para o backend nativo Vulkan (`rexgpu-xenos.so`) e correções em cabeçalhos C++23 (`<expected>`).

---

## [0.5.0-alpha] — 2026-09-07

Esta versão marca a **conclusão da migração para Linux 100% Nativo (ELF / Vulkan)** a partir da base `v0.3.2-alpha`, eliminando qualquer dependência de Wine, Proton, emuladores ou scripts Windows PowerShell, e introduzindo uma nova arquitetura para o instalador e distribuição.

### 🚀 Destaques da Versão
* **Migração 100% Nativa Linux**: Executável ELF compilado antecipadamente (AOT) via ReXGlue SDK com backend Vulkan puro (`librexgpu-xenos.so`).
* **Novo Launcher Nativo em C++23 / Qt6**: Substitui o antigo launcher Windows .NET/WPF, com botão `Import ISO` integrado, seleção de resolução e gerenciamento de saves.
* **Separação Arquitetural do Instalador**: Divisão clara entre o **Hub do Desenvolvedor** (build e empacotamento) e o **Ponto de Entrada do Jogador** (configuração de drivers, atalhos e inicialização).
* **Pacote Distribuível Autocontido**: Geração automatizada de `DantesInferno-Linux-v0.5.0-alpha.tar.gz` contendo todos os binários, bibliotecas de runtime, utilitários e documentação.

---

### ⚙️ Arquitetura e Scripts

#### Adicionado
* **Hub do Desenvolvedor (`installer/linux/install.sh`)**:
  - Menu para desenvolvedores com 5 opções: `1 - Setup`, `2 - Build`, `3 - Package`, `4 - Clean`, `5 - Exit`.
  - Opção `4 - Clean` para limpar cache de compilação, objetos `.o` (`out/`), código C++ gerado (`generated/default/`) e builds intermediários liberando ~1 GB em disco, com opção integrada de apagar também os dados brutos extraídos em `game/` (~5.5 GB adicionais, preservando `README.md`) e re-extrair sob demanda via `--extract-iso` ou no próximo build.
  - Suporte a automação completa via CLI (`--setup`, `--build`, `--package`, `--clean`, `--cli`, `--gui`, `--help`).
  - Verificação de compiladores Clang 18+, Clang++, Ninja, CMake e bibliotecas de desenvolvimento (`qt6-base-dev`, `libvulkan-dev`, `libx11-dev`, `libwayland-client`).
* **Ponto de Entrada do Usuário Final (`installer/linux/package_entrypoint.sh` / `launcher.sh` / `install.sh`)**:
  - Menu de 4 opções para o jogador: `1 - Configurar ambiente`, `2 - Preparar port nativo`, `3 - Iniciar Launcher`, `4 - Sair`.
  - Validação detalhada de drivers Vulkan (Mesa RADV/ANV, NVIDIA) e bibliotecas essenciais com verificação em profundidade e fallbacks de paths do sistema.
  - Criação de atalhos de sistema com aspas seguras (`Exec="${exec_target}"`) para caminhos contendo espaços e apóstrofos (ex: `~/Dante's Inferno PC PORT`).
  - Criação automática de atalho executável e confiável na **Área de Trabalho** (`xdg-user-dir DESKTOP` / `~/Área de trabalho`) com permissão GNOME (`gio set metadata::trusted true`) e no menu de aplicativos (`~/.local/share/applications`).
* **Script de Preparação do Ambiente Linux (`scripts/setup-linux.sh`)**:
  - Detecção automática de distribuições (Ubuntu, Debian, Linux Mint, Fedora, Arch Linux, Manjaro, openSUSE).
  - Instalação orientada de pacotes, clonagem do ReXGlue SDK v0.10.0 e compilação nativa de `rexglue` CLI e `extract-xiso`.
* **Empacotador Automatizado (`scripts/package-linux-release.sh`)**:
  - Monta a estrutura de distribuição oficial e gera o arquivo `dist-release/DantesInferno-Linux-v0.5.0-alpha.tar.gz`.
* **Desinstalador Limpo e Seguro (`uninstall.sh` / `installer/linux/uninstall.sh`)**:
  - Auto-detecção do diretório de instalação mesmo sem argumentos ou a partir de `launcher.ini`.
  - Limpeza total de atalhos `.desktop` (menu XDG e Área de Trabalho), ícones (`~/.local/share/icons`), caches e configurações do Launcher.

#### Modificado
* **Runner Nativo Direto (`run-linux.sh`)**:
  - Corrigido o erro crítico `[ERROR] --game_data_root does not exist: .../game` ao rodar fora do layout do repositório.
  - Resolução dinâmica de `GAME_DIR`: verifica se `default.xex` está na pasta atual (`$HERE`), em `$HERE/game` ou no caminho registrado em `launcher.ini` (`game_root`).
  - Resolução dinâmica de `DATA_DIR`: mapeia saves e logs para a pasta configurada no Launcher (`--user_data_root` e `--storage_root`).
  - Leitura das opções gráficas salvas no Launcher (`config/launcher.ini`): resolução interna (2x), anisotropic override (16x), post effect (FXAA), vsync e fullscreen.
  - Atualização dos parâmetros de linha de comando para o padrão oficial do ReXGlue (`--resolution_scale`).

#### Removido
* **Legado Windows / PowerShell**:
  - Removidos scripts de automação PowerShell: `setup.ps1`, `launcher/package-release.ps1`, `patches/apply_sdk_patches.ps1`.
  - Eliminada necessidade de prefixos Wine, Proton ou scripts auxiliares para emular chamadas do Windows.

---

### 🎮 Launcher Nativo em Qt6 (`launcher-linux`)

#### Adicionado
* **Botão `Import ISO` com Barra de Progresso**:
  - Diálogo de seleção de imagens ISO do Xbox 360 (`.iso`).
  - Extração em segundo plano via thread dedicada utilizando `extract-xiso`.
  - Feedback em tempo real com barra de progresso visual no Zenity e no Launcher.
* **Single Source of Truth para Versão**:
  - `launcher-linux/CMakeLists.txt` configurado para ler dinamicamente a versão oficial a partir de `launcher/DantesInfernoLauncher/version.txt` (`0.5.0-alpha`).

#### Corrigido
* **Extração Não Destrutiva (`ImportService::commit`)**:
  - Corrigido bug crítico onde a extração da ISO executava `rm -rf destination`, apagando executáveis nativos e bibliotecas pré-existentes caso o usuário escolhesse a mesma pasta da instalação.
  - Substituído por migração não destrutiva que mescla os arquivos extraídos preservando binários, scripts e bibliotecas intactos.
* **Validação de Bibliotecas e Falha Silenciosa ao Clicar em "PLAY"**:
  - Diagnosticada ausência de `librexruntime.so` e `librexgpu-xenos.so` no ambiente de execução.
  - Adicionada verificação prévia de bibliotecas no método `MainWindow::play()` com mensagens de erro claras caso alguma dependência esteja ausente.
  - Expansão automática do `LD_LIBRARY_PATH` englobando a pasta do binário, a pasta `lib/` e a pasta raiz da aplicação.
* **Suporte a Espaços e Caracteres Especiais**:
  - Normalização e tratamento de caminhos contendo apóstrofos e espaços em branco em todo o launcher e scripts.

---

### 🛡️ Segurança e Integridade de Dados

#### Corrigido
* **Prevenção de Exclusão Acidental no `uninstall.sh`**:
  - Corrigido comportamento de colisão onde `INSTALL_DIR` era idêntico a `GAME_ROOT`. Se o usuário optasse por manter os dados do jogo (~7 GB) e os saves, o script anterior realizava `rm -rf` na pasta inteira.
  - Agora, se o usuário escolher manter os dados/saves, o desinstalador remove **estritamente** os arquivos do motor (`dantes_inferno`, `dantes_inferno_launcher`, bibliotecas `.so`, `extract-xiso`, `run-linux.sh`, `uninstall.sh`), garantindo que os saves e a mídia do jogo sejam 100% preservados.
* **Trava de Segurança do Repositório**:
  - `uninstall.sh` impede a exclusão acidental do repositório de código-fonte caso seja executado dentro da pasta de desenvolvimento (bloqueio por presença de `dantes_inferno_manifest.toml` ou `.git`).

---

### 📚 Documentação

#### Adicionado
* **`MANUAL_LINUX.md`**: Guia abrangente de engenharia e uso, com distinção clara entre os fluxos do desenvolvedor (`installer/linux/install.sh`) e do jogador final (`launcher.sh` / `install.sh`).
* **`dist-release/README.md`**: Manual focado no jogador para o pacote distribuível.
* **`dist-release/README-LINUX.txt`**: Instruções em texto puro inclusas na raiz do pacote `.tar.gz`.

---

## [0.4.1-alpha] — 2026-09-03

### Corrigido
* Hotfix para crash ao carregar o jogo em configurações ultrawide.
* Proteção no hook de aspect ratio contra chamadas com matrizes não-proporcionais.

---

## [0.4.0-alpha] — 2026-09-02

### Adicionado
* Suporte a correção de proporção de tela ultrawide (21:9 anamórfico).
* Atalho de teclado para fechamento do jogo (`Alt+F4`).
* Sincronização e rastreamento de versão no manifest.

---

## [0.3.3-alpha] — 2026-09-01

### Adicionado
* Esboço do seletor de família de glifos de botões (Xbox, PlayStation DualShock/DualSense) no launcher.
* Adição de suporte inicial ao empacotamento AppImage no Linux ARM64.

---

## [0.3.2-alpha] — 2026-08-30

### Adicionado
* Build base inicial de recompilação estática com ReXGlue SDK v0.10.0.
* Correção do sistema de savegame utilizando fibras e setjmp/longjmp em `src/dantes_inferno_hooks.h`.
* Patches no gerador de código VMX para eliminar corrupção em vídeos FMV VP6/Bink (`docs/vp6_fmv_corruption_fix.md`).
* Hotfix do instalador para preservar configurações e sincronizar versão inicial.
