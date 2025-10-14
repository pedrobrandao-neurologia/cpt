# Teste de Desempenho Contínuo (CPT) - Análise do Código

Este documento detalha a arquitetura e o funcionamento de uma aplicação web para a realização de um Teste de Desempenho Contínuo (CPT). A aplicação é construída como um único arquivo HTML, utilizando Tailwind CSS para estilização e JavaScript puro para toda a lógica, sem a necessidade de um servidor backend.

---

## 🚀 Como Executar

Para rodar a aplicação, basta seguir estes passos:
1.  Copie todo o código-fonte.
2.  Salve o código em um arquivo com a extensão `.html` (ex: `cpt.html`).
3.  Abra este arquivo em qualquer navegador de internet moderno (como Google Chrome, Firefox, ou Edge).
A aplicação é totalmente funcional no lado do cliente.

---

## 🛠️ Arquitetura e Lógica do Código

O código está estruturado em três camadas principais dentro do arquivo `html`: a estrutura **HTML**, a estilização **CSS** e a lógica de programação **JavaScript**.

### 1. Estrutura HTML

O corpo (`<body>`) da aplicação é organizado como uma "Single Page Application" (SPA) simulada. Em vez de carregar novas páginas, a aplicação simplesmente exibe ou oculta diferentes seções (`<div>`) conforme a interação do usuário.

-   **`<div id="app">`**: O contêiner principal que centraliza todo o conteúdo.
-   **Telas (`<div class="screen">`)**: Cada tela principal da aplicação é um `div` com a classe `.screen`. A visibilidade é controlada pela classe `.active`.
    -   `#setupScreen`: A tela inicial, onde o usuário insere o ID do participante e escolhe a duração do teste.
    -   `#instructionsScreen`: Apresenta as regras do teste (pressionar a barra de espaço para todas as letras, exceto 'X').
    -   `#countdownScreen`: Uma tela de contagem regressiva (3, 2, 1) para preparar o usuário.
    -   `#testScreen`: A tela onde o teste é executado. Mostra o estímulo (a letra), a barra de progresso e a instrução básica.
    -   `#resultsScreen`: Exibida ao final do teste, mostrando um dashboard com as métricas de desempenho e as opções para exportar os resultados.

### 2. Estilização CSS

A aparência da aplicação é controlada por duas fontes:

-   **Tailwind CSS**: Importado via CDN, é o framework principal usado para criar o layout e o design. Classes como `bg-slate-900`, `text-cyan-400`, `flex`, `p-8`, etc., são do Tailwind.
-   **CSS Interno (`<style>`)**:
    -   `@import url(...)`: Importa a fonte "Inter" do Google Fonts.
    -   `.screen { display: none; }`: Garante que todas as telas fiquem ocultas por padrão.
    -   `.screen.active { display: flex; }`: Torna a tela ativa visível, usando `flexbox` para o layout.
    -   `#stimulus { transition: ... }`: Cria um efeito suave de "fade in/out" para a letra que aparece na tela.
    -   `.duration-btn:disabled { ... }`: Estiliza os botões de duração quando estão desabilitados.

### 3. Lógica JavaScript

Toda a inteligência da aplicação reside na tag `<script>`. Ela é organizada em seções lógicas:

#### a. Configurações e Estado

-   **`CONFIG` (Constante)**: Um objeto imutável que armazena os parâmetros de cada versão do teste.
    -   `'3'`, `'6'`, `'14'`: Chaves que representam as durações. Cada uma contém o número total de estímulos (`totalTrials`), a probabilidade de o alvo ('X') aparecer (`targetProbability`) e o número de blocos (`blocks`) para a análise de desempenho.
    -   `targetLetter`: Define o estímulo-alvo ('X').
    -   `ISIs`: "Inter-Stimulus Intervals". Um array com os possíveis intervalos de tempo (em milissegundos) entre o desaparecimento de uma letra e o aparecimento da próxima. A variação desses intervalos é crucial para o teste.
    -   `stimulusDuration`: Quanto tempo (em ms) cada letra permanece visível na tela.

