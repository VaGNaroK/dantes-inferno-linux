# Dante's Inferno (PC Port) — Edição Nativa Linux (AppImage x86_64)

[![Platform](https://img.shields.io/badge/Platform-Linux%20x86__64%20%7C%20Steam%20Deck-orange.svg)]()
[![Format](https://img.shields.io/badge/Format-AppImage%20%7C%20ELF%2064--bit-blue.svg)]()
[![Backend](https://img.shields.io/badge/Renderer-Dante%20Vulkan%20%2B%20Xenos-red.svg)]()
[![Recomp](https://img.shields.io/badge/Recompilation-ReXGlue%20v0.10.0-purple.svg)]()
[![Launcher](https://img.shields.io/badge/Launcher-Qt6%20PT--BR-brightgreen.svg)]()
[![Release](https://img.shields.io/badge/Release-v0.9.0--beta--linux-blue.svg)](../../releases/tag/v0.9.0-beta-linux)

Este repositório centraliza as **releases prontas para jogar (AppImage x86_64)**, documentação, manual técnico e histórico de versões do port nativo de **Dante's Inferno** (Xbox 360) para Linux e Steam Deck, utilizando recompilação estática antecipada (*Ahead-Of-Time* — AOT) e **Vulkan Nativo**.

> ⚡ **100% Nativo Linux (Plug-and-Play)**: Não necessita de Wine, Proton, emuladores ou prefixos Windows. O AppImage já embute o executável ELF 64-bit, o Launcher Qt6 em PT-BR, bibliotecas de runtime (`librexruntime.so`, `librexgpu-xenos.so`, Qt6, ICU, plugins X11/Wayland), `extract-xiso` e `shader_cache/`.

---

> [!IMPORTANT]
> ### ⚠️ AVISO LEGAL E SOBRE DIREITOS AUTORAIS (DISCLAIMER)
>
> 1. **ESTE REPOSITÓRIO NÃO CONTÉM O JOGO DANTE'S INFERNO.**
> 2. Nenhum arquivo proprietário, textura, áudio, modelo 3D, vídeo (FMV) ou código original da Electronic Arts / Visceral Games está incluído neste repositório ou nos pacotes de release.
> 3. O usuário **DEVE POSSUIR UMA CÓPIA ORIGINAL E LEGÍTIMA** do jogo *Dante's Inferno* para Xbox 360 (em imagem `.iso` ou arquivos extraídos contendo `default.xex` + Title Update 2 `default.xexp`).
> 4. O Launcher realiza a extração e preparação dos arquivos do jogo **localmente no computador do usuário**.

---

## 🚀 Como Jogar (AppImage — Recomendado)

### Passo 1: Baixar o AppImage
Baixe o arquivo mais recente na aba **[Releases](../../releases/latest)**:
* `DantesInferno-Linux-v0.9.0-beta-linux-x86_64.AppImage`

### Passo 2: Dar permissão de execução e abrir
```bash
chmod +x DantesInferno-Linux-v0.9.0-beta-linux-x86_64.AppImage
./DantesInferno-Linux-v0.9.0-beta-linux-x86_64.AppImage
```
*(Ou clique com o botão direito no arquivo → Propriedades → Permissões → "Permitir executar como programa" e dê duplo clique).*

### Passo 3: Configurar pelo Launcher Qt6 (PT-BR)
1. Na primeira execução, o Launcher abre **centralizado** e permite escolher a pasta de **Dados/Saves** e a pasta do **Jogo**.
2. Clique em **Importar ISO** para extrair sua imagem `.iso` do Xbox 360 automaticamente (com barra de progresso), ou aponte para uma pasta que já contenha `default.xex` (e `default.xexp` do Title Update 2).
3. Se precisar de drivers Vulkan ou quiser criar atalhos no sistema, acesse a aba **`SISTEMA`** (instalação 1-clique via `pkexec` e criação de atalhos na Área de Trabalho e Menu de Aplicativos).
4. Ajuste suas preferências em **`GRÁFICOS`**, **`ÁUDIO`** e **`IDIOMA`**, salve e clique em **JOGAR**!

---

## ✨ Destaques da Versão `v0.9.0-beta-linux`

Baseado no projeto original **[florinp93/hells-gate-recomp](https://github.com/florinp93/hells-gate-recomp)** (sincronizado até o commit **`3ce28ec`**) com melhorias e correções dedicadas ao Linux:

### 🎨 Gráficos e Desempenho
* **Renderizador Nativo Vulkan "Dante" (`--renderer=dante`)**: 100% funcional no Linux (X11 e Wayland via SDL3) rodando a **60 FPS cravados**, com suporte a taxas de quadros destravadas (60/120/180/240 FPS), anti-aliasing **SMAA**, sombras em alta resolução (`1x`–`4x`) e resolução até **4K UHD**.
* **V-Sync Estrito (`VK_PRESENT_MODE_FIFO_KHR`)**: elimina completamente o *screen tearing* quando "Sincronização vertical" está marcada no Launcher.
* **Ultrawide 21:9 Hor+ com Menus/HUD em 16:9 Centralizado** e **Glifos Kenney's Input Prompts** (129 ícones embutidos para PlayStation, Xbox e Teclado/Mouse).
* **Texture Studio & Mod Loader**: carregamento nativo de mods de textura em tempo real no Dante Renderer.
* **Fallback Xenos (`--renderer=xenos`)**: mantido como alternativa selecionável na aba *Gráficos*.

### 🔊 Áudio Limpo no Linux
* **Zero picotes / estalos in-game**: priorização automática do driver SDL3 `pulseaudio,pipewire,alsa` (evitando *xruns* do callback RT do PipeWire em DACs/headsets USB sob carga 3D).
* **Downmix Estéreo Nativo ITU-R BS.775**: detecção real dos canais físicos do dispositivo (2 canais) antes de abrir o stream, com suporte completo às opções do Launcher (*Forçar estéreo*, *Apenas canais frontais*, *Ganho de volume*, *Filtro passa-altas* e *de-clicker* suave).

### 🎮 Menu In-Game "Cheats" & Avanço Rápido Seguro
* **Menu `Cheats` no Start (Pausa)** (ou tecla **`F4`**): painel overlay temático navegável por **controle, teclado e mouse** (com bloqueio total do menu de pausa por trás):
  - **God Mode** (imortalidade + sem *knockback*)
  - **One-Hit Kill** (morte instantânea em inimigos)
  - **Damage Multiplier** (`1.0x` / `2.0x` / `4.0x`)
  - **Game Speed** (`0.5x` / `1.0x` / `1.5x` / `2.0x`)
  - **Hold L3/R3 to Skip**: segure o clique do analógico esquerdo (**`L3`**), direito (**`R3`**) ou **`Ctrl Direito`** durante qualquer cutscene ou vídeo (FMV) para avançar em **20x** com áudio mutado e **risco zero de *softlock***.

### 🖥️ Launcher Qt6 Nativo (100% em PT-BR)
* 5 abas completas: **`JOGAR`**, **`GRÁFICOS`**, **`ÁUDIO`**, **`IDIOMA`** e **`SISTEMA`**.
* **Aba `SISTEMA`**: diagnóstico em tempo real (distro, arquitetura, Vulkan ICD, `extract-xiso`), instalação 1-clique de drivers Vulkan via `pkexec` e criação de atalhos no Menu/Área de Trabalho.
* **Quit to Desktop** integrado ao menu principal e de pausa do jogo, suporte automático a **DLCs** (pasta `dlc/`) e sistema de **Conquistas** com notificações visuais e sonoras.

---

## 🎮 Controles e Atalhos no Jogo

| Atalho / Botão | Ação |
| :--- | :--- |
| **`Start` → `Cheats`** ou **`F4`** | Abre / fecha o menu in-game de **Cheats** |
| **Segurar `L3` ou `R3`** *(ou `Ctrl Direito`)* | **Fast-Forward seguro (20x)** para acelerar cutscenes e FMVs (sem conflitar com `L3+R3` da Redemption) |
| **`F1`** | Ativa / desativa o overlay de FPS e tempo de quadro |
| **`F7`** | Abre / fecha a lista de **Conquistas** |
| **`Alt + F4`** | Fecha o jogo imediatamente |
| **Controles** | Xbox, PlayStation (DualShock 4 / DualSense) e genéricos nativamente via SDL3 + Teclado e Mouse |

---

## 📚 Documentação e Código-Fonte

* **[MANUAL_LINUX.md](MANUAL_LINUX.md)** — Manual completo do Launcher Qt6, aba Sistema, Cheats, compilação manual e solução de problemas.
* **[CHANGELOG.md](CHANGELOG.md)** — Histórico detalhado desde a `v0.5.0-alpha` até a `v0.9.0-beta-linux`.
* **Código-Fonte do Port Linux:** [VaGNaroK/Dantes_Inferno_Linux_Port](https://github.com/VaGNaroK/Dantes_Inferno_Linux_Port)

---

## 🏆 Créditos e Reconhecimentos

* **[hells-gate-recomp](https://github.com/florinp93/hells-gate-recomp)** — Por **florinp93** e colaboradores do projeto original (base sincronizada até o commit `3ce28ec`).
* **[ReXGlue SDK](https://github.com/rexglue/rexglue-sdk)** — Recompiler estático PowerPC → C++23.
* **[DiligentCore](https://github.com/DiligentGraphics/DiligentCore)** — Engine Vulkan utilizada pelo Dante Renderer.
* **[XboxDev / extract-xiso](https://github.com/XboxDev/extract-xiso)** — Extração open-source de ISOs de Xbox 360.
* **[Kenney Input Prompts](https://kenney.nl/assets/input-prompts)** — Ícones de botões embutidos.

---

## 📄 Licença
* Este repositório contém apenas documentação, guias e releases do port Linux.
* Todos os direitos de propriedade intelectual referentes a *Dante's Inferno* pertencem à **Electronic Arts Inc.** e **Visceral Games**.
