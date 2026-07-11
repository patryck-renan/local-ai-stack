# Self-Hosted AI Workspace no Fedora (Ollama + Odysseus)

[![Fedora](https://img.shields.io/badge/Fedora-342B59?style=for-the-badge&logo=fedora&logoColor=white)]()
[![Docker](https://img.shields.io/badge/Docker-2CA5E0?style=for-the-badge&logo=docker&logoColor=white)]()
[![Ollama](https://img.shields.io/badge/Ollama-000000?style=for-the-badge&logo=ollama&logoColor=white)]()
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg?style=for-the-badge)]()

Este repositório documenta o passo a passo completo para transformar uma máquina rodando Fedora Linux em um laboratório de Inteligência Artificial 100% local e privado. 

A stack utiliza o **Ollama** como motor de inferência (rodando o Llama 3.1) e o **[Odysseus](https://github.com/pewdiepie-archdaemon/odysseus)** (via Docker) como interface de workspace (Deep Research, Agentes e RAG de documentos).

## 📋 Pré-requisitos e Cenário Ideal

- **Sistema Operacional:** Fedora Linux (testado no Fedora 43 Xfce).
- **Memória RAM:** Pelo menos 16 GB disponíveis (livres ou em cache/buffers) para rodar modelos de 8B a 14B parâmetros de forma fluida e sem uso de Swap.
- **Armazenamento:** Mínimo de 10 GB livres no SSD (para imagens do Docker e o arquivo do modelo LLM).
- **Topologia de Rede (Local):** As portas `7000` (Odysseus UI) e `11434` (Ollama API) estarão em uso no host.

---

## 🚀 Passo 1: Preparação do Ambiente (Docker)

Como o Fedora utiliza o Podman nativamente, instalaremos o Docker CE oficial para garantir total compatibilidade com a rede e os volumes exigidos pelo Odysseus.

```bash
# Atualize o sistema e instale dependências base
sudo dnf update -y
sudo dnf install -y dnf-plugins-core git curl

# Adicione o repositório oficial do Docker e instale os pacotes
sudo dnf config-manager addrepo --from-repofile=https://download.docker.com/linux/fedora/docker-ce.repo
sudo dnf install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Habilite o serviço e adicione seu usuário ao grupo Docker
sudo systemctl enable --now docker
sudo usermod -aG docker $USER    
```

🔐 Nota de Segurança (SecOps): Em ambientes de produção, adicionar o usuário ao grupo docker equivale a conceder privilégios de root. Para manter o princípio do menor privilégio, é recomendável não adicionar o usuário ao grupo e executar os comandos subsequentes do Docker com sudo explícito.


## 🧠 Passo 2: Instalação do Motor de IA (Ollama)

O Ollama gerenciará os modelos locais e os executará na CPU/GPU.

```bash
# Instale o Ollama via script oficial
curl -fsSL https://ollama.com/install.sh | sh

# Confirme se o serviço está ativo
sudo systemctl enable --now ollama
sudo systemctl status ollama

# Baixe o modelo Llama 3.1 (8 Bilhões de parâmetros)
ollama run llama3.1
```

💡 **Dica:** Para sair do chat no terminal após o download, digite `/bye`

## 🔧 Passo 3: Configuração de Rede (Systemd Override)

O Odysseus rodará isolado dentro de um contêiner Docker. Por padrão, o Ollama rejeita conexões externas (incluindo da rede virtual do Docker). Precisamos alterar o bind do serviço para `0.0.0.0`.

```bash
# Crie um arquivo de sobrescrita (override) no systemd
sudo systemctl edit ollama
```

Adicione o seguinte conteúdo no espaço em branco do editor:

```
[Service]
Environment="OLLAMA_HOST=0.0.0.0"
```

Aplique as mudanças:

```bash
sudo systemctl daemon-reload
sudo systemctl restart ollama
```

🛡️ Firewalld: Embora o Ollama agora escute em todas as interfaces, o firewalld nativo do Fedora continua bloqueando acessos externos à porta 11434. A infraestrutura permanece segura e restrita à sua máquina local e à sub-rede do Docker.

## 🖥️ Passo 4: Instalação do Workspace (Odysseus)

Clone o repositório oficial do Odysseus e suba a infraestrutura via Docker Compose.

```bash
# Clone e acesse o diretório
git clone https://github.com/pewdiepie-archdaemon/odysseus.git
cd odysseus

# Crie o arquivo de ambiente e inicie os contêineres
cp .env.example .env
docker compose up -d --build
```

Aguarde o processo de build. O sistema gerará uma senha de administrador aleatória no primeiro boot. Recupere-a com o comando:

```bash
docker compose logs odysseus | grep -i password
```

## 🔗 Passo 5: Integração Final

1. Abra o navegador e acesse: http://localhost:7000.
2. Faça login com a senha recuperada no Passo 4.
3. Navegue até Settings(Canto inferior esquerdo) > Add Models/Add Local Models.
4. No campo URL/Endpoint, preencha com o IP da bridge padrão do Docker: http://172.17.0.1:11434
5. Clique em 'Add'. Na tela inicial de Chat, selecione o modelo llama3.1 no menu suspenso.

✅ Pronto! Seu ambiente de IA 100% local, seguro e integrado está operacional.

---

## 📊 Observabilidade e Troubleshooting

Em caso de anomalias na infraestrutura, utilize os comandos de diagnóstico abaixo:

  - Monitorar uso de recursos do contêiner em tempo real:
      `sudo docker stats odysseus`

  - Inspecionar os logs e o status de saída do Ollama:
      `journalctl -u ollama -n 50 --no-pager`

  - Garantir que as portas de serviço estão alocadas:
      `ss -tulpn | grep -E "7000|11434"`
<br>

## 👨‍💻 Desenvolvedor

<div align="center">
  <table>
    <tr>
      <td align="center" width="200">
        <img src="https://raw.githubusercontent.com/Tarikul-Islam-Anik/Animated-Fluent-Emojis/master/Emojis/People%20with%20professions/Man%20Technologist%20Light%20Skin%20Tone.png" width="100" style="border-radius: 50%;" alt="Avatar Patryck"><br>
      </td>
      <td align="center" width="500">
        <h3>Patryck Pereira</h3>
        <p>📍 Porto Alegre, RS | Estudante de ADS</p>
        <code>Python</code> | <code>Linux (Fedora)</code> | <code>Cybersecurity</code> | <code>Cloud</code>
      </td>
      <td align="center" width="200">
        <a href="https://www.linkedin.com/in/patryck-pereira-5104a6140/" target="_blank">
          <img src="https://img.shields.io/badge/-LinkedIn-%230077B5?style=for-the-badge&logo=linkedin&logoColor=white" alt="LinkedIn">
        </a>
      </td>
    </tr>
  </table>
</div>

<div align="center">
  <p>🚀 <b>Laboratório de IA Self-Hosted</b> construído para processamento privado e estudos em infraestrutura local.</p>
  <img src="https://img.shields.io/badge/Status-Em%20Desenvolvimento-orange?style=flat-square" alt="Status">
  <img src="https://img.shields.io/badge/Lab-Ambiente%20Controlado-red?style=flat-square" alt="Lab">
</div>

---
*Copyright (c) 2026 Patryck Pereira. Licenciado sob os termos da MIT License.*

