# Manual do Coletor — Guia do Usuário

Guia do usuário final: instale o Docker, prepare a máquina e suba a stack completa com um único comando.

## Sumário

1. [O que é o Coletor](#1-o-que-é-o-coletor)
2. [Instalar o Docker](#2-instalar-o-docker)
   - [Linux](#linux-servidor)
   - [Windows](#windows)
   - [macOS](#macos)
   - [Verificar a instalação](#verificar-a-instalação)
3. [Instalar o Coletor](#3-instalar-o-coletor)
4. [Configuração inicial](#4-configuração-inicial)
5. [Usando o Coletor](#5-usando-o-coletor)
   - [Subir a stack](#subir-a-stack)
   - [Ver o estado](#ver-o-estado)
   - [Acompanhar logs](#acompanhar-logs)
   - [Atualizar](#atualizar)
   - [Derrubar a stack](#derrubar-a-stack)
6. [Solução de problemas](#6-solução-de-problemas)

## 1. O que é o Coletor

O **Coletor** é um programa único que sobe toda a aplicação para você. Basta rodar `coletor up` para ligar tudo e `coletor down` para desligar.

Para funcionar, o Coletor só precisa de uma coisa instalada na máquina: o **Docker**. O resto ele resolve sozinho.

> **Em três passos**
> Instalar o Docker → instalar o Coletor → rodar `coletor up`. As seções abaixo cobrem cada passo.

## 2. Instalar o Docker

O Coletor precisa do **Docker Engine** e do plugin **Docker Compose**. Escolha abaixo as instruções para o seu sistema operacional.

### Linux `servidor`

No Ubuntu, Debian e na maioria das distribuições, o jeito mais simples é o script oficial do Docker, que já inclui o plugin Compose:

```bash
# Baixa e executa o instalador oficial do Docker
$ curl -fsSL https://get.docker.com | sudo sh

# Recomendado: usar o Docker (e o Coletor) sem digitar "sudo" toda vez:
$ sudo usermod -aG docker $USER
# saia e entre na sessão de novo para valer
```

> **Por que entrar no grupo `docker`?**
> Assim você roda o `coletor` como o seu próprio usuário, sem `sudo`. Isso importa: o Coletor guarda os dados na sua pasta pessoal, e rodar com `sudo` os criaria no perfil errado (o do root). Veja a [seção 4](#4-configuração-inicial).

Depois, garanta que o serviço do Docker está ligado e inicia junto com a máquina:

```bash
$ sudo systemctl enable --now docker
```

### Windows

No Windows, instale o **Docker Desktop**, que já vem com o Engine e o Compose:

1. Baixe o instalador em [docker.com/products/docker-desktop](https://www.docker.com/products/docker-desktop/).
2. Execute o instalador e aceite ativar o **WSL 2** quando solicitado.
3. Reinicie o computador se for pedido.
4. Abra o **Docker Desktop** e espere o ícone indicar que está "rodando".

### macOS

No Mac, instale o **Docker Desktop** (existe versão para chips Apple Silicon — M1/M2/M3 — e para Intel):

1. Baixe em [docker.com/products/docker-desktop](https://www.docker.com/products/docker-desktop/), escolhendo a versão certa para o seu chip.
2. Abra o arquivo `.dmg` e arraste o Docker para a pasta *Aplicativos*.
3. Abra o Docker, conceda as permissões pedidas e espere ficar "rodando".

### Verificar a instalação

Abra um terminal (no Windows, o *PowerShell*) e rode:

```bash
$ docker --version
$ docker compose version
```

Se os dois comandos mostrarem um número de versão, o Docker está pronto. Para um teste final:

```bash
$ docker run hello-world
```

> **Deu certo?**
> Se apareceu a mensagem *"Hello from Docker!"*, está tudo funcionando. Pode seguir para instalar o Coletor.

## 3. Instalar o Coletor

O Coletor é um **único arquivo executável** — não há instalador. Baixe a versão mais recente (**0.0.6**) na página de releases:

> 🔗 **Download:** <https://github.com/PNP-CCV/coletor/releases/tag/0.0.6>

### Linux e macOS

```bash
# Baixe o executável (Linux x86-64)
$ curl -L -o coletor https://github.com/PNP-CCV/coletor/releases/download/0.0.6/coletor

# Dê permissão de execução e mova para um diretório do sistema
$ chmod +x coletor
$ sudo mv coletor /usr/local/bin/

# Confirme que o sistema encontra o programa
$ coletor --help
```

### Windows

Coloque o arquivo `coletor.exe` numa pasta de sua preferência (por exemplo `C:\coletor\`) e, no PowerShell, execute a partir dessa pasta:

```powershell
PS> cd C:\coletor
PS> .\coletor.exe --help
```

> **Dica**
> Se `coletor --help` listar os comandos `up`, `down`, `status`, `logs` e `update`, a instalação está correta.

## 4. Configuração inicial

**Não há nada para configurar.** O Coletor já vem com todas as definições embutidas: portas, banco de dados e endereços dos serviços. Você não precisa criar nem editar arquivos.

Só existem dois pontos de atenção no primeiro uso:

> ⚠️ **1. Não rode o Coletor com `sudo`**
> A pasta de dados fica no diretório do **seu usuário**, então o Coletor **não precisa de administrador**. Rodar com `sudo` criaria os dados no perfil errado (o do root) e poderia confundir as próximas execuções. No Linux, basta seu usuário estar no grupo `docker` (veja a [seção 2](#linux-servidor)); no Windows/macOS, o Docker Desktop já cuida disso.

> ⚠️ **2. O primeiro `coletor up` baixa as imagens**
> Na primeira vez, o Coletor baixa os componentes da aplicação pela internet. Pode levar alguns minutos, dependendo da conexão. Nas próximas vezes é quase instantâneo.

Os dados ficam guardados numa pasta do **seu usuário**, gerenciada pelo próprio Coletor:

| Sistema | Pasta de dados |
| --- | --- |
| Linux | `~/.local/share/coletor` |
| macOS | `~/Library/Application Support/coletor` |
| Windows | `%LocalAppData%\coletor` |

## 5. Usando o Coletor

São cinco comandos. No Linux, rode-os **sem `sudo`** — para isso, seu usuário precisa estar no grupo `docker` (veja a [seção 2](#linux-servidor)). No Windows/macOS, com o Docker Desktop aberto, rode normalmente.

### Subir a stack

Liga todos os serviços. É o comando principal do dia a dia.

```bash
$ coletor up
```

Quando terminar, a aplicação está no ar. Você pode fechar o terminal — os serviços continuam rodando em segundo plano.

Por padrão a aplicação web fica disponível na porta **8000**. Para publicar noutra porta do computador (por exemplo, se a 8000 já estiver em uso), use `--port`:

```bash
$ coletor up --port 9090
```

> A porta escolhida **fica salva**: os próximos `coletor up` e `coletor update` continuam usando a 9090, sem precisar repetir `--port`. Para voltar ao padrão, rode `coletor up --port 8000`.

### Ver o estado

Mostra quais serviços estão ligados e se estão saudáveis.

```bash
$ coletor status
```

### Acompanhar logs

Mostra as mensagens dos serviços em tempo real — útil para verificar se algo deu errado. Pressione `Ctrl + C` para parar de acompanhar (isso *não* desliga a aplicação).

```bash
# Logs de todos os serviços
$ coletor logs

# Logs de um serviço específico (ex.: o servidor web)
$ coletor logs web
```

### Atualizar

Baixa a versão mais nova dos componentes e reinicia a aplicação com ela.

```bash
$ coletor update
```

> ℹ️ **Antes de atualizar, o Coletor verifica se há uma versão mais nova do próprio programa.**
> Se houver, ele **avisa, mostra o link para baixar e não atualiza a stack** — assim você troca o `coletor` primeiro e só depois sobe os componentes novos. O mesmo aviso aparece (sem bloquear) nos comandos `up`, `status`, `logs` e `down`. Se não houver internet, a verificação é ignorada e o `update` segue normalmente. A consulta é feita no máximo uma vez por dia (o resultado fica guardado), então o uso normal não fica mais lento. Para ver sua versão: `coletor --version`.

### Derrubar a stack

Desliga a aplicação. **Por padrão, os dados são preservados** — ao rodar `up` de novo, tudo volta como estava.

```bash
$ coletor down
```

Se você quiser apagar *também o banco de dados* (começar do zero), use `--wipe`:

```bash
$ coletor down --wipe
```

> ⚠️ **Atenção: `--wipe` apaga o banco de dados de forma permanente**
> O Coletor vai pedir uma confirmação antes de apagar. Não há como desfazer. Use apenas se tiver certeza de que não precisa mais dos dados.

## 6. Solução de problemas

O Coletor verifica o ambiente antes de agir e, se algo estiver faltando, mostra uma mensagem clara. As mais comuns:

| Mensagem | O que fazer |
| --- | --- |
| `Docker não encontrado` | O Docker não está instalado. Volte à [seção 2](#2-instalar-o-docker). |
| `Docker daemon não está rodando` | O Docker está instalado mas desligado. No Windows/macOS, abra o **Docker Desktop**. No Linux, rode `sudo systemctl start docker`. |
| `permission denied` ao falar com o Docker | Seu usuário não está no grupo `docker`. Siga a [seção 2](#linux-servidor) (`usermod -aG docker`) e reabra a sessão. **Não** resolva isso rodando o `coletor` com `sudo`. |
| `plugin docker compose ausente` | Falta o plugin Compose. No Linux, reinstale pelo script oficial da [seção 2](#linux-servidor); no Windows/macOS, atualize o Docker Desktop. |
| `sem permissão para criar ...` | A pasta de dados do seu usuário não pôde ser criada. Verifique as permissões do seu diretório pessoal. Não use `sudo` — isso criaria os dados no perfil do root. |

> **Continua com problema?**
> Rode `coletor status` e `coletor logs` para ver o que os serviços estão reportando, e leve essas informações para o suporte técnico.

---

*Manual do Coletor — guia do usuário final.*
