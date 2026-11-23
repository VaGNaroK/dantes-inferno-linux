# Changelog — Dante's Inferno (Hell's Gate Recomp)

Todas as alterações notáveis neste projeto serão documentadas neste arquivo.

O formato é baseado em [Keep a Changelog](https://keepachangelog.com/pt-BR/1.0.0/) e este projeto segue o versionamento semântico.

## [0.9.0-beta-linux] — 2026-11-23

Marco principal do port nativo Linux x86_64: sincronizado com o projeto original (`florinp93/hells-gate-recomp`) até o commit **`3ce28ec`** (pós-`v0.8.6-beta-hotfix`) e com correções definitivas de vídeo (**Dante Renderer Vulkan**), áudio sem cortes no PipeWire/USB, **V-Sync estrito sem tearing**, **menu in-game de Cheats + Hold L3/R3 to Skip**, e empacotamento **AppImage x86_64** pronto para uso.

### 📦 Base do Projeto Original (até o commit `3ce28ec`)
* **Texture Studio & Mod Loader**: carregamento nativo de mods de textura no Dante Renderer e ferramenta para extração/edição de texturas.
* **Glifos Kenney's Input Prompts**: 129 ícones PNG embutidos em tempo de compilação (PlayStation e Teclado/Mouse) sem depender de Win32/GDI.
* **Ultrawide 16:9 Centralizado para Menus e HUD** e opção nativa **Quit to Desktop** no menu principal e de pausa.
* **Otimizações de GPU/CPU/VRAM**: sleep no GPU fence (`81461c7`), redução de CPU/VRAM (`dff9962`), DynamicHeapSize Vulkan de 256 MB, sobrevivência a falhas de alocação de textura (`c28e606`), correções de animação (`animation_hooks.cpp`), `memory_hooks.cpp` e fallback/aviso para GPUs incompatíveis (`3ce28ec`).

### ✨ Novas Modificações e Correções do Port Linux
* **Renderizador Nativo Vulkan "Dante" 100% funcional no Linux (60 FPS cravados)**:
  - Corrigido o cálculo de offset da faixa física `0xE0000000+` em `src/guest_memory.h` e `src/dantes_inferno_hooks.h` (`0` no Linux com páginas de 4 KB em vez de `+0x1000` do Windows), eliminando erros de tradução de shaders, tela rosa e travamento ao carregar saves.
  - Suporte nativo a swapchain Vulkan em **X11 e Wayland** via propriedades do SDL3 (`src/renderer/vulkan_device.cpp` e `WindowSDL::GetNativeWindowHandle()`).
  - Corrigido crash de inicialização no `SeedShaderStorage()` (movido para `OnPostSetup()` com proteção de ponteiro nulo).
* **Áudio 100% Limpo no Linux (Zero Picotes / Estalos In-Game)**:
  - Priorização automática do driver SDL3 `pulseaudio,pipewire,alsa` (`sdl_audio_driver.cpp`, `achievement_sound.cpp`, `AppRun`), eliminando *xruns* do callback RT do PipeWire em dispositivos USB sob carga 3D.
  - Detecção real dos canais nativos do dispositivo antes de abrir o stream, usando o downmix estéreo interno ITU-R BS.775 (2ch) em vez de enviar 6ch virtuais ao PipeWire.
  - Reconectadas todas as opções de áudio do launcher (`audio_force_stereo`, `audio_front_only`, `audio_output_gain`, `audio_highpass_hz`), sanitização de `NaN`/`Inf` e *de-clicker* suave.
* **V-Sync Estrito (Zero Screen Tearing)**:
  - `PresentTexture` em `src/renderer/renderer.cpp` agora respeita a opção `--vsync` do launcher.
  - Patch no DiligentCore (`SwapChainVkImpl.cpp`) priorizando `VK_PRESENT_MODE_FIFO_KHR` estrito sobre `FIFO_RELAXED`, eliminando o *tearing* residual na borda inferior.
* **Menu In-Game "Cheats" + Avanço Rápido Seguro de Cutscenes**:
  - Nova entrada **"Cheats"** injetada diretamente no menu de pausa (Start) + atalho `F4`, abrindo um painel overlay temático (ouro/brasa) navegável por **controle, teclado e mouse** com congelamento completo do menu de pausa atrás (`xam_input_suppressed`).
  - Cheats em tempo real: **God Mode**, **One-Hit Kill**, **Damage Multiplier** (`1.0x / 2.0x / 4.0x`), **Game Speed** (`0.5x / 1.0x / 1.5x / 2.0x`) e **Hold L3/R3 to Skip** (segurar o clique do analógico esquerdo `L3`, direito `R3` ou `Ctrl Direito` acelera cutscenes e FMVs em `20x` com áudio mutado e risco zero de *softlock*).
* **Launcher Qt6 PT-BR & AppImage x86_64 Autocontido**:
  - Janela do Launcher agora abre centralizada automaticamente no monitor ativo (`launcher-linux/main.cpp`).
  - Nova aba **`SISTEMA`** com diagnóstico em tempo real, instalação 1-clique de drivers Vulkan via `pkexec` e criação de atalhos.
  - Empacotamento **AppImage x86_64** (`scripts/package-linux-appimage.sh`) com Qt6, ICU, plugins X11/Wayland, `librexruntime.so`, `librexgpu-xenos.so`, `extract-xiso` e `shader_cache/` embutidos.

---

## [0.8.6-beta-hotfix-linux] — 2026-11-22

Esta versão sincroniza o port nativo Linux com o upstream oficial até o commit `3ce28ec` (integrando `v0.8.5-beta`, `v0.8.6-beta`, `v0.8.6-beta-hotfix` e 4 correções pós-hotfix), preservando integralmente o Launcher Qt6 em PT-BR, scripts nativos Linux e blindagens de compilação.

### 🚀 Destaques da Versão
* **Texture Studio & Mod Loader (`src/renderer/mod_loader.*`, `tools/texture_studio/`)**:
  - Carregamento nativo de mods de textura em tempo real no Dante Renderer e ferramenta completa para extração/edição de texturas (incluindo exportação de normal maps RGB padrão).
* **Glifos Kenney's Input Prompts (`assets/glyphs/kenney/`, `src/renderer/button_glyphs.cpp`)**:
  - 129 ícones PNG embutidos em tempo de compilação (`glyph_icons.cpp`) via `stb_image`, trazendo glifos nítidos de PlayStation e teclado/mouse nativamente no Linux sem depender de Win32/GDI.
* **Ultrawide 16:9 Centralizado para Menus e HUD**:
  - Menus e interface desenhados em área 16:9 centralizada em telas ultrawide.
* **Quit to Desktop Integrado (`src/menu_hooks.*`)**:
  - Opção nativa de sair para a área de trabalho adicionada diretamente ao menu principal e ao menu de pausa.
* **Otimizações de Desempenho, VRAM e Estabilidade**:
  - Sleep no GPU fence em vez de busy-spin (`81461c7`), redução de uso de CPU/VRAM em PCs mais modestos (`dff9962`), DynamicHeapSize Vulkan ampliado para 256 MB, sobrevivência a falhas de alocação de textura e descarte de views obsoletas (`c28e606`).
  - Correção de animações desconectando layers cujo blend graph ficou ilegível (`src/animation_hooks.cpp`), `src/memory_hooks.cpp`, fallback para Vulkan e aviso claro para GPUs incompatíveis (`3ce28ec`).
  - Diagnóstico aprimorado (`src/crash_log.h`, `src/renderer/thread_cpu.*`, `src/path_util.h`), teclas de dev protegidas atrás de `--dev_keys` e uso de configurações do launcher sem precisar abri-lo.
* **Blindagens Linux & Experiência Plug-and-Play**:
  - Patch unificado do ReXGlue SDK (`patches/sdk/rexglue-sdk-v0.10.0.patch`) atualizado com todas as mudanças upstream + manutenção das correções C++23 Clang (`__cpp_concepts`) e tolerância a entrypoints TU2.
  - Guards `#ifdef _WIN32` preservados/aplicados em `CMakeLists.txt`, `src/renderer/button_glyphs.cpp`, `src/renderer/vulkan_device.cpp`, `src/user_paths.cpp` e `SeedShaderStorage()` em `src/dantes_inferno_app.h`.
  - **Launcher Qt6 autocontido no pacote de release**: `scripts/package-linux-release.sh` agora empacota as bibliotecas runtime do Qt6, ICU e plugins X11/Wayland + `bin/qt.conf` (`RUNPATH=$ORIGIN:$ORIGIN/../lib`).
  - **Nova aba `SISTEMA` no Launcher Qt6 (`launcher-linux/system_service.*`)**: diagnóstico em tempo real (distro, arquitetura, loader/ICD Vulkan e `extract-xiso`), **instalação 1-clique de drivers Vulkan via `pkexec`** e criação de atalhos (Área de Trabalho e Menu) direto pelo Launcher.
  - **`launcher.sh` inteligente (sem Zenity desnecessário)**: abre o Launcher Qt6 diretamente quando todas as libs estão presentes, usando diálogo + `pkexec` apenas como fallback caso falte alguma biblioteca crítica (`--menu` mantém o menu clássico).

---

## [0.8.4-beta-linux] — 2026-10-16

Esta versão integra todas as atualizações do upstream oficial (`v0.8.0-beta` até `v0.8.4-beta`), trazendo o **novo renderizador nativo Vulkan ("Dante Renderer")**, suporte a **taxas de quadros destravadas (60, 120, 180 e 240 FPS)**, **anti-aliasing SMAA**, **sombras em alta resolução**, **sistema de conquistas sonoro e visual**, **glifos dinâmicos de botões**, **correções de áudio XMA** e o **Lançador Qt6 100% traduzido para Português do Brasil (PT-BR)**, mantendo a compatibilidade 100% nativa com Linux (ELF / Vulkan Puro).

### 🚀 Destaques da Versão
* **Novo Renderizador Nativo Vulkan "Dante" (`--renderer=dante`)**: 
  - Renderização direta in-process via DiligentCore, eliminando a sobrecarga de tradução da GPU Xenos do Xbox 360.
  - Suporte a sombras nítidas de alta definição com escala configurável de 1x a 4x (`--dante_shadow_scale=1..4`).
  - Pós-processamento anti-aliasing SMAA nativo sub-pixel (`--dante_smaa`), com embutimento de shaders em formato literal raw compatível com Clang 18.
  - O modo legado "Xenos" foi mantido como rota de compatibilidade/fallback (`--renderer=xenos`).
* **Taxas de Quadros Destravadas (60, 120, 180 e 240 FPS)**:
  - Suporte a altas taxas de atualização de tela (`--video_mode_refresh_rate=120|180|240`), mantendo a lógica de física e a interface do usuário travadas a 60 Hz para estabilidade perfeita de gameplay.
* **Sistema Integrado de Conquistas & Efeitos Sonoros**:
  - Novo sistema de notificações visuais toast (`achievement_ui.cpp`) e áudio (`achievement_sound.cpp`) ao desbloquear conquistas.
  - Novo overlay com métricas de desempenho e tempo de quadro.
* **Lançador Nativo Qt6 100% em Português do Brasil (PT-BR)**:
  - Todas as abas traduzidas: **JOGAR**, **GRÁFICOS**, **ÁUDIO** e **IDIOMA**.
  - Tradução integral de opções gráficas, qualidade de sombras, resolução (720p Nativo, 1440p Quad HD, 2160p 4K), anti-aliasing, buffer de áudio e configurações regionais do console.
  - Tradução completa dos diálogos de importação de ISO com barra de progresso, assistente de primeira inicialização e mensagens de diagnóstico.
  - Ajuste no script [`scripts/run-linux.sh`](file:///home/vagnarok/hells-gate-recomp-main/scripts/run-linux.sh) permitindo executar `./scripts/run-linux.sh --launcher` a qualquer momento de forma independente.
* **Blindagem Multiplataforma para Linux**:
  - Isolamento de dependências Win32: `gdi32` condicionado a `if(WIN32)` no CMake e inclusões de `<windows.h>` isoladas com `#ifdef _WIN32` em `button_glyphs.cpp`, `renderdoc_capture.cpp` e `renderer_config.cpp`.
  - Adaptação da inicialização de swapchain no `vulkan_device.cpp` para utilizar `Diligent::NativeWindow.WindowId` no Linux.
  - Patch unificado do ReXGlue SDK (`patches/sdk/rexglue-sdk-v0.10.0.patch`) consolidando as correções de áudio XMA upstream com as correções locais de C++23 no Clang (`__cpp_concepts`) e tolerância a entrypoints do Title Update 2 (TU2).
  - Adicionado suporte automatizado para clone e aplicação de patches do DiligentCore no [`scripts/setup-linux.sh`](file:///home/vagnarok/hells-gate-recomp-main/scripts/setup-linux.sh).

---

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
