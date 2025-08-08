# Guia Completo: Instalação e Criação de um Fluxo Kestra Inteligente

Este documento explica como instalar o Kestra usando Docker e como criar um fluxo de trabalho (workflow) flexível que executa scripts Python localizados em um computador Windows, mesmo o Kestra rodando em um ambiente Linux.

## Parte 1: Instalação e Configuração do Kestra

Esta seção aborda a configuração inicial para colocar o Kestra no ar e conectá-lo a uma pasta local do seu computador.

### Passo 1: Estrutura de Arquivos

1.  Crie uma pasta principal para o seu projeto, por exemplo, `kestra_ambiente`.
2.  Dentro dela, crie dois arquivos vazios: `docker-compose.yml` e `kestra.yml`.
3.  Crie também a pasta que conterá seus scripts, por exemplo, `C:\kestra`. Dentro dela, coloque seu primeiro script, `etl_script.py`.

### Passo 2: Configurar o `docker-compose.yml`

Este arquivo gerencia o container do Kestra. Cole o seguinte conteúdo no seu `docker-compose.yml`:

```yaml
services:
  kestra:
    image: kestra/kestra:latest-full
    command: server standalone --config /app/config/kestra.yml
    ports:
      - "8080:8080"
    volumes:
      # Mapeia o arquivo de configuração do Kestra
      - ./kestra.yml:/app/config/kestra.yml
      
      # Cria um volume para dados persistentes do Kestra
      - kestra_data:/app/storage
      
      # O "portal" que conecta a pasta do seu PC ao container
      - C:\kestra:/kestra_pc

volumes:
  kestra_data:
```
**Ponto-chave:** A linha `- C:\kestra:/kestra_pc` é a mais importante. Ela cria um "portal" onde a pasta `C:\kestra` do seu PC aparece como `/kestra_pc` dentro do container Kestra.

### Passo 3: Configurar o `kestra.yml`

Este arquivo define as configurações internas do Kestra. Cole o seguinte conteúdo nele:

```yaml
kestra:
  repository:
    type: h2
  queue:
    type: h2
  storage:
    type: local
    local:
      base-path: "/app/storage"
```

### Passo 4: Iniciar o Kestra

1.  Abra um terminal na pasta `kestra_ambiente`.
2.  Execute o comando para parar qualquer versão antiga e garantir uma limpeza:
    ```bash
    docker-compose down
    ```
3.  Execute o comando para iniciar o Kestra em segundo plano:
    ```bash
    docker-compose up -d
    ```
4.  Aguarde um minuto e acesse a interface do Kestra no seu navegador em **`http://localhost:8080`**.

## Parte 2: Criando o Fluxo Inteligente

Agora, vamos criar o fluxo que executa qualquer script da sua pasta `C:\kestra`, aceitando o caminho do Windows como entrada e "traduzindo-o" para o formato Linux.

### Passo 1: Criar um Novo Fluxo

1.  Na interface do Kestra, vá para a seção "Flows".
2.  Clique em "Create".
3.  Copie e cole o código YAML abaixo no editor.

### Passo 2: O Código do Fluxo

```yaml
id: python_traduz_caminho_windows
namespace: company.team

tasks:
  - id: executar_script_windows_em_linux
    type: io.kestra.plugin.scripts.shell.Commands
    
    taskRunner: 
      type: io.kestra.plugin.core.runner.Process
      
    commands:
      # O comando abaixo pega o caminho do Windows e o converte para um
      # caminho válido dentro do container Linux antes de executar.
      - >
        python /kestra_pc/projects/project_test_1/app.py

# --- SEÇÃO DE AGENDAMENTO ADICIONADA ---
triggers:
  - id: agendamento_as_14_25
    type: io.kestra.plugin.core.trigger.Schedule
    # Expressão Cron para "às 14:25, todos os dias"
    cron: "35 14 * * *"

```

### Passo 3: Executando o Fluxo

1.  Salve o fluxo.
2.  Clique no botão "Execute".
3.  Aparecerá um formulário com o campo **`caminho_windows`**.
4.  Cole o caminho completo do script que você deseja executar (ex: `C:\kestra\outra_pasta\outro_script.py`).
5.  Clique em "Execute" novamente.

