# Observatório de Dados dos Entregadores por Aplicativo

## MVP — Engenharia de Dados

Este repositório apresenta o desenvolvimento de um pipeline de dados voltado à organização e análise de informações sobre entregadores por aplicativo no Brasil. O projeto constitui uma primeira infraestrutura de dados para o desenvolvimento do **Observatório de Dados dos Entregadores por Aplicativo**.

O MVP foi desenvolvido em ambiente Databricks e utiliza microdados da PNAD Contínua referentes ao **4º trimestre de 2022** e ao **3º trimestre de 2024**. A solução organiza o processamento segundo a arquitetura Medallion, com camadas Bronze, Silver e Gold.

---

## 1. Problema e objetivo

O projeto parte da seguinte questão:

**Como construir um pipeline de dados capaz de transformar microdados da PNAD Contínua em uma estrutura organizada, reproduzível e reutilizável para a análise quantitativa dos entregadores por aplicativo no Brasil?**

O objetivo do MVP é desenvolver uma infraestrutura que permita:

- preservar os arquivos utilizados em sua forma original;
- extrair e harmonizar variáveis provenientes de diferentes períodos da PNAD Contínua;
- estabelecer um critério reproduzível para identificação dos entregadores plataformizados;
- utilizar os pesos amostrais na produção das estimativas populacionais;
- estruturar os dados por meio de modelagem dimensional;
- produzir tabelas analíticas destinadas ao consumo dos dados;
- caracterizar os entregadores segundo dimensões sociodemográficas e ocupacionais;
- comparar algumas dessas características com aquelas observadas entre os demais trabalhadores.

A proposta não consiste apenas em produzir indicadores pontuais, mas em construir uma estrutura que possa ser posteriormente ampliada com novos períodos, variáveis e fontes de dados.

---

## 2. Fonte de dados

O projeto utiliza microdados da **Pesquisa Nacional por Amostra de Domicílios Contínua — PNAD Contínua**, considerando:

- **4º trimestre de 2022**;
- **3º trimestre de 2024**.

Os arquivos originais são disponibilizados em formato de largura fixa. Como a posição de determinadas variáveis não permanece necessariamente igual entre os períodos, o pipeline realiza a extração e a harmonização das informações antes de sua utilização analítica.

Os dois arquivos utilizados totalizam **957.869 registros amostrais**:

- 2022: 478.091 registros;
- 2024: 479.778 registros.

---

## 3. Arquitetura do pipeline

O pipeline foi organizado segundo a arquitetura Medallion:

### Bronze

Responsável pela ingestão e preservação dos dados em sua forma original.

Nesta etapa, os arquivos da PNAD Contínua são armazenados sem alterações substantivas, permitindo preservar a fonte utilizada pelo projeto.

### Silver

Responsável pela extração, tratamento e harmonização das variáveis utilizadas na análise.

Entre as operações realizadas estão:

- extração das variáveis dos arquivos de largura fixa;
- compatibilização das posições das variáveis entre 2022 e 2024;
- conversão dos tipos de dados;
- tratamento das categorias selecionadas;
- criação de descrições para variáveis sociodemográficas;
- reconstrução do indicador de utilização de plataformas;
- construção do indicador analítico de entregador plataformizado.

A tabela resultante é:

`silver_entregadores_pnad`

### Gold

Responsável pela modelagem dimensional e pela produção das estruturas destinadas ao consumo analítico.

A Camada Gold preserva o universo harmonizado da PNAD utilizado pelo projeto e o organiza em uma tabela fato relacionada a seis dimensões.

---

## 4. Modelo dimensional

A tabela central do modelo é:

`gold_fact_trabalhador`

Cada linha corresponde a uma observação individual da PNAD Contínua em determinado período. Portanto, um registro da tabela fato não corresponde diretamente a uma pessoa da população brasileira. As estimativas populacionais são produzidas por meio da variável `peso_amostral`.

A tabela fato está relacionada às seguintes dimensões:

- `gold_dim_tempo`
- `gold_dim_sexo`
- `gold_dim_cor_raca`
- `gold_dim_faixa_etaria`
- `gold_dim_posicao_ocupacao`
- `gold_dim_atividade_principal`

O modelo também preserva indicadores relacionados à utilização de plataformas e à identificação dos entregadores plataformizados.

---

## 5. Tabelas analíticas

Além da tabela fato e das dimensões, a Camada Gold contém estruturas agregadas destinadas às análises:

