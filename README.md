# Pokédex Comunitária - Chatbot e Backend (n8n) 🤖

<p align="center">
  <img src="https://i.ibb.co/N25nLM7D/Captura-de-tela-2025-06-17-235100.png" alt="Visão geral dos workflows do Agente Pokédex no n8n">
</p>

## 📖 Sobre o Projeto

Este repositório contém o **cérebro** e o **backend** da experiência **Pokédex Comunitária**, desenvolvida para o evento **Unilavras na Praça**. Utilizando a plataforma de automação de workflows **n8n**, este sistema gerencia toda a lógica de conversação por meio do [**chat**](https://github.com/joaopaulofcc/Pokedex-Praca-Chat), processamento de dados e interação com o [**servidor do telão interativo**](https://github.com/joaopaulofcc/Pokedex-Praca-Telao).

O projeto é composto por um agente de IA principal que orquestra diversas ferramentas (outros workflows) para criar uma experiência de chat completa e dinâmica.

---

## 🚀 Arquitetura e Funcionalidades

O sistema é modular, onde um agente principal delega tarefas para workflows especializados, simulando uma arquitetura de microserviços.

1.  **Recepção da Mensagem:** Um usuário envia uma mensagem (ex: "Quero capturar um Pokémon!"), que aciona o workflow principal via webhook.
2.  **Agente de IA (Google Gemini):** O workflow `Principal_Agente_Pokedex.json` recebe a mensagem. Ele utiliza o **Google Gemini** para interpretar a intenção do usuário e seguir um roteiro de conversa pré-definido, que inclui um quiz e o processo de captura.
3.  **Uso de Ferramentas:** O agente principal invoca outras "ferramentas" (workflows) conforme a necessidade:
    * **Quiz:** Chama o `Ferramenta_buscarPerguntaQuiz.json` para obter uma pergunta aleatória do `BancoDePerguntasPokemon.csv`.
    * **Consulta:** Chama o `Ferramenta_buscarInfoPokemon.json` para consultar dados na PokéAPI e descrições personalizadas na planilha `DescricoesPokemon.csv`.
    * **Registro:** Chama o `Ferramenta_registrarCaptura.json` para verificar o status da captura no `StatusPokedex.csv` e registrar a nova captura no `LogCapturasPokemon.csv`.
4.  **Comando para o Telão:** Ao final de uma captura, o agente chama o `Ferramenta_ativarAnimacaoTelao.json`, que envia um comando via HTTP POST para o servidor do telão, ativando a animação visual para todo o público do evento.
5.  **Manutenção:** Em segundo plano, o `Ferramenta_Acorda_Aplicações.json` é executado a cada 3 minutos para manter o servidor do telão e o próprio webhook do chat sempre ativos.

---

## 🛠️ Componentes do Projeto

### Workflows do n8n (`/n8n_workflows/`)
* **`Principal_Agente_Pokedex.json`**: O orquestrador principal com o agente de IA.
* **`Ferramenta_buscarInfoPokemon.json`**: Ferramenta que busca e formata dados detalhados de um Pokémon.
* **`Ferramenta_registrarCaptura.json`**: Ferramenta que gerencia a lógica de registro de capturas novas ou repetidas.
* **`Ferramenta_buscarPerguntaQuiz.json`**: Ferramenta que sorteia uma pergunta do banco de dados do quiz.
* **`Ferramenta_ativarAnimacaoTelao.json`**: Ferramenta que envia o comando final para o servidor do telão.
* **`Ferramenta_Acorda_Aplicações.json`**: Workflow agendado para manter os serviços online.

### Fontes de Dados (`/data_sources/`)
* **`BancoDePerguntasPokemon.csv`**: Contém as perguntas e respostas para o quiz.
* **`DescricoesPokemon.csv`**: Armazena as descrições personalizadas para cada Pokémon.
* **`LogCapturasPokemon.csv`**: Grava um histórico completo de todas as capturas realizadas no evento.
* **`StatusPokedex.csv`**: Controla o estado atual da Pokédex (quais Pokémon já foram capturados e por quem).

---

## 🚀 Como Utilizar e Configurar

Para replicar este projeto, siga os passos abaixo.

### Pré-requisitos
* Uma instância do **n8n** (local, em nuvem ou via n8n.cloud).
* Uma **Conta Google** para usar o Google Sheets e o Google Gemini.
* Chave de API para o **Google Gemini**.

### Passo 1: Configurar as Planilhas (Google Sheets)
Os arquivos na pasta `/data_sources` são a base para as planilhas que os workflows utilizam.

1.  Acesse o [Google Sheets](https://sheets.google.com).
2.  Para cada arquivo `.csv` da pasta `data_sources`, crie uma nova planilha.
3.  Dentro de cada planilha nova, vá em **Arquivo > Importar**.
4.  Faça o upload do arquivo `.csv` correspondente (ex: para a planilha "StatusPokedex", importe o `StatusPokedex.csv`).
5.  Após criar cada planilha, **copie o ID da Planilha**. Você pode encontrá-lo na URL. Ex: `https://docs.google.com/spreadsheets/d/ID_DA_PLANILHA_AQUI/edit`. Guarde esses IDs.

### Passo 2: Configurar as Credenciais no n8n
É recomendado criar as credenciais antes de importar os workflows.

1.  No seu painel do n8n, vá em **Credentials** e clique em **Add credential**.
2.  Crie as seguintes credenciais:
    * **Google Sheets:** Procure por "Google Sheets", selecione e siga as instruções para conectar sua Conta Google.
    * **Google Gemini:** Procure por "Google Gemini", selecione e insira sua chave de API.
    * **Header Auth (para o Telão):**
        * Procure por **"Header Auth"**.
        * **Credential Name:** `Segredo Telao Pokedex`
        * **Name:** `x-webhook-secret`
        * **Value:** Cole aqui o mesmo segredo que o seu servidor do telão espera.

### Passo 3: Importar e Ajustar os Workflows
1.  No n8n, importe todos os arquivos `.json` da pasta `/n8n_workflows/`. Você pode fazer isso copiando o conteúdo do JSON e colando em um workflow em branco.
2.  Abra cada workflow importado e ajuste os nós que apresentarem erros:
    * **Nós do Google Sheets:** Nos nós de "Google Sheets", selecione a credencial que você criou no Passo 2. No campo "Document ID", cole o ID da planilha correspondente que você salvou no Passo 1.
    * **Nó do Google Gemini (no Agente Principal):** Selecione sua credencial do Gemini.
    * **Nó de HTTP Request (em `ativarAnimacaoTelao`):**
        * Atualize a **URL** para apontar para a sua instância do `Pokedex-Praca-Telao`.
        * Na autenticação ("Authentication"), selecione **"Header Auth"** e escolha a credencial `Segredo Telao Pokedex`.

### Passo 4: Ativar!
1.  Ative os workflows que precisam estar sempre ouvindo: `Principal_Agente_Pokedex` e `Ferramenta_Acorda_Aplicações`.
2.  Pronto! Seu backend está configurado para interagir com o chat e com o telão.

---

## ⚖️ Aviso Legal e Créditos

> Este é um projeto de fãs, sem fins lucrativos, criado para fins educacionais e de entretenimento. Pokémon e todos os nomes, imagens e sons associados são marcas registradas e direitos autorais de ©1995-2025 Nintendo, Game Freak e The Pokémon Company.

Créditos das fontes de dados externas utilizadas nos workflows:
* **Dados dos Pokémon:** [PokéAPI (pokeapi.co)](https://pokeapi.co/)
* **Redimensionamento de Imagens:** [images.weserv.nl](https://images.weserv.nl/)

---

## 🎓 Um Projeto do Curso de ADS

Este ecossistema de projetos foi criado pelo curso de **Análise e Desenvolvimento de Sistemas do Unilavras**, para o evento **Unilavras na Praça**. Ele é composto por três repositórios que operam em total sinergia:

1.  **[Pokedex-Praca-Chat](https://github.com/SEU_USUARIO/Pokedex-Praca-Chat)**: O **Gateway de Conversa**, responsável por receber as mensagens do usuário e repassá-las para o nosso agente de IA.
2.  **`Pokedex-Praca-Agente`** (este repositório): O **Cérebro da Operação**, contendo todos os workflows do n8n que processam a lógica, interagem com o usuário e gerenciam os dados.
3.  **[Pokedex-Praca-Telao](https://github.com/joaopaulofcc/Pokedex-Praca-Telao)**: O **Palco Visual**, que recebe os comandos do Agente e exibe as animações de captura em tempo real para o público.

O fluxo de dados (`Usuário ➔ Chat ➔ Agente ➔ Telão`) demonstra na prática a criação de uma aplicação *full-stack* moderna, modular e inteligente.

> **Venha fazer ADS e transforme suas ideias em realidade!**
