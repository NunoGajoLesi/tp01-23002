# Análise e Visualização de Dados Telemétricos de Fórmula 1

**Autor**: Nuno da Cunha Faria Gajo a23002

---

## Índice

1. [Ficheiros do projeto](#ficheiros-do-projeto)
2. [Ferramentas utilizadas](#ferramentas-utilizadas)
3. [Antes de executar](#antes-de-executar)
4. [Como executar](#como-executar)

---

## Ficheiros do projeto

```text
.
├── data
│   ├── input
│   │   ├── car_data_sujo.csv
│   │   └── singapore_gp_weather_sujo.json
│   └── output
│       ├── Buscar_api.csv
│       ├── car_data_limpo.csv
│       └── WeatherLimpo.csv
├── dataintegration
│   ├── DashboardF1.json
│   ├── Enviar_email_job_doCarData.kjb
│   ├── Enviar_email_job_doWheater.kjb
│   ├── FETCHAPI.ktr
│   ├── Limpar Car Data.ktr
│   └── limpar_json.ktr
└── doc
    └── 23002_doc.pdf
```

### `data/input` – dados de entrada

| Ficheiro | Descrição |
|---|---|
| `car_data_sujo.csv` | Telemetria do piloto 55 (Carlos Sainz), sessão 9159, com erros introduzidos manualmente. Tem 18620 linhas: 150 duplicadas, 60 com texto em `speed`, 40 com `speed` negativa, 30 com `speed` fora do intervalo, 50 com datas mal formatadas, 40 com `rpm` em branco, e a ordem das linhas baralhada. É a entrada de `Limpar Car Data.ktr`. |
| `singapore_gp_weather_sujo.json` | Dados meteorológicos do mesmo Grande Prémio, com 27 registos (3 reais e 21 simulados) e erros introduzidos: texto na temperatura do ar, humidade fora de 0–100, data mal formatada, temperatura da pista nula, vento negativo e 3 duplicados. É a entrada de `limpar_json.ktr`. |

### `data/output` – dados de saída

| Ficheiro | Descrição |
|---|---|
| `Buscar_api.csv` | Telemetria do piloto 44 obtida diretamente da API OpenF1 (mesma sessão, 9159), gerada por `FETCHAPI.ktr`. Os dados já vêm limpos, por isso não passam por validações. |
| `car_data_limpo.csv` | Telemetria limpa: sem duplicados nem valores inválidos, filtrada e ordenada por data (18250 linhas). Saída de `Limpar Car Data.ktr` e entrada do dashboard do Node-RED. |
| `WeatherLimpo.csv` | Meteorologia limpa (19 registos), convertida de JSON para CSV. Saída de `limpar_json.ktr`. |

### `dataintegration` – transformações, jobs e dashboard

| Ficheiro | Descrição |
|---|---|
| `DashboardF1.json` | Fluxo do Node-RED (exportado) que lê `car_data_limpo.csv` e mostra o dashboard: gráficos da velocidade, das rotações, do acelerador e da mudança, e um indicador da velocidade máxima. |
| `Enviar_email_job_doCarData.kjb` | Job do Kettle que corre `Limpar_Car_Data.ktr` e envia por email o resumo da limpeza (percentagem de dados corretos, mal formatados e duplicados). |
| `Enviar_email_job_doWheater.kjb` | Job do Kettle que corre `limpar_json.ktr` e envia por email o resumo da limpeza dos dados meteorológicos. |
| `FETCHAPI.ktr` | Transformação que obtém dados da API OpenF1 (*Generate rows*, *REST client*, *JSON input*) e grava-os em CSV. |
| `Limpar_Car_Data.ktr` | Transformação de limpeza da telemetria: conversão de tipos, filtro de velocidades e rotações, validação das datas, remoção de duplicados, ordenação e exportação para CSV. Inclui o bloco que conta as linhas e calcula as percentagens do resumo. |
| `limpar_json.ktr` | Transformação de limpeza dos dados meteorológicos (humidade entre 0 e 100, vento não negativo, temperatura da pista não nula), com o mesmo bloco de resumo, e conversão para CSV. |

### `doc` – documentação

| Ficheiro | Descrição |
|---|---|
| `23002_doc.pdf` | Relatório final do trabalho. |

---

## Ferramentas utilizadas

| Ferramenta | Para quê |
|---|---|
| **Pentaho Data Integration (Kettle) 10.2**, com o Spoon | Transformações (`.ktr`) e jobs (`.kjb`). Precisa de Java instalado, na versão exigida por esta versão do Kettle. |
| **Node.js** (versão LTS) e **Node-RED** | Execução do fluxo do dashboard. |
| **`@flowfuse/node-red-dashboard`** (Dashboard 2.0) | Nodes `ui-chart` e `ui-gauge` do dashboard. |

---

## Antes de executar

### 1. Instalar as ferramentas

- Instalar o **Pentaho Data Integration** e abrir o Spoon (`Spoon.bat`).
- Instalar o **Node.js** (versão LTS) e, a seguir, o Node-RED:

```bash
npm install -g --unsafe-perm node-red
```

### 2. Configurar o email

Nos dois jobs (`.kjb`), abrir a entrada **Mail** e preencher com a conta de quem executa:

- **Addresses**: destinatário e remetente.
- **Server**: substituir o utilizador e a palavra-passe.

### 3. Instalar o Dashboard 2.0 no Node-RED

Antes de importar o fluxo, é preciso instalar os nodes do dashboard: no editor do Node-RED, **Menu > Manage palette > Install**, pesquisar `@flowfuse/node-red-dashboard` e instalar. Sem este pacote, os nodes `ui-chart` e `ui-gauge` do fluxo aparecem como desconhecidos.

---

## Como executar

### 1. Limpar a telemetria (CSV) e enviar o resumo por email

1. No Spoon, **File > Open** e abrir `dataintegration/Enviar_email_job_doCarData.kjb`.
2. Carregar em **Run** (triângulo verde) e confirmar.
3. O job corre `Limpar_Car_Data.ktr`, grava `data/output/car_data_limpo.csv` e envia o email com as percentagens.

> Executar o **job**, e não a transformação sozinha: as variáveis com os resultados do resumo só chegam ao email quando a transformação corre dentro do job.

### 2. Limpar os dados meteorológicos (JSON) e enviar o resumo por email

1. No Spoon, abrir `dataintegration/Enviar_email_job_doWheater.kjb`.
2. Carregar em **Run**.
3. O job corre `limpar_json.ktr`, grava `data/output/WeatherLimpo.csv` e envia o email com as percentagens.

### 3. Obter dados da API

1. No Spoon, abrir `dataintegration/FETCHAPI.ktr`.
2. Carregar em **Run**.
3. A transformação grava `data/output/Buscar_api.csv`, com a telemetria do piloto 44. Para outro piloto, alterar o `driver_number` no URL do passo **REST client**.

### 4. Ver o dashboard no Node-RED

Executar primeiro o ponto 1, que gera o `car_data_limpo.csv` de que o dashboard precisa.

1. Arrancar o Node-RED: abrir um terminal e correr `node-red`.
2. Abrir o editor em `http://127.0.0.1:1880`.
3. **Menu > Import**, escolher `dataintegration/DashboardF1.json` e importar.
4. No node **file in**, corrigir o caminho para `data/output/car_data_limpo.csv` (ver [Antes de executar](#antes-de-executar)).
5. Carregar em **Deploy**.
6. Carregar no botão à esquerda do node **inject** para ler os dados.
7. Abrir o dashboard em `http://127.0.0.1:1880/dashboard`.

> O node **csv** do fluxo usa o ponto e vírgula (`;`) como separador, porque é o que o Kettle usa por defeito ao gravar o CSV. Se esse separador for alterado no Kettle, tem de ser alterado também no node **csv**.