- `gold_indicadores_gerais`
- `gold_perfil_sexo`
- `gold_perfil_cor_raca`
- `gold_perfil_faixa_etaria`
- `gold_perfil_posicao_ocupacao`
- `gold_perfil_atividade_principal`

Essas tabelas apresentam, conforme a dimensão analisada, registros amostrais, estimativas ponderadas e percentuais ponderados.

---

## 6. Estrutura do repositório

O projeto está organizado em três notebooks principais:

`01_ingestao_pnad_bronze`

Realiza a ingestão e a preservação dos arquivos utilizados pelo projeto.

`02_transformacao_pnad_silver`

Realiza a extração das variáveis, harmonização entre os períodos, tratamento dos dados e construção dos indicadores analíticos.

`03_construcao_pnad_gold`

Realiza a modelagem dimensional, construção das tabelas Gold, verificações de qualidade e análises dos resultados.

---

## 7. População analítica

O projeto diferencia o registro bruto de utilização de aplicativo para entrega da população definida analiticamente como **entregadores plataformizados**.

A identificação considera conjuntamente as variáveis relacionadas ao uso de plataformas, à atividade principal e à posição na ocupação, conforme as regras implementadas no pipeline.

Após a aplicação desse critério, foram identificados:

| Período | Registros amostrais | Estimativa ponderada |
|---|---:|---:|
| 4º trimestre de 2022 | 691 | 445.867 |
| 3º trimestre de 2024 | 778 | 487.285 |

Os valores ponderados representam estimativas populacionais e não devem ser confundidos com a quantidade de registros existentes nos microdados.

---

## 8. Principais análises

O pipeline permite analisar os entregadores plataformizados segundo:

- sexo;
- cor ou raça;
- faixa etária;
- posição na ocupação;
- atividade principal.

Também foram realizadas comparações entre entregadores plataformizados e os demais trabalhadores pertencentes ao universo ocupacional definido no projeto, considerando sexo, faixa etária e posição na ocupação.

Entre os resultados observados, destaca-se a predominância masculina entre os entregadores, a concentração nas faixas entre 25 e 44 anos e a elevada participação da categoria conta própria.

A classificação como trabalhador por conta própria é tratada como uma categoria estatística da pesquisa e não como evidência suficiente de autonomia substantiva no exercício da atividade.

---

## 9. Qualidade e validação

O pipeline incorpora verificações realizadas em diferentes etapas do processamento, incluindo:

- conferência da quantidade de registros ingeridos;
- validação da harmonização entre os períodos;
- controle da população de entregadores identificada;
- verificação das chaves do modelo dimensional;
- conferência das estimativas ponderadas;
- comparação dos totais produzidos pelas diferentes tabelas analíticas;
- validação da persistência das tabelas.

Essas verificações buscam avaliar a consistência interna do pipeline e não eliminam as limitações próprias das fontes e dos recortes utilizados.

---

## 10. Limitações

A comparação utiliza o 4º trimestre de 2022 e o 3º trimestre de 2024. Por corresponderem a períodos distintos, as diferenças observadas são interpretadas de forma descritiva e não, isoladamente, como evidência de uma trajetória temporal contínua.

O MVP também trabalha com um conjunto delimitado de variáveis. Outras dimensões poderão ser incorporadas em versões posteriores do pipeline.

Os códigos de atividade principal foram preservados na estrutura Gold. Sua utilização com denominações substantivas permanece condicionada à incorporação de uma correspondência classificatória devidamente documentada.

---

## 11. Possibilidades de evolução

O projeto foi estruturado para permitir sua ampliação sem reconstrução integral do pipeline.

Entre as possibilidades de desenvolvimento estão:

- incorporação de novos períodos;
- inclusão de novas variáveis e dimensões analíticas;
- enriquecimento da classificação das atividades;
- desenvolvimento de dashboards;
- incorporação de outras fontes de dados;
- ampliação do Observatório de Dados dos Entregadores por Aplicativo.

---


## 12. Estrutura da solução

Fluxo simplificado do projeto:

PNAD Contínua  
↓  
**Bronze — ingestão e preservação**  
↓  
**Silver — extração, tratamento e harmonização**  
↓  
**Gold — modelagem dimensional e tabelas analíticas**  
↓  
**Indicadores e análises**  
↓  
**Observatório de Dados dos Entregadores por Aplicativo**