# Dante's Inferno (PC Port) — Compilações e Informações Linux Nativo

[![Platform](https://img.shields.io/badge/Platform-Linux%20%7C%20Steam%20Deck-orange.svg)]()
[![Backend](https://img.shields.io/badge/Graphics-Vulkan%20Nativo-red.svg)]()
[![Binary](https://img.shields.io/badge/Format-ELF%2064--bit-blue.svg)]()
[![Recomp](https://img.shields.io/badge/Recompilation-ReXGlue%20v0.10.0-purple.svg)]()
[![Launcher](https://img.shields.io/badge/Launcher-Qt6%20Nativo-brightgreen.svg)]()
[![Release](https://img.shields.io/badge/Release-v0.6.6--beta-blue.svg)]()

Este repositório disponibiliza documentação, histórico de versões, manuais técnicos e informações sobre as compilações nativas de **Dante's Inferno** (Xbox 360) para Linux e Steam Deck via **Vulkan Nativo**, através de recompilação estática antecipada (*Ahead-Of-Time* - AOT).

> ⚡ **100% Nativo Linux**: Não necessita de Wine, Proton, emuladores ou prefixos Windows. O executável gerado é um binário ELF 64-bit nativo que roda diretamente sobre o driver Vulkan da sua GPU.

---

> [!IMPORTANT]
> ### ⚠️ AVISO LEGAL E SOBRE DIREITOS AUTORAIS (DISCLAIMER)
>
> 1. **ESTE REPOSITÓRIO NÃO CONTÉM O JOGO DANTE'S INFERNO.**
> 2. Nenhum arquivo proprietário, textura, áudio, modelo 3D, vídeo (FMV) ou código original da Electronic Arts / Visceral Games está incluído neste repositório.
> 3. O usuário **DEVE POSSUIR UMA CÓPIA ORIGINAL E LEGÍTIMA** do jogo *Dante's Inferno* para Xbox 360 (em formato de imagem ISO `.iso` ou arquivos contendo `default.xex`).
> 4. O Launcher oficial e os utilitários de extração realizam o processamento dos arquivos do jogo **localmente no computador do usuário**. O compartilhamento de arquivos protegidos por direitos autorais não é apoiado nem tolerado.

---

## 📖 Sobre as Compilações e Arquitetura

Este projeto é baseado no trabalho pioneiro de engenharia reversa do **[hells-gate-recomp](https://github.com/florinp93/dantes-inferno)** e no **[ReXGlue SDK](https://github.com/rexglue/rexglue-sdk)**.

O objetivo deste repositório é documentar e centralizar informações sobre os pacotes compilados para que qualquer usuário de Linux ou Steam Deck possa desfrutar do jogo com desempenho máximo, sem precisar configurar ambientes complexos de desenvolvimento (Clang, Ninja, CMake ou SDKs).

### Recursos da Edição Linux Nativa (v0.6.6-beta):
* **Suporte Nativo a Expansões (DLCs)**: Detecção e montagem automática de pacotes STFS na pasta `dlc/` durante a inicialização (`OnPostSetup`), integrando conteúdos adicionais como *Dark Forest*.
* **Executável ELF 64-bit nativo puro** (`bin/dantes_inferno`), traduzido de PowerPC para C++23 e compilado com Clang e Ninja (`-march=x86-64-v2`).
* **Renderização Vulkan Nativa** via `librexgpu-xenos.so`, garantindo alto framerate sem overhead de tradução.
* **Launcher Moderno em C++23 e Qt6** (`bin/dantes_inferno_launcher`):
  - Botão **Import ISO** com extração automática via `extract-xiso` em segundo plano com barra de progresso.
  - Seleção de resolução interna (720p, 1080p, 1440p 2x, 4K UHD 3x).
  - Suporte a monitores Ultrawide (21:9).
  - Suporte nativo ao overlay **MangoHud**.
  - Gerenciador de pastas de dados e saves.
* **Ponto de Entrada Dedicado do Jogador (`launcher.sh` / `install.sh`)**:
  - Menu interativo (Zenity ou Terminal CLI).
  - Verificação de drivers Vulkan e bibliotecas de runtime.
  - Criação de atalhos oficiais com ícone na **Área de Trabalho** (`xdg-user-dir DESKTOP`) e no **Menu de Aplicativos** sem duplicação de binários.
* **Runner Direto (`run-linux.sh`)**:
  - Auto-detecção inteligente dos arquivos do jogo (`default.xex`) na pasta atual ou a partir de `launcher.ini`.
  - Suporte a `--dlc_source_path` para carregamento imediato de expansões.
  - Herança automática das opções gráficas salvas no Launcher.
* **Desinstalador Limpo e Seguro (`uninstall.sh`)**:
  - Remove executáveis e atalhos do sistema sem apagar os arquivos extraídos do jogo (~7 GB) ou saves, salvo se expressamente confirmado pelo usuário.

---

## 🚀 Como Usar o Pacote Distribuível (Para Jogadores)

### Passo 1: Baixar a Versão Mais Recente
Baixe o pacote pré-compilado na aba **[Releases](../../releases)**:
* `DantesInferno-Linux-v0.6.6-beta.tar.gz`

Extraia o arquivo:
```bash
tar -xzf DantesInferno-Linux-v0.6.6-beta.tar.gz
cd DantesInferno-Linux-v0.6.6-beta
```

---

### Passo 2: Executar o Assistente / Launcher
Dê duplo clique em `launcher.sh` (ou `install.sh`) ou execute no terminal:
```bash
./launcher.sh
```

O menu apresentará as seguintes opções:
```text
========================================
       DANTE'S INFERNO - LINUX
========================================
1 - Configurar ambiente  (Verificar Vulkan e dependências de execução)
2 - Preparar port nativo (Criar atalhos na Área de Trabalho e Menu)
3 - Iniciar Launcher     (Abrir central do jogo para ISO, Saves e Jogar)
4 - Sair
```

* **1 - Configurar ambiente:** Valida se a sua GPU e bibliotecas do sistema suportam Vulkan.
* **2 - Preparar port nativo:** Instala o Launcher e cria os atalhos com ícone em alta definição na sua **Área de Trabalho** e no **Menu de Aplicativos**.
* **3 - Iniciar Launcher:** Inicia o Launcher em Qt6.

---

### Passo 3: Importar a ISO no Launcher e Jogar
1. No **Dante's Inferno Launcher**, clique em **Import ISO**.
2. Selecione a sua imagem ISO do Xbox 360 (`.iso`).
3. O Launcher extrairá os arquivos necessários automaticamente e configurará as pastas.
4. Ajuste suas preferências gráficas e clique em **PLAY** para jogar!

---

## 🛠️ Para Desenvolvedores: Hub de Compilação

Para quem compila a partir do código-fonte, o projeto disponibiliza o script de automação:

```bash
./installer/linux/install.sh
```

### Opções do Hub do Desenvolvedor:
* `1 - Setup`: Verifica dependências, instala ferramentas do sistema e compila os utilitários do SDK (`rexglue` e `extract-xiso`).
* `2 - Build`: Executa a tradução AOT (PowerPC -> C++23), aplica patches de fibras e compila os binários nativos ELF e o Launcher Qt6 em `out/build/linux-native/`.
* `3 - Package`: Monta a estrutura e gera o pacote distribuível `.tar.gz`.
* `4 - Clean`: Limpa o cache de compilação, objetos `.o`, código C++ gerado e logs, liberando ~1 GB de espaço em disco.
* `5 - Exit`: Encerra o script.

---

## 📚 Documentação Adicional

* **[MANUAL_LINUX.md](MANUAL_LINUX.md)** — Guia completo de engenharia, arquitetura, compilação manual e solução de problemas.
* **[CHANGELOG.md](CHANGELOG.md)** — Histórico completo de versões desde a `v0.3.2-alpha` até a `v0.5.0-alpha`.

---

## 🎮 Controles e Atalhos no Jogo

* **F1:** Ativa / desativa o overlay nativo ImGui com contador de FPS e estatísticas de renderização.
* **F2:** Ativa o avanço rápido (50x de velocidade sem limite de VSync) para acelerar vídeos e cutscenes. Pressione **F2** novamente para retornar à velocidade normal.
* **Alt + F4:** Fecha o jogo imediatamente e retorna à área de trabalho.
* **Controles:** Suporte nativo a controles de Xbox, PlayStation (DualShock 4 / DualSense) e controles genéricos via SDL3.

---

## 🏆 Créditos e Reconhecimentos

* **[hells-gate-recomp](https://github.com/florinp93/dantes-inferno)** — Por **florinp93** e todos os colaboradores do projeto original, pela engenharia reversa detalhada, correções do VMX, física e savegame.
* **[ReXGlue SDK](https://github.com/rexglue/rexglue-sdk)** — Pelo recompiler estático PowerPC -> C++23.
* **[XboxDev / extract-xiso](https://github.com/XboxDev/extract-xiso)** — Pela ferramenta open-source de extração de ISOs.

---

## 📄 Licença

* Este repositório contém apenas documentação, guias de compilação e histórico do projeto.
* Todos os direitos de propriedade intelectual, marcas e direitos autorais referentes a *Dante's Inferno* pertencem à **Electronic Arts Inc.** e **Visceral Games**.
