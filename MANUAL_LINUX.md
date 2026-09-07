# Guia Completo: Compilação e Execução Nativa no Linux (Sem Wine/Proton)
## Dante's Inferno (Hell's Gate Recomp) - Edição Nativa Linux (ELF / Vulkan)

Este manual descreve o passo a passo detalhado para preparar o ambiente no Linux, traduzir o código de máquina PowerPC do Xbox 360 para C++23 antecipadamente (*Ahead-of-Time*), compilar os binários nativos Linux (formato ELF) com backend Vulkan e utilizar o novo Launcher em Qt6 ou o Hub interativo em Zenity.

---

## Índice

1. [Visão Geral e Arquitetura Nativa](#1-visão-geral-e-arquitetura-nativa)
2. [Pré-requisitos do Sistema](#2-pré-requisitos-do-sistema)
3. [Fluxo Recomendado para Novos Usuários: Hub Unificado (`install.sh`)](#3-fluxo-recomendado-para-novos-usuários-hub-unificado-installsh)
4. [Passo a Passo Manual via Linha de Comando](#4-passo-a-passo-manual-via-linha-de-comando)
   * [Passo 1: Preparação do Ambiente (`setup-linux.sh`)](#passo-1-preparação-do-ambiente-setup-linuxsh)
   * [Passo 2: Preparação dos Arquivos do Jogo (`game/`)](#passo-2-preparação-dos-arquivos-do-jogo-game)
   * [Passo 3: Geração do Código C++ (Codegen AOT)](#passo-3-geração-do-código-c-codegen-aot)
   * [Passo 4: Aplicação dos Patches de Fibras e Save](#passo-4-aplicação-dos-patches-de-fibras-e-save)
   * [Passo 5: Compilação Nativa (Jogo ELF + Launcher Qt6)](#passo-5-compilação-nativa-jogo-elf--launcher-qt6)
   * [Passo 6: Execução do Jogo e Launcher Qt6](#passo-6-execução-do-jogo-e-launcher-qt6)
   * [Passo 7: Empacotamento para Distribuição (`package-linux-release.sh`)](#passo-7-empacotamento-para-distribuição-package-linux-releasesh)
5. [Resolução de Problemas e Dicas de Performance](#5-resolução-de-problemas-e-dicas-de-performance)

---

## 1. Visão Geral e Arquitetura Nativa

O **Hell's Gate Recomp** não utiliza emulação em tempo de execução (JIT nem interpretador como Xenia ou RPCS3). O código do executável `default.xex` do Xbox 360 é totalmente traduzido para código C++23 pelo **ReXGlue SDK** e compilado diretamente pelo **Clang** em um binário executável **ELF nativo** do Linux.

* **Execução**: **100% Nativa**. Não necessita de Wine, Proton, emuladores ou prefixos Windows.
* **Backend Gráfico**: **Vulkan Nativo** de alta performance com suporte a Wayland e X11.
* **Launcher Gráfico**: Desenvolvido em **C++23 e Qt6** (`dantes_inferno_launcher`), permitindo ajustar resoluções, Ultrawide (21:9), filtros anisotrópicos, MSAA e controle de áudio diretamente.
* **Hub e Automação**: Assistente em **Zenity** (`install.sh`) com fallback via terminal CLI para compilar, instalar e empacotar em qualquer ambiente.

---

## 2. Pré-requisitos do Sistema

* **Sistema Operacional**: Distribuição Linux x86_64 moderna (Ubuntu 22.04+, Linux Mint 21/22, Fedora 39+, Arch Linux, etc.).
* **Placa de Vídeo / Driver**: GPU compatível com **Vulkan** (AMD com Mesa RADV, Intel com ANV, ou NVIDIA com drivers proprietários).
* **Espaço em Disco**: ~10 GB livres durante a compilação.

### Instalação de Pacotes Base:

#### Ubuntu / Debian / Linux Mint:
```bash
sudo apt update
sudo apt install -y git cmake ninja-build clang clang++ python3 python3-pip curl tar \
                    pkg-config libx11-xcb-dev libwayland-dev libvulkan-dev qt6-base-dev zenity
```

#### Fedora:
```bash
sudo dnf install -y git cmake ninja-build clang clang-tools-extra python3 python3-pip curl tar \
                    pkgconf-pkg-config libX11-devel wayland-devel vulkan-headers vulkan-loader-devel \
                    qt6-qtbase-devel zenity
```

#### Arch Linux / Manjaro:
```bash
sudo pacman -S --needed git cmake ninja clang python python-pip curl tar \
                        pkgconf libx11 wayland vulkan-headers vulkan-icd-loader \
                        qt6-base zenity
```

---

## 3. Fluxos de Trabalho: Desenvolvedor vs. Jogador Final

O projeto foi totalmente refatorado para estabelecer uma **separação estrita de responsabilidades** entre o ambiente de engenharia (código-fonte) e o produto entregue ao jogador comum.

### Por que essa separação foi necessária?
Anteriormente, o script `install.sh` misturava opções de compilação C++, configuração de compiladores, instalação permanente no sistema (`~/.local/share`), atalhos de desktop, desinstalador e execução direta em um único menu confuso:
* **Para o Desenvolvedor**: Não faz sentido poluir pastas do sistema (`~/.local/share` ou atalhos na Área de Trabalho) apenas para compilar e testar uma alteração no código C++.
* **Para o Jogador Final**: Quem apenas deseja jogar não possui Clang 18+, Ninja, bibliotecas de desenvolvimento do Qt6 ou ferramentas de compilação, e não deve ser exposto a menus de engenharia. O jogador recebe um pacote autocontido com binários pré-compilados.

---

### A. Para o Desenvolvedor: Hub de Compilação (`installer/linux/install.sh`)

No repositório de código-fonte, o script [`installer/linux/install.sh`](installer/linux/install.sh) é a ferramenta central e exclusiva de engenharia e build nativo:

```bash
./installer/linux/install.sh
```

*(Abre automaticamente a interface gráfica em Zenity caso disponível, ou funciona em modo terminal via `--cli` / argumentos diretos).*

```text
======================================================
   Dante's Inferno - Hub de Compilação e Build Nativo
======================================================
  1 - Setup    Configurar ambiente de desenvolvimento e dependências
  2 - Build    Compilar o port nativo Linux (ReXGlue + Clang + Ninja)
  3 - Package  Gerar o pacote de distribuição (dist-release/)
  4 - Clean    Limpar cache de compilação e dados brutos (game/)
  5 - Exit     Sair
```

#### Detalhamento das Opções do Desenvolvedor:
1. **`⚙️  1 - Setup: Configurar ambiente de desenvolvimento e dependências`**
   * Detecta a distribuição Linux (Ubuntu/Debian, Fedora, Arch/Manjaro, openSUSE).
   * Valida e orienta a instalação de compiladores (Clang 18+, Clang++), ferramentas (CMake 3.25+, Ninja, Git, Python 3) e bibliotecas de desenvolvimento (`qt6-base-dev`, `libvulkan-dev`, `libx11-dev`, `libwayland-client`).
   * Clona e aplica os patches de VMX e compilação no **ReXGlue SDK v0.10.0** (`thirdparty/rexglue-sdk/`).
   * Compila os utilitários nativos de suporte: `tools/bin/rexglue` (CLI de codegen) e `tools/bin/extract-xiso` (extrator de ISOs).
2. **`🔨 2 - Build: Compilar o port nativo Linux (ReXGlue + Clang + Ninja)`**
   * Verifica a presença de `game/default.xex` (oferece extração de ISO caso ausente).
   * Executa a tradução AOT de PowerPC para C++23: `rexglue codegen dantes_inferno_manifest.toml`.
   * Aplica os patches de contexto de save e fibras (`apply_generated_patches.py`).
   * Configura o CMake com Clang++ e Ninja com otimizações `-march=x86-64-v2`.
   * Compila o executável ELF do jogo (`dantes_inferno`) e o Launcher nativo (`dantes_inferno_launcher`) em `out/build/linux-native/`.
   * Copia imediatamente as bibliotecas de runtime (`librexruntime.so`, `librexgpu-xenos.so`) para a pasta de build.
   * **Não faz instalações globais no sistema operacional**.
3. **`📦 3 - Package: Gerar o pacote de distribuição (dist-release/)`**
   * Executa `scripts/package-linux-release.sh`.
   * Monta a estrutura autocontida com binários ELF, bibliotecas de runtime, utilitário `extract-xiso`, scripts dedicados ao jogador, ícones em alta resolução e documentação.
   * Produz o arquivo compactado `dist-release/DantesInferno-Linux-v<VERSÃO>.tar.gz` a partir da fonte única de versão (`launcher/DantesInfernoLauncher/version.txt`).
4. **`🧹 4 - Clean: Limpar cache de compilação e dados brutos (game/)`**
   * Remove com segurança arquivos e diretórios intermediários de compilação gerados:
     - `out/` (árvore de build do CMake/Ninja, objetos `.o` ~800 MB).
     - `generated/default/` (arquivos C++ gerados pelo ReXGlue codegen ~120 MB).
     - `tools/extract-xiso-src/build/` (build intermediário do extrator).
     - Logs temporários de compilação em `~/.cache/dantes-inferno-dev/`.
   * **Limpeza opcional de dados brutos (`game/`)**: Pergunta se o desenvolvedor deseja também apagar os arquivos brutos extraídos do jogo (`game/default.xex`, `game/bigfile0.viv`, `game/bigfile1.viv`, etc., liberando ~5.5 GB adicionais). O arquivo `game/README.md` é preservado. Sempre que for necessária uma nova compilação no futuro, basta utilizar a opção `2 - Build` (ou `--extract-iso`) para extrair a ISO novamente para a pasta `game/`.
   * **Preserva 100% intactos** os pacotes prontos em `dist-release/` e saves do jogador.
5. **`❌ 5 - Exit: Sair`**

#### Automação via Linha de Comando (CLI Flags):
```bash
./installer/linux/install.sh --setup    # Executa o setup de dependências
./installer/linux/install.sh --build    # Dispara a compilação nativa
./installer/linux/install.sh --package  # Gera o pacote tar.gz
./installer/linux/install.sh --clean    # Limpa cache e arquivos de compilação
./installer/linux/install.sh --cli      # Força interface em modo texto
./installer/linux/install.sh --gui      # Força interface Zenity
./installer/linux/install.sh --help     # Exibe resumo de ajuda
```

---

### B. Para o Jogador Final: Pacote Distribuível (`launcher.sh` / `install.sh`)

O jogador que baixa o arquivo `DantesInferno-Linux-v<VERSÃO>.tar.gz` não precisa compilar nada. Ao descompactar o arquivo, ele encontra o ponto de entrada dedicado [`launcher.sh`](launcher.sh) (e seu atalho conveniente `install.sh`):

```bash
./launcher.sh   # ou ./install.sh
```

```text
========================================
       DANTE'S INFERNO - LINUX
========================================
1 - Configurar ambiente (Verificar Vulkan e dependências de execução)
2 - Preparar port nativo (Criar atalhos na Área de Trabalho e Menu)
3 - Iniciar Launcher (Abrir central do jogo para ISO, Saves e Jogar)
4 - Sair
```

#### Detalhamento das Opções do Jogador:
1. **`⚙️  1 - Configurar ambiente`**
   * Verifica se a GPU suporta **Vulkan 1.2+** (drivers Mesa RADV para AMD, ANV para Intel ou driver proprietário NVIDIA).
   * Valida a presença de bibliotecas de execução básicas no sistema (como `libstdc++.so.6`, `libX11.so.6`, `libc.so.6`).
   * Informa de forma amigável os comandos do gerenciador de pacotes da distro caso falte algum driver gráfico, sem tentar compilar nada.
2. **`🎮 2 - Preparar port nativo`**
   * Permite instalar os arquivos em `~/.local/share/dantes-inferno` ou em uma pasta personalizada escolhida pelo jogador.
   * Cria o atalho oficial com ícone no **Menu de Aplicativos** (`~/.local/share/applications/dantes-inferno.desktop`) com aspas protetoras (`Exec="${exec_target}"`) para suportar pastas com espaços e apóstrofos (ex: `~/Dante's Inferno PC PORT`).
   * Cria o atalho executável na **Área de Trabalho** do usuário (`xdg-user-dir DESKTOP`, como `~/Área de trabalho` ou `~/Desktop`) e aplica a confiança nativa no GNOME (`gio set metadata::trusted true`).
3. **`🚀 3 - Iniciar Launcher`**
   * Executa o `dantes_inferno_launcher` em Qt6 com `LD_LIBRARY_PATH` configurado.
   * No Launcher, o jogador clica em **Import ISO** para selecionar sua imagem de disco original de Xbox 360 (`.iso`), ajusta suas preferências visuais e clica em **PLAY**.
4. **`❌ 4 - Sair`**

---

### C. Scripts Auxiliares do Pacote

* **Runner Direto (`run-linux.sh`)**:
  - Permite rodar o jogo diretamente pelo terminal ou integrá-lo à Steam como jogo não-Steam.
  - **Detecção Inteligente**: Localiza automaticamente o arquivo `default.xex` na pasta atual, em `game/` ou a partir do `launcher.ini` (`game_root`).
  - **Saves Automáticos**: Carrega a pasta de saves a partir de `launcher.ini` (`data_root`).
  - **Herança de Preferências**: Lê automaticamente a escala de resolução (720p a 4K UHD), filtro anisotrópico (16x), FXAA, vsync e tela cheia salvas no Launcher.
* **Desinstalador Limpo e Seguro (`uninstall.sh`)**:
  - Remove com segurança os atalhos da Área de Trabalho e do menu de aplicativos.
  - **Proteção de Dados**: Se o jogo tiver sido instalado na mesma pasta onde a ISO foi extraída, o desinstalador remove **apenas** os binários e bibliotecas do executável, mantendo intactos seus arquivos de jogo (`default.xex`, `.viv` ~7 GB) e seus saves caso você escolha preservá-los.

---

## 4. Passo a Passo Manual via Linha de Comando

Se você preferir executar cada etapa manualmente no terminal, siga os passos abaixo:

### Passo 1: Preparação do Ambiente (`setup-linux.sh`)

Execute o script automatizado de preparação:

```bash
./scripts/setup-linux.sh
```

Esse script realiza:
1. Verificação de todas as ferramentas e bibliotecas do sistema (Clang, Qt6, Vulkan, Ninja).
2. Download e inicialização dos submódulos do **ReXGlue SDK** (`v0.10.0`).
3. Aplicação dos patches de VMX (vetorização, física e cutscenes VP6).
4. Compilação da ferramenta de linha de comando nativa `tools/bin/rexglue`.
5. Compilação da ferramenta nativa `tools/bin/extract-xiso` (para descompactar ISOs do Xbox 360).
6. Instalação das bibliotecas Python (`pillow`, `texture2ddecoder`) necessárias para processamento de assets.

---

## Passo 2: Preparação dos Arquivos do Jogo (`game/`)

O projeto necessita do executável extraído do Xbox 360 em `game/default.xex`.

Se você tiver uma imagem ISO (`.iso`):
```bash
tools/bin/extract-xiso -x /caminho/para/seu_jogo.iso -d game/
```
*(Ou utilize o menu interativo `./installer/linux/install.sh`, que extrai a ISO automaticamente via interface gráfica).*

---

## Passo 3: Geração do Código C++ (Codegen AOT)

Com o arquivo `game/default.xex` posicionado, execute o gerador de código C++:

```bash
tools/bin/rexglue codegen dantes_inferno_manifest.toml
```

Os arquivos C++ gerados serão criados dentro de `generated/default/`.

---

## Passo 4: Aplicação dos Patches de Fibras e Save

Dante's Inferno utiliza um sistema customizado de fibras para o sistema de savegame. Aplique os patches no código gerado:

```bash
python3 patches/generated/apply_generated_patches.py
```

---

## Passo 5: Compilação Nativa (Jogo ELF + Launcher Qt6)

Configure o CMake e compile com Ninja:

```bash
cmake -S . -B out/build/linux-native -G Ninja \
    -DCMAKE_BUILD_TYPE=Release \
    -DCMAKE_C_COMPILER=clang \
    -DCMAKE_CXX_COMPILER=clang++ \
    -DREXSDK_DIR=thirdparty/rexglue-sdk \
    -DDANTESINFERNO_BUILD_LAUNCHER=ON

cmake --build out/build/linux-native -j$(nproc)
```

Binários gerados:
* `out/build/linux-native/dantes_inferno` (Executável ELF nativo do jogo)
* `out/build/linux-native/launcher-linux/dantes_inferno_launcher` (Launcher Qt6 nativo)

---

## Passo 6: Execução do Jogo e Launcher Qt6

### Opção A: Pelo Launcher Qt6
Abra o launcher gráfico para configurar gráficos, áudio e controles:
```bash
./out/build/linux-native/launcher-linux/dantes_inferno_launcher
```

### Opção B: Pelo Runner Nativo
```bash
./scripts/run-linux.sh
```
Variáveis de ambiente opcionais:
* `RES_SCALE=2` (1 = 720p, 1.5 = 1080p, 2 = 1440p, 3 = 4K)
* `ULTRAWIDE_ASPECT=2.333333` (Ativa suporte nativo a monitores 21:9)
* `MANGOHUD=1` (Ativa overlay de FPS e consumo de hardware)

### Opção C: Pelo Hub do Desenvolvedor
```bash
./installer/linux/install.sh --build
```

---

## Passo 7: Empacotamento para Distribuição (`package-linux-release.sh`)

Para criar o pacote distribuível independente para jogadores Linux (sem necessidade de compilar código-fonte):

```bash
./scripts/package-linux-release.sh
```
Ou via Hub:
```bash
./installer/linux/install.sh --package
```

O pacote será gerado em:
`dist-release/DantesInferno-Linux-v0.5.0-alpha.tar.gz`

Quem baixar esse arquivo só precisará descompactar e executar `./launcher.sh` (ou `./install.sh`), que apresentará o menu do jogador com verificação de ambiente, atalhos na Área de Trabalho e abertura do Launcher.

---

## 5. Resolução de Problemas e Dicas de Performance

* **Falta do Qt6 no CMake**: Certifique-se de que o pacote de desenvolvimento está instalado (`qt6-base-dev` no Ubuntu/Debian ou `qt6-qtbase-devel` no Fedora).
* **Overlay MangoHud**: Para ativar estatísticas de FPS e hardware, basta instalar `mangohud` no sistema e rodar com `MANGOHUD=1 ./scripts/run-linux.sh`.
* **Atalhos no Jogo**:
  * `Alt+F4`: Fecha o jogo imediatamente e retorna à área de trabalho.
  * `F1`: Ativa/desativa o overlay interno de FPS do ReXGlue.
  * `F2`: Ativa aceleração rápida (50x) para pular cutscenes de vídeo (FMVs).