-   **`state` (Variável)**: O "cérebro" da aplicação. Um objeto que guarda todas as informações que mudam durante o uso.
    -   `participantId`: Armazena o ID inserido pelo usuário.
    -   `stimuli`: Será um array com a sequência de letras a ser mostrada, gerada aleatoriamente.
    -   `responses`: Um array que registra cada tentativa, incluindo qual letra foi mostrada, se o usuário respondeu, o tempo de reação, etc. É o "dado bruto" do teste.
    -   `summaryResults`: Armazena os resultados calculados (total de omissões, comissões, etc.).

#### b. Mapeamento do DOM

-   **`screens` e `elements`**: Objetos que guardam referências diretas aos elementos HTML. Isso melhora a performance e a organização, evitando buscas repetitivas no DOM com `document.getElementById()`.

#### c. Fluxo da Aplicação (Funções Principais)

1.  **Início e Configuração**:
    -   O usuário digita um ID. A função `validateSetup()` é chamada a cada letra digitada, habilitando os botões de duração apenas quando o campo de ID não está vazio.
    -   Ao clicar em um botão de duração, `selectDuration()` é acionada. Ela salva o ID e a configuração do teste no `state` e chama `showScreen('instructions')` para exibir a tela de instruções.

2.  **Geração dos Estímulos**:
    -   Quando o teste vai começar, a função `generateStimuli()` é chamada. Ela cria uma lista de objetos, onde cada objeto representa um estímulo (uma letra). A função calcula quantos alvos ('X') devem existir com base na `targetProbability` e os distribui aleatoriamente entre as outras letras. Cada estímulo também recebe um ISI aleatório.

3.  **Execução do Teste**:
    -   A função `runTest()` inicia o processo, que é controlado pela `showNextStimulus()`.
    -   `showNextStimulus()` é uma função recursiva baseada em `setTimeout`. Ela:
        -   Verifica se o teste terminou.
        -   Pega o próximo estímulo da lista `state.stimuli`.
        -   Mostra a letra na tela e inicia um cronômetro (`state.stimulusStartTime = Date.now()`).
        -   Agenda o desaparecimento da letra após `CONFIG.stimulusDuration` (250ms).
        -   Agenda a chamada para si mesma (para mostrar a próxima letra) após o ISI do estímulo atual.
        -   Se o usuário não responder a tempo (antes da próxima letra aparecer), a função `recordResponse(false, null)` é chamada, registrando uma **omissão**.

4.  **Captura de Resposta**:
    -   `handleKeyPress` e `handleTouch` ouvem por interações do usuário.
    -   Quando uma interação ocorre, `recordResponse(true, reactionTime)` é chamada.
    -   `recordResponse()` para o cronômetro, calcula o tempo de reação, e chama `getResponseType()` para classificar a resposta.
    -   `getResponseType()` usa uma lógica simples para definir o tipo de erro/acerto:
        -   Respondeu ao 'X'? **Erro de Comissão** (impulsividade).
        -   Não respondeu a outra letra? **Erro de Omissão** (desatenção).
        -   Respondeu a outra letra? **Acerto** (`correct_hit`).
        -   Não respondeu ao 'X'? **Acerto** (`correct_rejection`).
    -   Toda essa informação é salva no array `state.responses`.

5.  **Cálculo e Exibição dos Resultados**:
    -   Ao final de todas as tentativas, `endTest()` chama `calculateAndShowResults()`.
    -   Esta função itera sobre o array `state.responses` para contar o total de omissões e comissões.
    -   Ela também calcula a média e o desvio padrão do tempo de reação (Variabilidade do TR) apenas para os acertos (`correct_hit`).
    -   Por fim, preenche a tela `#resultsScreen` com esses dados calculados.

#### d. Funções de Exportação

-   As funções `exportCSV()`, `exportJSON()`, e `exportPDF()` são acionadas pelos botões na tela de resultados.
-   **CSV**: Constrói uma string de texto formatada com vírgulas e a força o download através de um link `<a>` criado dinamicamente.
-   **JSON**: Converte os objetos `state.summaryResults` e `state.responses` em uma string JSON e usa a mesma técnica de download.
-   **PDF**: Utiliza as bibliotecas `jsPDF` e `jspdf-autotable` (importadas no `<head>`). Ela cria um documento PDF em tempo real, adicionando um título, uma tabela formatada para o resumo e uma tabela maior para os dados brutos, e então inicia o download.
