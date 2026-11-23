# Manual Completo — Dante's Inferno (Port Nativo Linux)
## Versão `v0.9.0-beta-linux` (AppImage x86_64 / ELF / Vulkan Nativo)

Este manual cobre tanto o uso completo do **AppImage e do Launcher Qt6 (para jogadores)** quanto a **compilação manual a partir do código-fonte (para desenvolvedores)**.

---

## Índice

1. [Visão Geral e Arquitetura](#1-visão-geral-e-arquitetura)
2. [Guia do Jogador: AppImage e Launcher Qt6 (PT-BR)](#2-guia-do-jogador-appimage-e-launcher-qt6-pt-br)
   * [2.1. Execução do AppImage](#21-execução-do-appimage)
   * [2.2. Aba JOGAR (ISO, Title Update 2, Saves e DLCs)](#22-aba-jogar-iso-title-update-2-saves-e-dlcs)
   * [2.3. Aba GRÁFICOS (Dante Renderer, SMAA, V-Sync, Sombras e Ultrawide)](#23-aba-gráficos-dante-renderer-smaa-v-sync-sombras-e-ultrawide)
   * [2.4. Aba ÁUDIO (PulseAudio/PipeWire, Estéreo e Filtros)](#24-aba-áudio-pulseaudiopipewire-estéreo-e-filtros)
   * [2.5. Aba IDIOMA](#25-aba-idioma)
   * [2.6. Aba SISTEMA (Diagnóstico, Drivers Vulkan e Atalhos)](#26-aba-sistema-diagnóstico-drivers-vulkan-e-atalhos)
   * [2.7. Modo Portátil (`portable.txt`)](#27-modo-portátil-portabletxt)
3. [Menu In-Game "Cheats" e Avanço Rápido de Cutscenes](#3-menu-in-game-cheats-e-avanço-rápido-de-cutscenes)
4. [Guia do Desenvolvedor: Compilação Manual](#4-guia-do-desenvolvedor-compilação-manual)
5. [Resolução de Problemas e Dicas](#5-resolução-de-problemas-e-dicas)

---

## 1. Visão Geral e Arquitetura

O **Hell's Gate Recomp (Linux Port)** não utiliza emulação em tempo de execução (JIT/interpretador) nem Wine/Proton. O executável `default.xex` (+ Title Update 2 `default.xexp`) do Xbox 360 é traduzido antecipadamente (*Ahead-of-Time*) de PowerPC para C++23 pelo **ReXGlue SDK v0.10.0** e compilado com **Clang** para um binário **ELF x86_64 nativo**.

* **Renderizador Principal (`renderer=dante`)**: motor Vulkan nativo in-process via **DiligentCore** (X11 e Wayland via SDL3), com SMAA, sombras até 4x, V-Sync estrito (`VK_PRESENT_MODE_FIFO_KHR`), cache persistente de pipelines (`pipelines.bin`) e Mod Loader de texturas.
* **Renderizador Alternativo (`renderer=xenos`)**: backend Vulkan Xenos legado (`librexgpu-xenos.so`) mantido como opção de compatibilidade.
* **Áudio Nativo SDL3**: prioriza automaticamente `pulseaudio,pipewire,alsa` para evitar *xruns* do callback RT do PipeWire em dispositivos USB, com downmix estéreo interno ITU-R BS.775 (2 canais).
* **Distribuição Autocontida**: empacotado em um único arquivo **AppImage x86_64** contendo o jogo, Launcher Qt6 PT-BR, libs Qt6/ICU, plugins X11/Wayland, `librexruntime.so`, `librexgpu-xenos.so`, `extract-xiso` e `shader_cache/`.

---

## 2. Guia do Jogador: AppImage e Launcher Qt6 (PT-BR)

### 2.1. Execução do AppImage

Baixe `DantesInferno-Linux-v0.9.0-beta-linux-x86_64.AppImage` na página de [Releases](../../releases) e execute:

```bash
chmod +x DantesInferno-Linux-v0.9.0-beta-linux-x86_64.AppImage
./DantesInferno-Linux-v0.9.0-beta-linux-x86_64.AppImage
```

A janela do Launcher abre **centralizada automaticamente** no monitor ativo.

---

### 2.2. Aba JOGAR (ISO, Title Update 2, Saves e DLCs)

* **Pasta do Jogo (`game_root`)**: aponte para a pasta onde estão seus arquivos extraídos (`default.xex`, `default.xexp`, `bigfile0.viv`, `bigfile1.viv`) ou clique em **Importar ISO** para extrair automaticamente sua `.iso` do Xbox 360 com barra de progresso usando o `extract-xiso` embutido.
  * *Recomendado:* inclua também o arquivo `default.xexp` (Title Update 2 — Media ID `49028C6A`) na mesma pasta do `default.xex`.
* **Pasta de Dados / Saves (`data_root`)**: local onde ficam salvos seus saves, logs (`logs/dantes_inferno.log`), cache de shaders/pipelines (`cache/dante/`) e configurações (`config/launcher.ini`).
* **DLCs**: coloque os arquivos de DLC (pacotes STFS) na subpasta `dlc/` dentro da pasta de dados/jogo — o port detecta e instala automaticamente na inicialização (`OnPostSetup`).
* **JOGAR**: inicia o jogo aplicando todas as configurações salvas.

---

### 2.3. Aba GRÁFICOS (Dante Renderer, SMAA, V-Sync, Sombras e Ultrawide)

* **Renderizador**:
  * **Dante (Vulkan Nativo — Recomendado)**: 60 FPS constantes, menor uso de CPU, suporte a SMAA, sombras em alta resolução e mods de textura.
  * **Xenos (Fallback Vulkan)**: opção de compatibilidade.
* **Escala de Resolução**: `1x` (720p Nativo), `1.5x` (1080p Full HD), `2x` (1440p Quad HD), `3x` (2160p 4K UHD).
* **Anti-aliasing SMAA (Renderizador Dante)**: suavização sub-pixel de bordas com baixíssimo custo de GPU.
* **Qualidade das Sombras (Dante)**: escala de mapa de sombras de `1x` a `4x`.
* **Sincronização Vertical (V-Sync)**: utiliza **`VK_PRESENT_MODE_FIFO_KHR`** estrito no Dante Renderer, eliminando completamente o *screen tearing*.
* **Taxa de Atualização (FPS)**: suporta 60, 120, 180 e 240 Hz (com interface/menus mantidos em 60 Hz para estabilidade).
* **Tela Cheia / Ultrawide (21:9 Hor+)**: menus e HUD são mantidos centralizados em 16:9 automaticamente em telas ultrawide.
* **Filtro Anisotrópico**: até `16x`.

> **Nota sobre o primeiro carregamento:** na primeira vez que você visita novas áreas no **Dante Renderer**, podem ocorrer breves quedas de 2–5 FPS enquanto novos pipelines Vulkan são compilados. Eles são gravados em `cache/dante/pipelines.bin` e pré-compilados nas sessões seguintes, mantendo **60.0 FPS cravados**.

---

### 2.4. Aba ÁUDIO (PulseAudio/PipeWire, Estéreo e Filtros)

O port Linux já seleciona automaticamente o backend `pulseaudio` (ou `pipewire-pulse`, com fallback para `pipewire` e `alsa`) para garantir áudio contínuo sem estalos mesmo em headsets/DACs USB sob carga 3D.

Opções disponíveis no Launcher (todas integradas diretamente ao `SDLAudioDriver`):
* **Forçar saída estéreo (2 canais)** *(Recomendado: Ligado)*: força o downmix interno ITU-R BS.775, evitando que o PipeWire receba 6 canais com LFE cheio em fones/caixas estéreo.
* **Usar apenas canais frontais**: descarta canais surround/LFE e mantém os canais frontais limpos.
* **Quadros em fila (Buffer)**: padrão `32` (ajustável de `16` a `64` quadros).
* **Ganho de volume**: controle linear de `10%` a `100%` (padrão `50%`).
* **Filtro passa-altas**: remove ruído sub-grave e offset DC (`0`–`500 Hz`, padrão `100 Hz`).

---

### 2.5. Aba IDIOMA

Permite definir o idioma do console virtual (Inglês, Espanhol, Francês, Alemão, Italiano, etc.) e o código de país.

---

### 2.6. Aba SISTEMA (Diagnóstico, Drivers Vulkan e Atalhos)

Nova aba dedicada à experiência *plug-and-play* no Linux:
* **Diagnóstico em tempo real**: exibe distribuição Linux, arquitetura, presença do loader Vulkan, ICDs instalados (NVIDIA / AMD RADV / Intel ANV) e status do `extract-xiso`.
* **Instalar Drivers Vulkan (1-clique)**: detecta sua distro (Ubuntu/Debian/Mint, Fedora, Arch/Manjaro, openSUSE) e instala os pacotes Vulkan corretos usando elevação gráfica (`pkexec`).
* **Criar Atalhos no Sistema**: gera atalhos com ícone oficial no **Menu de Aplicativos** e na **Área de Trabalho** apontando diretamente para o AppImage atual.

---

### 2.7. Modo Portátil (`portable.txt`)

Se você criar um arquivo vazio chamado `portable.txt` na mesma pasta do AppImage, o Launcher armazenará saves, configurações e cache localmente em uma subpasta `userdata/` ao lado do AppImage.

---

## 3. Menu In-Game "Cheats" e Avanço Rápido de Cutscenes

Durante o gameplay, aperte **Start** e selecione **`Cheats`** (ou pressione **`F4`** no teclado) para abrir o painel overlay temático:

* **Navegação**: funciona com **Controle** (`D-Pad` / `Analógico Esquerdo`, `A` alterna, `LB`/`RB` ou `Esquerda`/`Direita` muda valores, `B`/`Start` fecha), **Teclado** (`Setas`/`WASD`, `Enter`/`Espaço`, `Esc`/`Backspace`/`F`) e **Mouse**. Enquanto o painel está aberto, o menu de pausa atrás fica totalmente congelado.
* **Opções disponíveis**:
  1. **God Mode (`ON` / `OFF`)**: imortalidade total (zera dano e bloqueia reações de golpe/*knockback*).
  2. **One-Hit Kill (`ON` / `OFF`)**: qualquer ataque derrota inimigos instantaneamente.
  3. **Damage Multiplier (`1.0x` / `2.0x` / `4.0x`)**: multiplica o dano causado aos inimigos.
  4. **Game Speed (`0.5x` / `1.0x` / `1.5x` / `2.0x`)**: ajusta a velocidade geral do jogo.
  5. **Hold L3/R3 to Skip (`ON` / `OFF`, padrão `ON`)**: segure o clique do analógico esquerdo (**`L3`**), do analógico direito (**`R3`**) ou **`Ctrl Direito`** no teclado por mais de `0,25s` durante qualquer cutscene ou vídeo (FMV) para acelerar em **20x** com áudio mutado e indicador `>> FAST FORWARD (20x)`. Soltar o botão restaura tudo instantaneamente, executando todos os gatilhos de fase sem risco de *softlock* e sem conflitar com `L3+R3` (Redemption).

---

## 4. Guia do Desenvolvedor: Compilação Manual

### Pré-requisitos (Ubuntu / Debian / Linux Mint)
```bash
sudo apt update
sudo apt install -y git cmake ninja-build clang python3 python3-pip curl tar \
                    pkg-config libx11-dev libx11-xcb-dev libwayland-dev libvulkan-dev \
                    qt6-base-dev libgl1-mesa-dev
```

### Passo a passo (na raiz do código-fonte)
```bash
# 1. Preparar SDK (ReXGlue v0.10.0 + patches), DiligentCore (+ patches) e ferramentas
./scripts/setup-linux.sh

# 2. Colocar seus arquivos extraídos em game/ (default.xex + default.xexp TU2)
#    (ou extrair da ISO com tools/bin/extract-xiso -x jogo.iso -d game/)

# 3. Gerar código C++ AOT e aplicar patches de fibras/save
thirdparty/rexglue-sdk/out/linux-amd64/rexglue codegen dantes_inferno_manifest.toml
python3 patches/generated/apply_generated_patches.py

# 4. Compilar jogo + Launcher Qt6 (Clang + Ninja)
cmake -S . -B out/build/linux-native -G Ninja \
    -DCMAKE_BUILD_TYPE=Release \
    -DCMAKE_C_COMPILER=clang \
    -DCMAKE_CXX_COMPILER=clang++ \
    -DREXSDK_DIR="$PWD/thirdparty/rexglue-sdk" \
    -DDANTESINFERNO_BUILD_LAUNCHER=ON
cmake --build out/build/linux-native -j$(nproc)
ctest --test-dir out/build/linux-native --output-on-failure

# 5. Empacotar AppImage x86_64 (dist-release/)
./scripts/package-linux-appimage.sh
```

---

## 5. Resolução de Problemas e Dicas

* **Áudio (overrides manuais):** por padrão o port usa `SDL_AUDIO_DRIVER=pulseaudio,pipewire,alsa`. Caso deseje testar outro driver explicitamente, basta rodar `SDL_AUDIO_DRIVER=pipewire ./DantesInferno-Linux-v0.9.0-beta-linux-x86_64.AppImage` ou `SDL_AUDIO_DRIVER=alsa ...`.
* **Logs de diagnóstico:** ficam salvos em `<pasta_de_dados>/logs/dantes_inferno.log`.
* **Cache de pipelines Vulkan:** salvo em `<pasta_de_dados>/cache/dante/`. Se atualizar o driver de vídeo e notar qualquer artefato, basta apagar essa pasta para o cache ser regenerado automaticamente.
* **Overlay de FPS / Conquistas:** use **`F1`** para FPS, **`F7`** para a lista de Conquistas, **`F4`** (ou **Start → Cheats**) para o menu de Cheats e **`Alt+F4`** para sair imediatamente.
* **MangoHud:** basta executar `MANGOHUD=1 ./DantesInferno-Linux-v0.9.0-beta-linux-x86_64.AppImage`.
