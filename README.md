# Dante's Inferno (PC Port) — Compilações e Instalador Linux

[![Linux](https://img.shields.io/badge/Platform-Linux%20%7C%20Steam%20Deck-orange.svg)]()
[![Backend](https://img.shields.io/badge/Direct3D%2012-vkd3d--proton-blue.svg)]()
[![Recomp](https://img.shields.io/badge/Recompilation-ReXGlue%20v0.10.0-purple.svg)]()
[![Status](https://img.shields.io/badge/Status-Playable-brightgreen.svg)]()

Este repositório disponibiliza compilações pré-configuradas e um instalador gráfico (Zenity/CLI) para executar o port de **Dante's Inferno** (Xbox 360) no Linux e no Steam Deck através de recompilação estática nativa.

---

> [!IMPORTANT]
> ### ⚠️ AVISO LEGAL E SOBRE DIREITOS AUTORAIS (DISCLAIMER)
>
> 1. **ESTE REPOSITÓRIO NÃO CONTÉM O JOGO DANTE'S INFERNO.**
> 2. Nenhum arquivo proprietário, textura, áudio, modelo 3D, vídeo (FMV) ou código original da Electronic Arts / Visceral Games está incluído neste repositório ou em seus binários de distribuição.
> 3. O usuário **DEVE POSSUIR UMA CÓPIA ORIGINAL E LEGÍTIMA** do jogo *Dante's Inferno* para Xbox 360 (em formato de imagem ISO `.iso` ou arquivos extraídos contendo `default.xex`).
> 4. O instalador incluído realiza a extração e integração dos arquivos do jogo **localmente no computador do usuário**. O compartilhamento de arquivos protegidos por direitos autorais não é apoiado nem tolerado.

---

## 📖 Sobre este Repositório

Este repositório é baseado e derivado do projeto original **[hells-gate-recomp](https://github.com/florinp93/dantes-inferno)**.

O objetivo deste repositório específico é **compilar, empacotar e disponibilizar builds prontas para Linux**, acompanhadas de um instalador automatizado amigável para que qualquer usuário de Linux ou Steam Deck possa desfrutar do jogo sem a necessidade de configurar ferramentas avançadas de desenvolvimento (como toolchains cruzados de C++, Clang, Ninja, ReXGlue SDK ou Windows SDK).

### Recursos das Compilações Linux:
* **Executável compilado nativamente** com emulação precisa de GPU Xenos via Direct3D 12.
* **Assistente Gráfico Zenity** com seleção de ISO, extração integrada e verificação de arquivos.
* **Fallback completo para Terminal (CLI)** para servidores, distros mínimas ou preferência do usuário.
* **Configuração automática do Proton**: detecta instalações existentes do Steam ou baixa automaticamente uma versão portátil e independente do **GE-Proton** (sem exigir Steam instalado).
* **Integração total com o sistema operacional (XDG)**: cria atalhos no menu de aplicativos (`.desktop`) com ícone oficial em alta resolução.
* **Ajustes gráficos fáceis**: suporte integrado para resoluções 720p, 1080p, 1440p (2x padrão) e 4K UHD, além de integração com **MangoHud**.

---

## 🏆 Créditos e Reconhecimentos

Agradecimentos especiais a todos que tornaram este port viável:

* **[hells-gate-recomp](https://github.com/florinp93/dantes-inferno)** — Por **florinp93** e todos os colaboradores do projeto original, responsáveis pela engenharia reversa detalhada, correções de instruções vetoriais VMX/AltiVec, física de colisão, compatibilidade de saves com fibras/setjmp e suporte ao jogo.
* **[ReXGlue SDK](https://github.com/rexglue/rexglue-sdk)** — Pelo excelente kit de desenvolvimento e recompiler estático PowerPC (PPC) -> C++23.
* **Valve / Equipe do Proton / [GloriousEggroll](https://github.com/GloriousEggroll/proton-ge-custom)** — Pela compatibilidade impecável de APIs DirectX 12 e Vulkan no Linux via `vkd3d-proton`.
* **[XboxDev / extract-xiso](https://github.com/XboxDev/extract-xiso)** — Pela ferramenta open-source de extração de ISOs do Xbox.

---

## 💻 Requisitos do Sistema

* **Sistema Operacional:** Qualquer distribuição Linux moderna (Ubuntu, Linux Mint, Debian, Fedora, Arch Linux, Manjaro, Pop!_OS, etc.) ou **SteamOS / Steam Deck**.
* **Placa de Vídeo (GPU):** GPU compatível com **Vulkan** (AMD, Intel ou NVIDIA com drivers proprietários recentes).
* **Espaço em Disco:** ~8 GB livres para os dados do jogo extraídos.
* **Mídia do Jogo:** Imagem ISO original de *Dante's Inferno* para Xbox 360.
* **Interface Gráfica (opcional):** `zenity` (instalado por padrão no GNOME, KDE, Cinnamon, XFCE).

---

## 🚀 Tutorial de Instalação e Uso

### Passo 1: Baixar a Versão Mais Recente

1. Acesse a aba **[Releases](../../releases)** deste repositório.
2. Baixe o pacote compactado mais recente (ex: `DantesInferno-Linux-v0.3.2-alpha.tar.gz`).
3. Extraia o arquivo no local de sua preferência:
   ```bash
   tar -xzf DantesInferno-Linux-v*.tar.gz
   cd DantesInferno-Linux-v*
   ```

---

### Passo 2: Executar o Instalador

Dê duplo clique no arquivo `install.sh` no seu gerenciador de arquivos ou execute pelo terminal:

```bash
./install.sh
```

*(Caso seu sistema não tenha interface gráfica ou prefira o terminal, adicione a flag `--cli`: `./install.sh --cli`)*

---

### Passo 3: Passos do Assistente (Zenity)

O assistente gráfico abrirá o menu interativo:

1. Escolha a opção **🎮 Instalar Jogo**.
2. **Pasta de Instalação:** Escolha onde deseja salvar o jogo (o padrão recomendado é `~/.local/share/dantes-inferno`).
3. **Seleção da ISO:** Selecione o arquivo `.iso` original do seu jogo através da janela de arquivos.
4. **Extração Automática:** O instalador extrairá os arquivos do jogo com barra de progresso visual e validará a integridade do `default.xex`.
5. **Configuração do Proton:**
   - Se você possui Steam instalado, o instalador usará sua compatibilidade existente.
   - Caso contrário, o instalador perguntará se deseja baixar e configurar o **GE-Proton portátil**. Basta aceitar e ele cuidará de tudo automaticamente.
6. **Integração no Sistema:** Um atalho oficial será criado no menu de aplicativos do seu ambiente Linux.

---

### Passo 4: Como Jogar

Você pode iniciar o jogo de três formas:

1. **Pelo Menu do Sistema:** Procure por **Dante's Inferno** no menu de aplicativos do seu ambiente desktop (GNOME, KDE Plasma, etc.) e clique no ícone.
2. **Pelo Lançador Direto:**
   ```bash
   ~/.local/share/dantes-inferno/run-linux.sh
   ```
3. **Pelo Menu do `install.sh`:** Selecione **🚀 Iniciar / Jogar Dante's Inferno** para escolher resolução interna (720p, 1080p, 1440p ou 4K) e ativar o overlay do MangoHud.

---

## ⌨️ Comandos e Parâmetros dos Scripts

O script `install.sh` aceita comandos diretos para facilitar automações e atalhos:

| Comando | Descrição |
|---|---|
| `./install.sh` | Abre o menu gráfico principal (Zenity) |
| `./install.sh --install` | Abre diretamente o assistente de instalação do jogo |
| `./install.sh --run` | Inicia o jogo com diálogo de opções gráficas |
| `./install.sh --setup` | Executa a verificação e instalação de dependências |
| `./install.sh --build` | Compila o port nativo a partir do código-fonte (para desenvolvedores) |
| `./install.sh --package` | Gera um novo arquivo `.tar.gz` de distribuição |
| `./install.sh --uninstall` | Executa o desinstalador para remover o jogo e atalhos |
| `./install.sh --cli` | Força a execução em modo texto no terminal |
| `./install.sh --help` | Exibe o manual de parâmetros |

---

## 🎮 Controles e Atalhos Durante o Jogo

* **F1:** Ativa / desativa o overlay nativo ImGui com contador de frames (FPS reais) e gráfico de frametime.
* **F2:** Ativa o modo de avanço rápido (50x de velocidade sem limite de VSync) para acelerar ou pular vídeos/cutscenes. Pressione **F2** novamente para voltar à velocidade normal.
* **Controles:** Suporte nativo a controles de Xbox, PlayStation (DualShock 4 / DualSense) e controles genéricos via SDL3/Proton.

---

## 🗑️ Como Desinstalar

Caso deseje remover o jogo e todos os atalhos criados no sistema:
```bash
~/.local/share/dantes-inferno/uninstall.sh
```
Ou selecione a opção **🗑️ Desinstalar Jogo** no menu do `install.sh`.

---

## 📄 Licença

* Este projeto de empacotamento e os scripts de instalação são distribuídos sob licença de código aberto para fins de preservação e estudo de recompilação estática.
* Todos os direitos de propriedade intelectual, marcas e direitos autorais referentes a *Dante's Inferno* pertencem à **Electronic Arts Inc.** e **Visceral Games**.
