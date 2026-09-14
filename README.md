**Pontifícia Universidade Católica do Rio de Janeiro (PUC-Rio)**  

**Autor:** Carlos Eduardo Pereira Viana

**Matrícula:** 4052026000806  
**Data:** Setembro/2026
<br><br>
<br><br>

**MVP — Engenharia de Dados**


# MVP de Engenharia de Dados
## Observatório de Dados dos Entregadores por Aplicativo 

Este projeto apresenta um **MVP de Engenharia de Dados** desenvolvido para a disciplina de Engenharia de Dados da PUC-Rio. A solução parte de uma questão vinculada à análise do trabalho por plataformas e busca transformá-la em um problema de Engenharia de Dados: como organizar microdados estatísticos de forma rastreável, reproduzível e extensível para produzir indicadores sobre os entregadores por aplicativo?

Para isso, foi desenvolvido um pipeline no **Databricks**, organizado segundo a arquitetura Medallion, com camadas **Bronze, Silver e Gold**. O fluxo contempla ingestão, tratamento, harmonização, modelagem dimensional, controles de qualidade e disponibilização dos dados para análise.

A primeira versão do produto utiliza microdados da **Pesquisa Nacional por Amostra de Domicílios Contínua (PNAD Contínua)** referentes ao **4º trimestre de 2022** e ao **3º trimestre de 2024** e constitui a infraestrutura inicial de um **Observatório de Dados dos Entregadores por Aplicativo**.

---

## Índice

1. [Contexto de Negócio e Perguntas](#1-contexto-de-negócio-e-perguntas-etapas-2-e-41)
2. [Carga dos Dados](#2-carga-dos-dados-etapa-42)
3. [Modelagem e Catálogo de Dados](#3-modelagem-e-catálogo-de-dados-etapa-43)
4. [Pipeline de Dados](#4-pipeline-de-dados-etapa-44)
5. [Qualidade de Dados](#5-qualidade-de-dados-etapa-45)
6. [Análise de Dados](#6-análise-de-dados-etapa-45)
7. [Autoavaliação](#7-autoavaliação)
8. [Como Reproduzir o Projeto](#8-como-reproduzir-o-projeto)
9. [Evidências da Solução](#9-evidências-da-solução)

---

# 1. Contexto de Negócio e Perguntas (Etapas 2 e 4.1)

## 1.1 Contexto e problema

O trabalho intermediado por plataformas digitais constitui uma modalidade de atividade cuja análise quantitativa envolve desafios relacionados não apenas à disponibilidade dos dados, mas também à forma como esses dados são identificados, tratados e organizados para responder a perguntas específicas.

Neste MVP, o problema de Engenharia de Dados consiste em transformar os microdados da **PNAD Contínua** em uma estrutura analítica capaz de identificar e caracterizar os entregadores plataformizados, preservando simultaneamente as informações necessárias para produzir estimativas populacionais e estabelecer comparações com os demais trabalhadores.

Os arquivos originais utilizados no projeto não constituem uma base analítica previamente organizada para essa finalidade. A construção dos indicadores exige selecionar variáveis distribuídas nos microdados, considerar diferenças de posição entre os arquivos dos períodos analisados, aplicar regras de identificação da população de interesse, preservar o peso amostral e disponibilizar os resultados em estruturas adequadas ao consumo analítico.

O problema pode ser sintetizado pela seguinte questão:

> **Como construir um pipeline de dados capaz de transformar os microdados da PNAD Contínua em uma estrutura organizada, rastreável e reutilizável para a análise quantitativa dos entregadores por aplicativo?**

A proposta não consiste apenas em produzir resultados pontuais, mas em estruturar o processamento de forma que as etapas entre o dado de origem e os indicadores finais possam ser identificadas, verificadas e posteriormente ampliadas.

## 1.2 Objetivo

O objetivo geral do MVP é **desenvolver um pipeline de Engenharia de Dados para ingestão, transformação, harmonização, modelagem e análise dos microdados da PNAD Contínua relacionados ao trabalho de entregadores por aplicativo**.

A solução foi organizada segundo a arquitetura **Medallion**, separando o processamento em três camadas:

- **Bronze:** ingestão e preservação dos dados de origem;
- **Silver:** extração, tipagem, harmonização e construção das variáveis necessárias à análise;
- **Gold:** modelagem dimensional, persistência de estruturas destinadas ao consumo analítico, controles de qualidade e produção dos indicadores.

Como produto, o MVP busca estabelecer uma primeira infraestrutura de dados para o desenvolvimento de um **Observatório de Dados dos Entregadores por Aplicativo**, permitindo que novos períodos, variáveis e fontes possam ser incorporados posteriormente sem a necessidade de reconstrução integral do pipeline.

## 1.3 Perguntas do MVP

A construção do pipeline foi orientada pelas seguintes perguntas:

1. **Qual é a dimensão estimada da população de entregadores plataformizados nos períodos analisados?**

2. **Como os entregadores plataformizados se distribuem segundo sexo, cor ou raça e faixa etária?**

3. **Como os entregadores plataformizados se distribuem segundo sua posição na ocupação?**

4. **Quais atividades principais aparecem associadas aos entregadores identificados nos microdados?**

5. **Como a composição dos entregadores plataformizados se diferencia daquela observada entre os demais trabalhadores, especialmente segundo sexo, faixa etária e posição na ocupação?**

As perguntas orientaram as decisões tomadas ao longo do pipeline, desde a seleção das variáveis na camada Silver até a definição da granularidade, das dimensões, dos marts analíticos e das análises produzidas na camada Gold.

Nem todas as dimensões disponíveis nos microdados foram incorporadas ao MVP. A seleção foi delimitada pelas perguntas definidas para esta primeira versão e pela possibilidade de validar adequadamente as variáveis utilizadas.

## 1.4 Fonte e estrutura dos dados brutos

A fonte utilizada é a **Pesquisa Nacional por Amostra de Domicílios Contínua (PNAD Contínua)**.

O MVP utiliza dois arquivos de microdados:

| Período de referência | Registros no arquivo bruto |
|---|---:|
| 4º trimestre de 2022 | 478.091 |
| 3º trimestre de 2024 | 479.778 |
| **Total processado** | **957.869** |

Os microdados são disponibilizados em arquivos de **largura fixa**, nos quais a extração das variáveis depende da posição e do tamanho definidos para cada campo.

Essa característica foi particularmente relevante para o pipeline porque algumas variáveis utilizadas pelo projeto ocupam posições distintas nos arquivos dos dois períodos. Por essa razão, a camada Silver utiliza um mapeamento específico para cada ano antes de realizar a harmonização das estruturas.

Entre as informações selecionadas para o MVP estão:

| Informação | Utilização no projeto |
|---|---|
| Peso amostral | Produção das estimativas ponderadas |
| Sexo | Caracterização sociodemográfica |
| Idade | Construção das faixas etárias |
| Cor ou raça | Caracterização sociodemográfica |
| Posição na ocupação | Caracterização ocupacional |
| Atividade principal | Identificação e caracterização da atividade |
| Variáveis sobre uso de plataformas | Identificação do trabalho intermediado por plataformas |
| Informação sobre entrega | Identificação dos entregadores |

A camada Silver preserva o conjunto harmonizado dos **957.869 registros**, em vez de restringir fisicamente a base aos entregadores. Essa decisão permite que a camada Gold utilize o mesmo universo tanto para a caracterização da população de interesse quanto para análises comparativas com os demais trabalhadores.

## 1.5 Identificação da população analítica

A identificação dos entregadores não foi realizada apenas a partir da presença de uma resposta isolada sobre atividades de entrega.

O pipeline reconstrói a variável derivada utilizada para identificação do trabalho por plataformas a partir da combinação das informações necessárias e, posteriormente, constrói o indicador analítico `entregador_plataformizado`.

A aplicação dessa regra resultou nos seguintes controles:

| Período | Registros amostrais identificados | Estimativa ponderada |
|---|---:|---:|
| 4º trimestre de 2022 | 691 | aproximadamente 445,9 mil |
| 3º trimestre de 2024 | 778 | aproximadamente 487,3 mil |

A distinção entre **registros amostrais** e **estimativas ponderadas** é mantida ao longo de todo o projeto. Uma linha dos microdados corresponde a uma observação amostral da PNAD Contínua e não deve ser interpretada diretamente como uma pessoa na população. As estimativas apresentadas nas análises utilizam o peso amostral correspondente.

## 1.6 Condições de uso dos dados



Os microdados utilizados no projeto são disponibilizados publicamente pelo
Instituto Brasileiro de Geografia e Estatística (IBGE). A instituição
disponibiliza os microdados da PNAD Contínua para uso público, após tratamento
destinado à preservação do sigilo das informações individualizadas.

A disponibilização dos dados pelo IBGE está inserida no contexto da Política de
Dados Abertos do Poder Executivo Federal. Dessa forma, os microdados públicos
podem ser utilizados para a produção de indicadores e análises pelos usuários,
observadas as condições de disseminação e preservação do sigilo estabelecidas
pela instituição.

Neste projeto, os dados são utilizados exclusivamente para fins acadêmicos e
analíticos, com identificação da PNAD Contínua e do IBGE como fonte das
informações.

## Referências institucionais

- IBGE. Pesquisa Nacional por Amostra de Domicílios Contínua — Microdados trimestrais. Disponível em: https://ftp.ibge.gov.br/Trabalho_e_Rendimento/Pesquisa_Nacional_por_Amostra_de_Domicilios_continua/Trimestral/Microdados/

- IBGE. PNAD Contínua — Documentação dos microdados. Disponível em: https://ftp.ibge.gov.br/Trabalho_e_Rendimento/Pesquisa_Nacional_por_Amostra_de_Domicilios_continua/Trimestral/Microdados/Documentacao/

- IBGE. Infraestrutura Nacional de Dados Abertos — INDA. Disponível em: https://www.ibge.gov.br/acesso-informacao/acoes-e-programas/infraestrutura-nacional-de-dados-abertos-inda.html?lang=pt-BR

- IBGE. Plano de Dados Abertos 2024–2025. Disponível em: https://www.ibge.gov.br/np_download/novoportal/documentos_institucionais/Plano_de_Dados_Abertos_IBGE_2024_2025.pdf



# 2. Carga dos Dados (Etapa 4.2)

## 2.1 Obtenção e organização dos microdados

Os dados utilizados no MVP correspondem aos microdados da **Pesquisa Nacional por Amostra de Domicílios Contínua (PNAD Contínua)**, produzida pelo Instituto Brasileiro de Geografia e Estatística (IBGE).

Foram utilizados dois arquivos referentes aos períodos selecionados para o projeto:

| Período | Arquivo utilizado | Registros |
|---|---|---:|
| 4º trimestre de 2022 | `PNADC_2022_trimestre4.txt` | 478.091 |
| 3º trimestre de 2024 | `PNADC_2024_trimestre3.txt` | 479.778 |
| **Total** | — | **957.869** |

Os arquivos foram disponibilizados no ambiente de armazenamento do Databricks e organizados segundo o período de referência, preservando os microdados em seu formato original antes das transformações realizadas nas etapas posteriores.

No ambiente utilizado pelo projeto, os arquivos extraídos foram organizados nos seguintes diretórios:


/Volumes/workspace/bronze/pnad_raw/2022/extracted/PNADC_2022_trimestre4.txt

/Volumes/workspace/bronze/pnad_raw/2024/extracted/PNADC_2024_trimestre3.txt

# 2. Carga dos Dados (Etapa 4.2)

## 2.1 Obtenção e organização dos microdados

Os dados utilizados no MVP correspondem aos microdados da **Pesquisa Nacional por Amostra de Domicílios Contínua (PNAD Contínua)**, produzida pelo Instituto Brasileiro de Geografia e Estatística (IBGE).

Foram utilizados dois arquivos referentes aos períodos selecionados para o projeto:

| Período | Arquivo utilizado | Registros |
|---|---|---:|
| 4º trimestre de 2022 | `PNADC_2022_trimestre4.txt` | 478.091 |
| 3º trimestre de 2024 | `PNADC_2024_trimestre3.txt` | 479.778 |
| **Total** | — | **957.869** |

Os arquivos foram disponibilizados no ambiente de armazenamento do Databricks e organizados segundo o período de referência, preservando os microdados em seu formato original antes das transformações realizadas nas etapas posteriores.

No ambiente utilizado pelo projeto, os arquivos extraídos foram organizados nos seguintes diretórios:

`/Volumes/workspace/bronze/pnad_raw/2022/extracted/PNADC_2022_trimestre4.txt`

`/Volumes/workspace/bronze/pnad_raw/2024/extracted/PNADC_2024_trimestre3.txt`

Essa organização permite distinguir os arquivos de origem segundo o período analisado e estabelece a entrada da camada Bronze do pipeline.

## 2.2 Características da carga

Os microdados utilizados possuem estrutura de **largura fixa**. Diferentemente de arquivos tabulares convencionais, como CSV, as variáveis não são separadas por delimitadores. Cada informação ocupa posições previamente determinadas dentro de cada registro.

Por essa razão, a etapa de carga preserva inicialmente cada linha do arquivo em sua forma bruta. A interpretação das posições e a extração das variáveis necessárias são realizadas posteriormente, na camada Silver.

Essa separação foi adotada para evitar que regras analíticas fossem incorporadas prematuramente à etapa de ingestão e para manter uma distinção entre as responsabilidades das três camadas:

**Dado de origem → Bronze → Silver → Gold**

- **Bronze:** preservação e validação inicial;
- **Silver:** extração, tipagem e harmonização;
- **Gold:** modelagem e consumo analítico.

A camada Bronze, portanto, não realiza a identificação dos entregadores nem produz os indicadores utilizados na análise. Sua função é estabelecer a entrada dos microdados no pipeline e verificar sua disponibilidade para as transformações subsequentes.

## 2.3 Implementação da camada Bronze

A carga e a validação inicial dos dados estão documentadas no notebook:

`01_ingestao_pnad_bronze`

O notebook foi mantido separado das etapas de transformação e modelagem, permitindo que cada camada do pipeline possua uma responsabilidade específica.

Na Bronze são realizadas as seguintes operações:

1. definição e verificação dos diretórios utilizados no projeto;
2. inspeção dos arquivos de origem;
3. extração dos arquivos utilizados;
4. verificação da existência dos microdados extraídos;
5. leitura inicial dos arquivos;
6. conferência da quantidade de registros.

A validação final da carga identificou:

- **478.091 registros** no arquivo do 4º trimestre de 2022;
- **479.778 registros** no arquivo do 3º trimestre de 2024;
- **957.869 registros** considerados conjuntamente no pipeline.

Essas contagens funcionam como controles iniciais para as etapas seguintes e permitem verificar posteriormente se a harmonização entre os períodos preservou o universo de registros processado.

## 2.4 Separação entre ingestão e transformação

Uma decisão adotada no desenvolvimento do MVP foi manter a camada Bronze restrita às operações relacionadas à ingestão e à validação inicial dos arquivos.

Durante o desenvolvimento do pipeline, algumas verificações exploratórias foram necessárias para compreender a estrutura dos microdados. Na versão final, entretanto, as operações relacionadas à seleção de variáveis, reconstrução de indicadores, tipagem e criação da população analítica foram concentradas na camada Silver.

Essa separação busca tornar mais clara a responsabilidade de cada etapa e facilitar a reprodução do processamento:

| Camada | Responsabilidade principal |
|---|---|
| Bronze | Ingestão e preservação dos dados de origem |
| Silver | Extração, tratamento, tipagem e harmonização |
| Gold | Modelagem, agregação, validação e análise |

## 2.5 Rastreabilidade da carga

A organização adotada permite acompanhar o percurso dos dados desde os arquivos originais até as estruturas destinadas à análise.

Os microdados permanecem identificados pelo período de referência na camada Bronze. Na Silver, as estruturas de 2022 e 2024 são harmonizadas e recebem a variável `ano_pnad`, que preserva a identificação de sua origem temporal. Na Gold, essa informação é incorporada ao modelo dimensional por meio da dimensão de tempo.

Dessa forma, mesmo após as transformações realizadas pelo pipeline, permanece possível distinguir a origem temporal das observações utilizadas nas análises.

## 2.6 Evidências da carga

Para documentar a execução desta etapa, serão incorporadas ao repositório evidências visuais do ambiente Databricks, contemplando a organização dos arquivos, a execução do notebook da camada Bronze e os controles de quantidade de registros.

**[INSERIR SCREENSHOT DA ORGANIZAÇÃO/CARGA DOS ARQUIVOS NO DATABRICKS]**

**[INSERIR SCREENSHOT DA VALIDAÇÃO DAS CONTAGENS DE 2022 E 2024]**

O código correspondente a esta etapa está disponível no notebook `01_ingestao_pnad_bronze`.

# 3. Modelagem e Catálogo de Dados (Etapa 4.3)

## 3.1 Estratégia de modelagem

A camada Gold foi estruturada a partir de um **modelo dimensional em esquema estrela**, composto por uma tabela fato central e seis dimensões descritivas.

A opção pelo modelo dimensional procura separar as observações utilizadas nas análises das características empregadas para descrevê-las, facilitando a construção de agregações, comparações e indicadores.

O modelo final é composto pelas seguintes tabelas:

| Tipo | Tabela |
|---|---|
| Fato | `gold_fact_trabalhador` |
| Dimensão | `gold_dim_tempo` |
| Dimensão | `gold_dim_sexo` |
| Dimensão | `gold_dim_cor_raca` |
| Dimensão | `gold_dim_faixa_etaria` |
| Dimensão | `gold_dim_posicao_ocupacao` |
| Dimensão | `gold_dim_atividade_principal` |

Além da tabela fato e das dimensões, foram construídos marts analíticos derivados da estrutura Gold, destinados ao consumo dos principais indicadores do projeto.

## 3.2 Granularidade da tabela fato

A granularidade constitui uma decisão central da modelagem.

No modelo desenvolvido:

> **Uma linha da tabela `gold_fact_trabalhador` corresponde a uma observação individual da PNAD Contínua em determinado período de referência.**

A tabela fato possui **957.869 registros**, correspondentes ao conjunto harmonizado das observações processadas no 4º trimestre de 2022 e no 3º trimestre de 2024.

É importante distinguir a unidade armazenada na tabela fato da população estimada. Cada registro corresponde a uma **observação amostral da PNAD Contínua**, e não diretamente a uma pessoa na população brasileira. Para produzir estimativas populacionais, o modelo preserva a variável `peso_amostral`.

A tabela fato também preserva o indicador `entregador_plataformizado`, permitindo identificar a população de interesse sem restringir fisicamente a Gold apenas aos entregadores. Essa decisão possibilita produzir tanto indicadores específicos desse grupo quanto comparações com os demais trabalhadores pertencentes ao universo analítico.

## 3.3 Estrutura do esquema estrela

A tabela central do modelo é:

`gold_fact_trabalhador`

Ela se relaciona às seguintes dimensões:

- `gold_dim_tempo`;
- `gold_dim_sexo`;
- `gold_dim_cor_raca`;
- `gold_dim_faixa_etaria`;
- `gold_dim_posicao_ocupacao`;
- `gold_dim_atividade_principal`.

As relações são estabelecidas pelas respectivas chaves dimensionais, entre elas:

- `id_tempo`;
- `id_sexo`;
- `id_cor_raca`;
- `id_faixa_etaria`;
- `id_posicao_ocupacao`;
- `id_atividade_principal`.

A tabela fato concentra as observações, as chaves necessárias aos relacionamentos, o `peso_amostral` e os indicadores empregados nas análises. As dimensões organizam os atributos descritivos utilizados para segmentar e interpretar os resultados.

## 3.4 Diagrama Entidade-Relacionamento

O modelo dimensional foi documentado por meio de um **Diagrama Entidade-Relacionamento (DER)**, representando a tabela `gold_fact_trabalhador`, as seis dimensões e suas relações.

O diagrama foi construído durante o desenvolvimento da camada Gold e armazenado no ambiente Databricks como:

`/Volumes/workspace/gold/artefatos_projeto/der_gold_entregadores.png`

**[INSERIR AQUI O DER DO MODELO GOLD]**

A representação visual complementa o catálogo de dados e permite observar a organização do esquema estrela utilizado no MVP.

## 3.5 Catálogo de Dados

O catálogo documenta as tabelas que compõem a camada Gold, sua função no modelo, granularidade, principais campos, domínios e linhagem.

### 3.5.1 `gold_fact_trabalhador`

**Descrição:** tabela fato central do modelo dimensional.

**Granularidade:** uma observação individual da PNAD Contínua em determinado período de referência.

**Quantidade de registros:** **957.869**.

**Origem:** `silver_entregadores_pnad`.

**Linhagem:** construída a partir da base harmonizada da camada Silver, com atribuição das chaves correspondentes às dimensões da camada Gold.

| Campo | Tipo lógico | Descrição | Domínio/Função |
|---|---|---|---|
| `id_tempo` | Inteiro | Chave de relacionamento com `gold_dim_tempo` | Período de referência |
| `id_sexo` | Inteiro | Chave de relacionamento com `gold_dim_sexo` | Categoria de sexo |
| `id_cor_raca` | Inteiro | Chave de relacionamento com `gold_dim_cor_raca` | Categoria de cor ou raça |
| `id_faixa_etaria` | Inteiro | Chave de relacionamento com `gold_dim_faixa_etaria` | Faixa etária |
| `id_posicao_ocupacao` | Inteiro | Chave de relacionamento com `gold_dim_posicao_ocupacao` | Posição na ocupação |
| `id_atividade_principal` | Chave/código | Chave de relacionamento com `gold_dim_atividade_principal` | Atividade principal |
| `peso_amostral` | Decimal | Peso amostral utilizado nas estimativas populacionais | Valor positivo derivado de `V1028` |
| `entregador_plataformizado` | Indicador | Identifica os entregadores plataformizados | Indicador analítico derivado na Silver |

A preservação dos demais trabalhadores na tabela fato é intencional. Em vez de produzir uma Gold restrita aos entregadores, o modelo mantém o universo harmonizado necessário às análises comparativas.

### 3.5.2 `gold_dim_tempo`

**Descrição:** dimensão responsável pela identificação dos períodos de referência utilizados no MVP.

**Quantidade de registros:** **2**.

**Origem:** variável `ano_pnad`, preservada na camada Silver.

| Campo/Função | Descrição | Domínio |
|---|---|---|
| `id_tempo` | Chave da dimensão temporal | Dois períodos modelados |
| Período de referência | Identificação temporal utilizada nas análises | 4º trimestre de 2022 e 3º trimestre de 2024 |

A dimensão permite preservar a origem temporal das observações mesmo após a harmonização das bases.

### 3.5.3 `gold_dim_sexo`

**Descrição:** dimensão utilizada para organizar as categorias de sexo presentes nos microdados.

**Quantidade de registros:** **2**.

**Origem:** `sexo_desc`, construída na camada Silver a partir da variável de sexo dos microdados.

| Campo/Função | Descrição | Domínio |
|---|---|---|
| `id_sexo` | Chave da dimensão | Categorias válidas |
| Descrição de sexo | Categoria utilizada nas análises | Homem; Mulher |

### 3.5.4 `gold_dim_cor_raca`

**Descrição:** dimensão destinada à organização das categorias de cor ou raça utilizadas nas análises.

**Quantidade de registros:** **6**.

**Origem:** `cor_raca_desc`, construída na camada Silver a partir da variável `cor_raca`.

| Campo/Função | Descrição | Domínio |
|---|---|---|
| `id_cor_raca` | Chave da dimensão | Categorias da variável |
| Descrição de cor ou raça | Categoria utilizada nas análises | Branca; Preta; Amarela; Parda; Indígena; Ignorado |

A dimensão preserva as categorias existentes na estrutura harmonizada, inclusive a categoria destinada aos registros classificados como ignorados.

### 3.5.5 `gold_dim_faixa_etaria`

**Descrição:** dimensão construída para organizar a variável `idade_anos` em intervalos utilizados nas análises.

**Quantidade de registros:** **8**.

**Origem:** `idade_anos`, construída na camada Silver a partir da idade registrada nos microdados.

| Campo/Função | Descrição | Domínio |
|---|---|---|
| `id_faixa_etaria` | Chave da dimensão | Faixas etárias modeladas |
| Descrição da faixa | Intervalo etário utilizado nas análises | 14–17; 18–24; 25–34; 35–44; 45–54; 55–64; 65+; categoria técnica |

As sete faixas substantivas utilizadas nas análises são:

- 14 a 17 anos;
- 18 a 24 anos;
- 25 a 34 anos;
- 35 a 44 anos;
- 45 a 54 anos;
- 55 a 64 anos;
- 65 anos ou mais.

A oitava categoria possui função técnica para valores não classificados e não é utilizada na interpretação substantiva das distribuições etárias.

### 3.5.6 `gold_dim_posicao_ocupacao`

**Descrição:** dimensão destinada à classificação da posição na ocupação dos trabalhadores.

**Quantidade de registros:** **8**.

**Origem:** variável `V4012`, extraída e harmonizada na camada Silver.

| Campo/Função | Descrição |
|---|---|
| `id_posicao_ocupacao` | Chave da dimensão |
| Descrição da posição | Categoria de posição na ocupação utilizada nas análises |

As categorias de origem correspondem a:

1. trabalhador doméstico;
2. militar;
3. empregado do setor privado;
4. empregado do setor público;
5. empregador;
6. trabalhador por conta própria;
7. trabalhador familiar auxiliar;
8. categoria técnica utilizada pelo modelo para registros não classificados.

Nas análises comparativas, a categoria técnica é excluída do universo substantivamente interpretado.

A classificação como **trabalhador por conta própria** é tratada no MVP como uma categoria estatística de posição na ocupação. Ela não é tomada, isoladamente, como equivalente empírico de autonomia substantiva no exercício da atividade.

### 3.5.7 `gold_dim_atividade_principal`

**Descrição:** dimensão destinada à preservação dos códigos de atividade principal associados às observações.

**Quantidade de registros:** **223**.

**Origem:** variável `V4013`, extraída e harmonizada na camada Silver.

| Campo/Função | Descrição |
|---|---|
| `id_atividade_principal` | Chave/código utilizado no relacionamento com a tabela fato |
| Código de atividade principal | Código preservado a partir de `V4013` |

No desenvolvimento do MVP, optou-se por não atribuir descrições textuais não validadas ao conjunto completo dos códigos de `V4013`. A dimensão preserva os códigos necessários ao modelo, enquanto a interpretação substantiva permanece limitada às informações cuja correspondência pôde ser validada no escopo do projeto.

Essa decisão evita incorporar ao produto classificações que não tenham sido suficientemente verificadas.

## 3.6 Marts analíticos

Além do esquema estrela, a camada Gold contém tabelas agregadas destinadas ao consumo das análises.

| Tabela | Registros | Função |
|---|---:|---|
| `gold_indicadores_gerais` | 2 | Indicadores gerais por período |
| `gold_perfil_sexo` | 4 | Perfil dos entregadores segundo sexo |
| `gold_perfil_cor_raca` | 10 | Perfil segundo cor ou raça |
| `gold_perfil_faixa_etaria` | 14 | Perfil segundo faixa etária |
| `gold_perfil_posicao_ocupacao` | 8 | Perfil segundo posição na ocupação |
| `gold_perfil_atividade_principal` | 32 | Perfil segundo atividade principal |

Os marts são derivados das estruturas da camada Gold e concentram agregações utilizadas nas análises finais. Sua construção permite reutilizar os indicadores sem reconstruir repetidamente as mesmas operações de filtragem, agrupamento e ponderação.

## 3.7 Linhagem dos dados

A linhagem geral do modelo é:

**PNAD Contínua → Bronze → `silver_entregadores_pnad` → dimensões + `gold_fact_trabalhador` → marts analíticos → indicadores, comparações e visualizações**

Na **Bronze**, os arquivos de origem são preservados e submetidos às verificações iniciais.

Na **Silver**, as variáveis são extraídas de suas posições nos arquivos de largura fixa, tipadas e harmonizadas. Nessa camada também são reconstruídos `SD14001` e o indicador `entregador_plataformizado`.

A tabela `silver_entregadores_pnad` preserva as **957.869 observações** processadas e constitui a fonte da modelagem dimensional.

Na **Gold**, são construídas as seis dimensões e a tabela `gold_fact_trabalhador`. A partir dessas estruturas são produzidos os marts e, posteriormente, as análises.

Essa organização permite acompanhar o percurso das informações entre a origem e o consumo analítico, mantendo separadas as etapas de ingestão, transformação, modelagem e análise.

## 3.8 Validação do modelo

Após a construção do esquema estrela, foram realizadas verificações de integridade entre a tabela fato e as dimensões.

A tabela `gold_fact_trabalhador` contém **957.869 registros**, preservando o total da base harmonizada da camada Silver.

As dimensões apresentam as seguintes quantidades:

| Dimensão | Registros |
|---|---:|
| `gold_dim_tempo` | 2 |
| `gold_dim_sexo` | 2 |
| `gold_dim_cor_raca` | 6 |
| `gold_dim_faixa_etaria` | 8 |
| `gold_dim_posicao_ocupacao` | 8 |
| `gold_dim_atividade_principal` | 223 |

Também foram verificadas as chaves dimensionais da tabela fato. Os controles finais apresentaram **zero registros sem correspondência nas seis dimensões**, indicando consistência referencial entre a tabela fato e as estruturas dimensionais utilizadas pelo modelo.

Os marts também foram auditados após sua construção, permitindo verificar a coerência das agregações com os controles estabelecidos nas etapas anteriores.

## 3.9 Evidências da modelagem e do catálogo

A documentação visual desta etapa será composta por evidências da modelagem efetivamente implementada no Databricks.

**[INSERIR SCREENSHOT DO DIAGRAMA ENTIDADE-RELACIONAMENTO]**

**[INSERIR SCREENSHOT DAS TABELAS GOLD PERSISTIDAS NO DATABRICKS]**

**[INSERIR SCREENSHOT DO CATÁLOGO/ESTRUTURA DAS TABELAS E CAMPOS]**

**[INSERIR SCREENSHOT DA VALIDAÇÃO DAS CHAVES DIMENSIONAIS — RESULTADO ZERO PARA CHAVES SEM CORRESPONDÊNCIA]**

A implementação da modelagem dimensional, do catálogo, dos marts e das verificações correspondentes está disponível no notebook `03_construcao_pnad_gold`.


# 4. Pipeline de Dados (Etapa 4.4)

## 4.1 Organização do pipeline

O pipeline foi desenvolvido no **Databricks** e organizado em três notebooks independentes, correspondentes às camadas Bronze, Silver e Gold da arquitetura Medallion.

| Ordem | Notebook | Camada | Responsabilidade |
|---|---|---|---|
| 1 | `01_ingestao_pnad_bronze` | Bronze | Ingestão e validação inicial dos microdados |
| 2 | `02_transformacao_pnad_silver` | Silver | Extração, tipagem, harmonização e construção das variáveis analíticas |
| 3 | `03_construcao_pnad_gold` | Gold | Modelagem dimensional, persistência das estruturas analíticas, validações e análises |

A separação em três notebooks procura atribuir responsabilidades específicas a cada etapa e tornar mais visível o percurso realizado pelos dados.

O fluxo geral pode ser sintetizado da seguinte forma:

**Microdados da PNAD Contínua → Bronze → Silver → Gold → Marts analíticos → Indicadores e visualizações**

A execução deve respeitar essa sequência, uma vez que cada camada utiliza estruturas produzidas ou validadas na etapa anterior.

## 4.2 Camada Bronze — `01_ingestao_pnad_bronze`

A primeira etapa do pipeline corresponde à ingestão e à validação inicial dos arquivos de microdados.

São utilizados:

- `PNADC_2022_trimestre4.txt`, com **478.091 registros**;
- `PNADC_2024_trimestre3.txt`, com **479.778 registros**.

Os arquivos são mantidos em sua estrutura de largura fixa. A Bronze não realiza a identificação dos entregadores nem aplica as transformações analíticas utilizadas posteriormente.

Entre as operações realizadas no notebook estão:

1. definição e verificação dos diretórios;
2. inspeção dos arquivos de origem;
3. extração dos arquivos utilizados;
4. verificação dos microdados extraídos;
5. leitura inicial dos arquivos;
6. conferência da quantidade de registros.

Essa etapa estabelece os controles iniciais que serão utilizados para verificar a preservação das observações nas camadas posteriores.

## 4.3 Camada Silver — `02_transformacao_pnad_silver`

A camada Silver concentra as principais operações de transformação e harmonização dos microdados.

Como os arquivos possuem estrutura de largura fixa e algumas variáveis ocupam posições diferentes em 2022 e 2024, foi construído um mapeamento específico para cada período. Após a extração, as variáveis são harmonizadas em uma estrutura comum.

Entre as variáveis incorporadas ao processamento estão:

- `V1028` — peso amostral;
- `V4012` — posição na ocupação;
- `V40121` — informação complementar sobre trabalhador familiar auxiliar;
- `V4013` — atividade principal;
- `sexo`;
- `idade`;
- `cor_raca`;
- `S140091`;
- `S140092`;
- `S140093`;
- `S140093A`, quando existente no período;
- `S140094`.

A partir dessas informações são construídas e harmonizadas variáveis utilizadas nas etapas seguintes, entre elas:

- `ano_pnad`;
- `peso_amostral`;
- `idade_anos`;
- `sexo_desc`;
- `cor_raca_desc`;
- `entregador_app`;
- `app_taxi`;
- `app_transporte_passageiros`;
- `app_servicos`;
- `SD14001`;
- `app_entrega`;
- `entregador_plataformizado`.

### Reconstrução de `SD14001`

Uma etapa relevante da Silver é a reconstrução da variável derivada `SD14001`, utilizada para identificar trabalhadores plataformizados no trabalho principal.

A regra combina informações sobre utilização de diferentes tipos de plataformas com informações de atividade principal e, em determinadas situações, posição na ocupação.

Para os serviços de entrega, a identificação não é realizada apenas a partir de `S140093`. A regra considera também condições relacionadas a `V4013`, `V4012` e `V40121`, conforme aplicável.

A reconstrução foi submetida a controles antes da definição do indicador analítico utilizado no restante do projeto.

### Construção de `entregador_plataformizado`

Após a reconstrução de `SD14001`, foi construído o indicador:

`entregador_plataformizado`

O indicador identifica os registros que simultaneamente apresentam informação de atividade de entrega por aplicativo e atendem aos critérios da variável derivada de trabalho plataformizado utilizados no projeto.

Os controles resultaram em:

| Período | Registros amostrais | Estimativa ponderada |
|---|---:|---:|
| 4º trimestre de 2022 | 691 | 445.866,9 |
| 3º trimestre de 2024 | 778 | 487.284,9 |

Esses valores foram preservados como controles para as etapas posteriores do pipeline.

### Persistência da Silver

Após as transformações, a estrutura harmonizada é persistida em formato **Delta** como:

`silver_entregadores_pnad`

A tabela possui **957.869 registros** e preserva as observações dos dois períodos, juntamente com as variáveis originais selecionadas, os campos tipados e os indicadores construídos.

A opção por manter o universo harmonizado, em vez de persistir somente os entregadores, permite que a camada Gold utilize a mesma base tanto para a caracterização da população de interesse quanto para comparações com os demais trabalhadores.

## 4.4 Camada Gold — `03_construcao_pnad_gold`

A camada Gold transforma a estrutura harmonizada da Silver em um modelo destinado ao consumo analítico.

A fonte dessa etapa é:

`silver_entregadores_pnad`

A partir dela são construídas seis dimensões:

- `gold_dim_tempo`;
- `gold_dim_sexo`;
- `gold_dim_cor_raca`;
- `gold_dim_faixa_etaria`;
- `gold_dim_posicao_ocupacao`;
- `gold_dim_atividade_principal`.

A tabela fato central é:

`gold_fact_trabalhador`

A tabela fato preserva as **957.869 observações** da Silver e incorpora as chaves dimensionais necessárias às análises, além do `peso_amostral` e dos indicadores utilizados para identificar a população analítica.

A Gold também incorpora controles de integridade destinados a verificar a correspondência entre as chaves da tabela fato e as respectivas dimensões. Os controles finais apresentaram **zero registros sem correspondência nas seis dimensões**.

## 4.5 Construção dos marts analíticos

A partir das estruturas Gold foram construídos seis marts:

| Mart | Registros | Finalidade |
|---|---:|---|
| `gold_indicadores_gerais` | 2 | Indicadores gerais por período |
| `gold_perfil_sexo` | 4 | Perfil segundo sexo |
| `gold_perfil_cor_raca` | 10 | Perfil segundo cor ou raça |
| `gold_perfil_faixa_etaria` | 14 | Perfil segundo faixa etária |
| `gold_perfil_posicao_ocupacao` | 8 | Perfil segundo posição na ocupação |
| `gold_perfil_atividade_principal` | 32 | Perfil segundo atividade principal |

Os marts concentram agregações utilizadas nas análises e permitem separar a modelagem dimensional das estruturas preparadas para consumo.

Os totais produzidos pelos marts foram comparados com os controles construídos nas etapas anteriores, permitindo verificar a consistência entre a população identificada na Silver, a tabela fato e os resultados agregados da Gold.

## 4.6 Persistência e reutilização dos dados

A persistência constitui uma etapa do pipeline e não apenas uma operação intermediária de execução dos notebooks.

Na Silver, a base harmonizada é gravada como:

`silver_entregadores_pnad`

Na Gold, são persistidas a tabela fato, as seis dimensões e os marts analíticos.

Essa organização permite que as estruturas tratadas sejam reutilizadas sem a necessidade de executar novamente todas as transformações sempre que uma análise for realizada.

A separação também estabelece diferentes níveis de processamento:

| Camada | Estrutura resultante |
|---|---|
| Bronze | Microdados preservados em sua estrutura de origem |
| Silver | Base harmonizada e persistida em Delta |
| Gold | Modelo dimensional e marts analíticos persistidos |

## 4.7 Reprodutibilidade e ordem de execução

Para reproduzir o pipeline, os notebooks devem ser executados na seguinte ordem:

**1. `01_ingestao_pnad_bronze`**

Realiza a ingestão, extração e validação inicial dos arquivos.

**2. `02_transformacao_pnad_silver`**

Realiza a extração posicional das variáveis, harmonização entre os períodos, tipagem, reconstrução de `SD14001`, construção de `entregador_plataformizado` e persistência de `silver_entregadores_pnad`.

**3. `03_construcao_pnad_gold`**

Constrói as dimensões, a tabela `gold_fact_trabalhador`, os marts analíticos, os controles de qualidade e as análises finais.

A divisão em notebooks independentes permite executar e verificar cada etapa separadamente, ao mesmo tempo em que preserva a dependência lógica entre as camadas.

## 4.8 Estrutura do repositório

O código do pipeline está versionado no repositório `dados_analytics_pucrj`, organizado da seguinte forma:

| Arquivo | Função |
|---|---|
| `README.md` | Documentação do MVP |
| `01_ingestao_pnad_bronze` | Pipeline da camada Bronze |
| `02_transformacao_pnad_silver` | Pipeline da camada Silver |
| `03_construcao_pnad_gold` | Modelagem e análises da camada Gold |

Os três notebooks correspondentes ao pipeline estão, portanto, versionados no mesmo repositório utilizado para a entrega do projeto.

## 4.9 Evidências do pipeline e da persistência

Para documentar a execução do pipeline e a persistência das estruturas na plataforma de nuvem, serão incorporadas as seguintes evidências:

**[INSERIR SCREENSHOT DO REPOSITÓRIO MOSTRANDO `01_ingestao_pnad_bronze`, `02_transformacao_pnad_silver`, `03_construcao_pnad_gold` E `README.md`]**

**[INSERIR SCREENSHOT DA TABELA `silver_entregadores_pnad` PERSISTIDA]**

**[INSERIR SCREENSHOT DAS DIMENSÕES E DA TABELA `gold_fact_trabalhador` PERSISTIDAS]**

**[INSERIR SCREENSHOT DOS MARTS GOLD PERSISTIDOS]**

Essas evidências permitem relacionar a documentação apresentada no README às estruturas efetivamente construídas e persistidas no Databricks.


# 5. Qualidade de Dados (Etapa 4.5)

## 5.1 Estratégia de qualidade

A qualidade dos dados foi tratada ao longo das diferentes etapas do pipeline, e não apenas após a construção das tabelas finais. Os controles procuram verificar se os arquivos foram corretamente processados, se as transformações preservaram o universo esperado e se as estruturas destinadas à análise permaneceram consistentes.

As verificações foram organizadas em torno de cinco dimensões: **completude, consistência, unicidade, acurácia e identificação de valores potencialmente anômalos**. A aplicação dessas dimensões foi adaptada às características dos microdados da PNAD Contínua e às transformações efetivamente realizadas no MVP.

Além dos testes técnicos, algumas decisões de tratamento foram orientadas pela necessidade de não transformar automaticamente valores ausentes ou categorias técnicas em informações substantivas.

## 5.2 Completude

A primeira verificação de completude ocorreu ainda na camada Bronze, por meio da conferência da quantidade de registros dos arquivos utilizados.

| Período | Registros |
|---|---:|
| 4º trimestre de 2022 | 478.091 |
| 3º trimestre de 2024 | 479.778 |
| **Total** | **957.869** |

Após a extração e harmonização das variáveis, a tabela `silver_entregadores_pnad` permaneceu com **957.869 registros**.

A tabela `gold_fact_trabalhador`, construída posteriormente a partir da Silver, também apresentou **957.869 registros**.

Dessa forma, o controle entre as camadas permitiu verificar que o processo de transformação e modelagem preservou a quantidade total de observações processadas:

**Bronze: 957.869 → Silver: 957.869 → Gold: 957.869**

Esse controle é particularmente relevante porque a base Silver e a tabela fato não foram construídas apenas com os entregadores. O universo harmonizado foi deliberadamente preservado para permitir análises comparativas posteriores.

## 5.3 Consistência entre os períodos

Os arquivos de 2022 e 2024 apresentam diferenças nas posições ocupadas por algumas variáveis dentro dos registros de largura fixa.

Por essa razão, foi utilizado um mapeamento específico para cada período antes da harmonização.

Entre as variáveis tratadas estão:

- `V1028`;
- `V4012`;
- `V40121`;
- `V4013`;
- `sexo`;
- `idade`;
- `cor_raca`;
- `S140091`;
- `S140092`;
- `S140093`;
- `S140093A`;
- `S140094`.

A variável `S140093A`, presente na estrutura considerada para 2024, apresentou registros sem preenchimento e não foi utilizada na reconstrução de `SD14001`.

Após a extração posicional, os dois períodos foram convertidos para uma estrutura comum antes de sua união na tabela `silver_entregadores_pnad`.

Esse procedimento procura evitar que diferenças de posição entre os arquivos sejam interpretadas como diferenças substantivas entre as variáveis.

## 5.4 Tipagem e padronização das variáveis

Na Silver foram construídas variáveis tipadas e descritivas destinadas a reduzir a dependência direta da codificação original durante as análises.

Entre elas estão:

- `peso_amostral`;
- `idade_anos`;
- `sexo_desc`;
- `cor_raca_desc`;
- `ano_pnad`.

O campo `peso_amostral` é derivado de `V1028` e convertido para formato numérico apropriado às operações de ponderação.

A idade é convertida para `idade_anos` e posteriormente utilizada na construção das faixas etárias da camada Gold.

As categorias de sexo e cor ou raça são preservadas em seus campos de origem e também representadas por variáveis descritivas, permitindo manter simultaneamente a rastreabilidade e uma estrutura mais adequada ao consumo analítico.

## 5.5 Validação do peso amostral

Como as estimativas populacionais produzidas no MVP dependem de `peso_amostral`, sua conversão e comportamento foram verificados após a extração.

Os controles apresentaram valores positivos nos dois períodos, com as seguintes estatísticas descritivas:

| Período | Mínimo | Máximo | Média aproximada |
|---|---:|---:|---:|
| 2022 | 8,08 | 18.229,62 | 440,32 |
| 2024 | 5,23 | 9.093,43 | 442,08 |

A amplitude observada não foi tratada automaticamente como erro ou outlier a ser removido. Por se tratar de uma variável de ponderação proveniente do desenho amostral, valores elevados não foram excluídos apenas por sua distância em relação à média.

Essa decisão evita aplicar critérios genéricos de remoção de outliers a uma variável cuja distribuição possui função específica na produção das estimativas populacionais.

## 5.6 Validação da reconstrução de `SD14001`

A reconstrução de `SD14001` constituiu um dos principais pontos de controle da camada Silver.

A variável combina informações relacionadas ao uso de plataformas digitais com atividade principal e posição na ocupação, conforme as condições incorporadas ao processamento.

Após a reconstrução, a distribuição obtida foi:

| Período | `SD14001 = 1` | `SD14001 = 2` | Nulo |
|---|---:|---:|---:|
| 2022 | 2.203 | 175.960 | 299.928 |
| 2024 | 2.743 | 181.414 | 295.621 |

Os valores nulos não foram automaticamente recodificados como ausência de trabalho por plataforma. Eles são preservados como resultado das condições de aplicabilidade e da estrutura das informações utilizadas na construção da variável.

Essa distinção evita transformar ausência de classificação em resposta negativa.

## 5.7 Validação da identificação dos entregadores

A variável `S140093` permite identificar registros associados à utilização de aplicativo em atividades de entrega, mas o indicador analítico final do projeto não foi construído exclusivamente a partir dessa resposta.

Foi realizado um cruzamento entre `S140093` e `SD14001` para verificar a relação entre a informação de entrega e a classificação como trabalhador plataformizado.

Os controles apresentaram:

| Período | `S140093 = 1` | Entregadores classificados como plataformizados |
|---|---:|---:|
| 2022 | 892 | 691 |
| 2024 | 985 | 778 |

Em termos ponderados:

| Período | `S140093 = 1` | `entregador_plataformizado` |
|---|---:|---:|
| 2022 | 570.639,1 | 445.866,9 |
| 2024 | 622.340,1 | 487.284,9 |

Essa diferença constitui uma verificação importante do pipeline: responder afirmativamente à questão associada à entrega por aplicativo não produz, isoladamente, a população analítica utilizada no MVP.

O indicador `entregador_plataformizado` foi definido a partir da combinação entre a informação de entrega e a classificação reconstruída em `SD14001`.

O resultado final utilizado como controle é:

- **2022:** 691 observações amostrais, correspondentes a aproximadamente **445,9 mil** trabalhadores na estimativa ponderada;
- **2024:** 778 observações amostrais, correspondentes a aproximadamente **487,3 mil** trabalhadores na estimativa ponderada.

Esses valores são utilizados como referências para verificar as agregações produzidas posteriormente na camada Gold.

## 5.8 Unicidade e granularidade

A análise de unicidade foi orientada pela granularidade definida para o modelo.

A tabela `gold_fact_trabalhador` possui uma linha para cada observação individual da PNAD Contínua em determinado período de referência.

Não se pressupõe, portanto, que atributos como sexo, idade, posição na ocupação ou atividade principal sejam individualmente únicos. Esses campos podem naturalmente se repetir entre diferentes observações.

A verificação de unicidade deve ser compreendida em relação à unidade de análise definida para a tabela fato, evitando a aplicação de critérios de deduplicação que poderiam eliminar observações amostrais válidas.

## 5.9 Integridade referencial da camada Gold

Após a construção do modelo dimensional, foram realizados controles para verificar se as chaves existentes em `gold_fact_trabalhador` possuíam correspondência nas respectivas dimensões.

Foram verificadas:

- `id_tempo`;
- `id_sexo`;
- `id_cor_raca`;
- `id_faixa_etaria`;
- `id_posicao_ocupacao`;
- `id_atividade_principal`.

Os controles finais apresentaram **zero registros sem correspondência nas seis dimensões**.

Esse resultado indica que as observações da tabela fato permaneceram associadas às categorias necessárias ao funcionamento do esquema estrela.

## 5.10 Tratamento de categorias técnicas e valores não classificados

Nem toda ausência de informação foi convertida em uma categoria substantiva.

Na construção das dimensões, foram preservadas categorias técnicas quando necessárias para impedir a perda de registros durante a modelagem.

Na dimensão de faixa etária, por exemplo, a categoria técnica permite acomodar registros que não pertencem às sete faixas utilizadas na interpretação. Essa categoria é posteriormente excluída quando a análise exige exclusivamente faixas etárias substantivas.

Procedimento semelhante é adotado nas análises de posição na ocupação, nas quais registros associados à categoria técnica do modelo não são incorporados à interpretação substantiva das distribuições.

A distinção entre categorias analíticas e técnicas permite preservar a integridade do modelo sem atribuir significado sociológico a registros cuja função é apenas operacional.

## 5.11 Acurácia e limites de validação

No escopo deste MVP, a acurácia foi tratada principalmente por meio da rastreabilidade entre os campos de origem e as variáveis derivadas, da conferência das regras utilizadas nas transformações e da comparação dos resultados entre diferentes etapas do pipeline.

Não se assume, entretanto, que os controles realizados permitam avaliar todas as dimensões possíveis de acurácia dos microdados produzidos pela pesquisa de origem.

O objetivo dos testes é verificar se o pipeline preserva e transforma de maneira consistente as informações utilizadas pelo projeto, sem atribuir às rotinas desenvolvidas a capacidade de validar externamente o processo de produção da PNAD Contínua.

## 5.12 Validação das agregações

Os resultados produzidos pelos marts da camada Gold foram comparados com os controles estabelecidos anteriormente na Silver.

Os marts persistidos são:

| Mart | Registros |
|---|---:|
| `gold_indicadores_gerais` | 2 |
| `gold_perfil_sexo` | 4 |
| `gold_perfil_cor_raca` | 10 |
| `gold_perfil_faixa_etaria` | 14 |
| `gold_perfil_posicao_ocupacao` | 8 |
| `gold_perfil_atividade_principal` | 32 |

A auditoria permite verificar se as agregações destinadas ao consumo analítico permanecem compatíveis com os totais de controle da população de entregadores plataformizados.

Esse encadeamento cria pontos de conferência entre diferentes estágios:

**arquivos de origem → Silver → indicador analítico → Gold → marts → resultados**

## 5.13 Síntese dos controles de qualidade

Os principais controles realizados podem ser sintetizados da seguinte forma:

| Dimensão | Verificação realizada | Resultado/decisão |
|---|---|---|
| Completude | Contagem entre Bronze, Silver e Gold | 957.869 registros preservados |
| Consistência | Harmonização das posições das variáveis entre 2022 e 2024 | Estrutura comum construída na Silver |
| Tipagem | Conversão das variáveis utilizadas nas análises | Campos analíticos tipados e padronizados |
| Peso amostral | Verificação de valores e estatísticas descritivas | Valores preservados; ausência de remoção automática de extremos |
| `SD14001` | Distribuição e conferência da reconstrução | Regra validada antes da criação do indicador final |
| População analítica | Cruzamento de `S140093` com `SD14001` | 691 registros em 2022 e 778 em 2024 |
| Unicidade | Verificação segundo a granularidade da fato | Preservação das observações amostrais |
| Integridade referencial | Correspondência das seis chaves dimensionais | Zero registros sem correspondência |
| Categorias técnicas | Separação entre categorias operacionais e substantivas | Categorias técnicas preservadas e excluídas quando necessário à análise |
| Agregações | Auditoria dos marts contra os controles anteriores | Totais compatíveis com a população analítica |

## 5.14 Evidências dos controles de qualidade

As evidências visuais desta etapa serão selecionadas entre os controles efetivamente executados nos notebooks Silver e Gold.

**[INSERIR SCREENSHOT DA VALIDAÇÃO DE `SD14001` E/OU DO CRUZAMENTO COM `S140093`]**

**[INSERIR SCREENSHOT DA VALIDAÇÃO DE `entregador_plataformizado`: 691 EM 2022 E 778 EM 2024]**

**[INSERIR SCREENSHOT DA PRESERVAÇÃO DOS 957.869 REGISTROS NA SILVER/GOLD]**

**[INSERIR SCREENSHOT DA VERIFICAÇÃO DE INTEGRIDADE DAS CHAVES DIMENSIONAIS — ZERO REGISTROS SEM CORRESPONDÊNCIA]**

A implementação dos tratamentos e controles está distribuída principalmente entre os notebooks `02_transformacao_pnad_silver` e `03_construcao_pnad_gold`.



# 6. Análise de Dados (Etapa 4.5)

A etapa analítica utiliza as estruturas construídas na camada Gold para responder às perguntas formuladas no início do MVP.

As estimativas apresentadas nesta seção utilizam `peso_amostral`. Dessa forma, é necessário distinguir a quantidade de observações existentes na amostra da estimativa correspondente à população.

Também é importante considerar que os períodos analisados não correspondem ao mesmo trimestre: são utilizados o **4º trimestre de 2022** e o **3º trimestre de 2024**. As diferenças observadas são apresentadas de maneira descritiva e não são interpretadas como uma tendência temporal contínua.

## 6.1 Dimensão estimada dos entregadores plataformizados

A primeira pergunta do MVP é:

> **Qual é a dimensão estimada da população de entregadores plataformizados nos períodos analisados?**

A aplicação do indicador `entregador_plataformizado` resultou nos seguintes valores:

| Período | Observações amostrais | Estimativa ponderada |
|---|---:|---:|
| 4º trimestre de 2022 | 691 | 445.866,9 |
| 3º trimestre de 2024 | 778 | 487.284,9 |

Os resultados correspondem a aproximadamente **445,9 mil entregadores plataformizados no 4º trimestre de 2022** e **487,3 mil no 3º trimestre de 2024**.

A diferença entre as duas estimativas é de aproximadamente **41,4 mil trabalhadores**, equivalente a uma variação relativa de cerca de **9,3%**.

Essa diferença permite observar que a estimativa obtida para o período analisado de 2024 é superior àquela encontrada para 2022. Entretanto, como os dados correspondem a trimestres distintos, o resultado deve ser compreendido como uma comparação entre dois recortes específicos, e não como evidência suficiente de uma trajetória contínua de crescimento.

**[INSERIR GRÁFICO 1 — ESTIMATIVA DE ENTREGADORES PLATAFORMIZADOS POR PERÍODO]**

## 6.2 Composição segundo sexo

A segunda pergunta do MVP procura compreender como os entregadores se distribuem segundo características sociodemográficas.

Em relação ao sexo, os resultados ponderados foram:

| Período | Homem | Mulher |
|---|---:|---:|
| 2022 | 76,0% | 24,0% |
| 2024 | 76,7% | 23,3% |

Nos dois períodos, aproximadamente **três em cada quatro entregadores plataformizados são homens**.

A estabilidade aproximada das proporções nos dois recortes sugere uma composição marcadamente masculina da população identificada pelo indicador utilizado no MVP.

Esse resultado descreve a composição da população estimada, mas, isoladamente, não permite estabelecer as razões pelas quais homens e mulheres aparecem em proporções distintas nessa atividade.

**[INSERIR GRÁFICO 2 — DISTRIBUIÇÃO DOS ENTREGADORES SEGUNDO SEXO]**

## 6.3 Composição segundo cor ou raça

A distribuição ponderada segundo cor ou raça apresentou:

| Cor ou raça | 2022 | 2024 |
|---|---:|---:|
| Branca | 41,38% | 45,22% |
| Preta | 13,17% | 14,25% |
| Amarela | 1,08% | 0,49% |
| Parda | 44,23% | 39,78% |
| Indígena | 0,14% | 0,26% |

Nos dois períodos, as categorias **parda e branca concentram as maiores parcelas dos entregadores identificados**.

Em 2022, a categoria parda apresenta a maior participação, com **44,23%**, seguida pela branca, com **41,38%**. No período analisado de 2024, a categoria branca aparece com **45,22%**, enquanto a parda corresponde a **39,78%**.

A participação da categoria preta passa de **13,17% para 14,25%** entre os dois recortes.

Essas diferenças descrevem a composição estimada dos entregadores em cada período. Sua interpretação deve considerar tanto a natureza amostral da PNAD Contínua quanto o fato de que o MVP não realiza testes de significância estatística destinados a determinar se as diferenças entre os dois recortes ultrapassam a variabilidade esperada das estimativas.

**[INSERIR GRÁFICO 3 — DISTRIBUIÇÃO DOS ENTREGADORES SEGUNDO COR OU RAÇA]**

## 6.4 Composição segundo faixa etária

A distribuição por faixa etária apresenta concentração nas idades intermediárias:

| Faixa etária | 2022 | 2024 |
|---|---:|---:|
| 14–17 | 0,85% | 0,64% |
| 18–24 | 16,17% | 17,36% |
| 25–34 | 35,00% | 34,52% |
| 35–44 | 27,03% | 26,62% |
| 45–54 | 14,75% | 14,30% |
| 55–64 | 4,57% | 5,75% |
| 65+ | 1,63% | 0,80% |

A faixa de **25 a 34 anos** apresenta a maior participação nos dois períodos, seguida pela faixa de **35 a 44 anos**.

Consideradas conjuntamente, as pessoas entre **25 e 44 anos representam aproximadamente 62,0% dos entregadores em 2022 e 61,1% em 2024**.

Os resultados indicam, portanto, uma concentração da população analisada nessas faixas etárias, sem que isso signifique ausência de trabalhadores mais jovens ou mais velhos na atividade.

**[INSERIR GRÁFICO 4 — DISTRIBUIÇÃO DOS ENTREGADORES SEGUNDO FAIXA ETÁRIA]**

## 6.5 Posição na ocupação

A terceira pergunta formulada para o MVP é:

> **Como os entregadores plataformizados se distribuem segundo sua posição na ocupação?**

A análise da dimensão `gold_dim_posicao_ocupacao` mostra predominância da categoria **trabalhador por conta própria**.

Nos dois períodos, aproximadamente **três quartos dos entregadores plataformizados são classificados como trabalhadores por conta própria**:

- **2022: aproximadamente 75,0%**;
- **2024: aproximadamente 75,5%**.

As demais posições ocupacionais apresentam participações consideravelmente menores na composição dos entregadores identificados.

Esse resultado é relevante para a caracterização estatística da atividade, mas requer uma distinção conceitual. A classificação como trabalhador por conta própria é uma categoria estatística de posição na ocupação e **não deve ser tomada automaticamente como equivalente à existência de autonomia substantiva no processo de trabalho**.

O dado permite identificar como esses trabalhadores aparecem classificados nos microdados. A discussão sobre autonomia, controle e relações estabelecidas com as plataformas demanda outras dimensões de análise que não podem ser inferidas exclusivamente a partir dessa categoria.

**[INSERIR GRÁFICO 5 — DISTRIBUIÇÃO DOS ENTREGADORES SEGUNDO POSIÇÃO NA OCUPAÇÃO]**

## 6.6 Atividade principal

A quarta pergunta do MVP é:

> **Quais atividades principais aparecem associadas aos entregadores identificados nos microdados?**

A variável `V4013` foi preservada na camada Silver e posteriormente incorporada à dimensão `gold_dim_atividade_principal`.

A análise permitiu identificar a presença de diferentes códigos de atividade principal entre os trabalhadores classificados como entregadores plataformizados.

Entretanto, durante o desenvolvimento do MVP, optou-se por não incorporar ao produto descrições textuais para o conjunto completo dos códigos sem que houvesse uma correspondência integralmente validada no escopo do projeto.

Por essa razão, `gold_dim_atividade_principal` preserva os códigos de `V4013`, e o mart `gold_perfil_atividade_principal` mantém as respectivas agregações, mas os resultados não são apresentados como uma classificação substantiva completa das atividades.

Essa decisão constitui também um controle sobre a interpretação: a existência de um código nos microdados não autoriza atribuir a ele uma descrição que não tenha sido previamente validada.

A dimensão permanece disponível na arquitetura do Observatório e poderá ser enriquecida posteriormente com a documentação correspondente, sem necessidade de reconstrução das demais etapas do pipeline.

## 6.7 Comparação com os demais trabalhadores

A quinta pergunta do MVP amplia a análise para além da descrição interna da população de entregadores:

> **Como a composição dos entregadores plataformizados se diferencia daquela observada entre os demais trabalhadores, especialmente segundo sexo, faixa etária e posição na ocupação?**

A preservação do universo harmonizado na `gold_fact_trabalhador` permite realizar essa comparação utilizando a mesma estrutura de dados e os mesmos critérios de ponderação.

Para essa etapa, os registros foram classificados em dois grupos:

- **Entregadores plataformizados**;
- **Demais trabalhadores**.

A comparação foi realizada segundo sexo, faixa etária e posição na ocupação.

### Sexo

A participação masculina entre os entregadores é superior à observada entre os demais trabalhadores nos dois períodos:

| Período | Grupo | Homens | Mulheres |
|---|---|---:|---:|
| 2022 | Entregadores plataformizados | 76,0% | 24,0% |
| 2022 | Demais trabalhadores | 56,8% | 43,2% |
| 2024 | Entregadores plataformizados | 76,7% | 23,3% |
| 2024 | Demais trabalhadores | 56,5% | 43,5% |

A diferença na participação masculina é de aproximadamente **19,2 pontos percentuais em 2022** e **20,2 pontos percentuais em 2024**.

A comparação permite observar que a predominância masculina não decorre apenas da composição geral dos trabalhadores presentes no universo de comparação. Ela aparece de forma mais acentuada entre os entregadores identificados.

**[INSERIR GRÁFICO 6 — COMPARAÇÃO SEGUNDO SEXO]**

### Faixa etária

Também aparecem diferenças na composição etária.

A faixa de **25 a 34 anos** corresponde a aproximadamente **35,0% dos entregadores em 2022**, frente a **25,0% entre os demais trabalhadores**. Em 2024, as proporções são de aproximadamente **34,5% e 24,4%**, respectivamente.

Quando agrupadas as faixas de **25 a 44 anos**, os resultados são:

| Período | Entregadores plataformizados | Demais trabalhadores |
|---|---:|---:|
| 2022 | 62,0% | 51,1% |
| 2024 | 61,1% | 52,1% |

A diferença é próxima de **10 pontos percentuais**, indicando maior concentração dos entregadores entre 25 e 44 anos nos dois recortes analisados.

**[INSERIR GRÁFICO 7 — COMPARAÇÃO SEGUNDO FAIXA ETÁRIA]**

### Posição na ocupação

A diferença mais acentuada aparece na posição na ocupação.

A categoria **trabalhador por conta própria** corresponde a:

| Período | Entregadores plataformizados | Demais trabalhadores |
|---|---:|---:|
| 2022 | 75,0% | 25,4% |
| 2024 | 75,5% | 24,4% |

Em sentido distinto, os empregados do setor privado representam aproximadamente **50,6% dos demais trabalhadores em 2022 e 51,9% em 2024**, enquanto entre os entregadores correspondem a aproximadamente **5,2% e 4,7%**, respectivamente.

Os resultados mostram uma diferença expressiva na forma como os dois grupos se distribuem segundo a posição na ocupação.

Novamente, essa comparação descreve uma classificação estatística. A elevada presença da categoria por conta própria entre os entregadores não permite concluir, por si só, que esses trabalhadores possuam maior autonomia efetiva sobre as condições de realização de seu trabalho.

**[INSERIR GRÁFICO 8 — COMPARAÇÃO SEGUNDO POSIÇÃO NA OCUPAÇÃO]**

## 6.8 Síntese das respostas às perguntas do MVP

As análises realizadas permitem responder às cinco perguntas formuladas no início do projeto.

| Pergunta | Resultado principal |
|---|---|
| Qual é a dimensão estimada dos entregadores plataformizados? | Aproximadamente 445,9 mil no 4º trimestre de 2022 e 487,3 mil no 3º trimestre de 2024 |
| Como se distribuem segundo sexo, cor ou raça e faixa etária? | Predominância masculina, maior participação das categorias branca e parda e concentração entre 25 e 44 anos |
| Como se distribuem segundo posição na ocupação? | Aproximadamente três quartos são classificados como trabalhadores por conta própria |
| Quais atividades principais aparecem associadas aos entregadores? | Os códigos de `V4013` foram preservados e agregados, mas sua descrição completa não foi incorporada sem validação documental integral |
| Como se diferenciam dos demais trabalhadores? | Os entregadores apresentam maior participação masculina, maior concentração entre 25 e 44 anos e presença consideravelmente mais elevada da categoria por conta própria |

Em conjunto, os resultados permitem construir uma caracterização quantitativa inicial da população identificada como entregadores plataformizados nos dois períodos selecionados.

O principal resultado do MVP, entretanto, não se restringe aos indicadores produzidos. A construção do pipeline estabelece uma infraestrutura capaz de preservar os microdados, harmonizar períodos distintos, documentar as transformações, produzir estimativas ponderadas e reutilizar as estruturas para novas perguntas.

Nesse sentido, as análises apresentadas constituem uma primeira aplicação do **Observatório de Dados dos Entregadores por Aplicativo**, e não um encerramento de suas possibilidades analíticas.


# 7. Autoavaliação

## 7.1 Atingimento dos objetivos

O objetivo proposto para este MVP foi desenvolver um pipeline de Engenharia de Dados capaz de transformar microdados da PNAD Contínua em uma estrutura organizada, rastreável e reutilizável para a análise quantitativa dos entregadores por aplicativo.

Considero que esse objetivo foi alcançado.

A solução desenvolvida permite partir dos arquivos de microdados em largura fixa, preservar sua estrutura de origem, extrair e harmonizar variáveis de períodos distintos, reconstruir indicadores necessários à identificação da população de interesse, persistir uma base tratada e organizar os dados em um modelo dimensional destinado ao consumo analítico.

O pipeline resultante possui três camadas com responsabilidades distintas:

**Bronze → Silver → Gold**

A Bronze preserva e valida os arquivos de origem; a Silver concentra a extração, tipagem, harmonização e construção das variáveis analíticas; e a Gold organiza os dados em um esquema estrela, produz marts e disponibiliza as estruturas utilizadas nas análises.

A tabela `silver_entregadores_pnad` e a `gold_fact_trabalhador` preservam as **957.869 observações** processadas, enquanto o indicador `entregador_plataformizado` permite identificar a população específica de interesse sem eliminar da arquitetura os registros necessários às comparações.

Dessa forma, o produto final não se limita à produção de um conjunto de gráficos. O MVP estabelece uma infraestrutura inicial para o **Observatório de Dados dos Entregadores por Aplicativo**, que poderá ser ampliada sem necessidade de reconstruir integralmente as etapas já desenvolvidas.

## 7.2 Principais dificuldades e aprendizados

Uma das principais dificuldades encontradas esteve relacionada à própria estrutura dos microdados da PNAD Contínua.

Os arquivos utilizados possuem largura fixa, exigindo a identificação das posições ocupadas pelas variáveis antes de sua transformação em uma estrutura tabular adequada às análises. Além disso, algumas posições diferem entre os arquivos de 2022 e 2024, o que tornou necessária a construção de um mapeamento específico por período antes da harmonização.

Outro ponto relevante foi a identificação da população analítica.

A informação de realização de entregas por aplicativo, representada por `S140093`, não foi tratada isoladamente como equivalente à população final de entregadores plataformizados. A reconstrução de `SD14001` exigiu combinar diferentes informações dos microdados e verificar os resultados antes da construção de `entregador_plataformizado`.

Esse processo evidenciou a importância de compreender as regras associadas às variáveis antes de transformá-las em indicadores analíticos.

O desenvolvimento também permitiu compreender de maneira mais concreta a função das diferentes etapas da arquitetura Medallion. Ao longo da construção do MVP, tornou-se importante separar operações que inicialmente poderiam ser realizadas em um único notebook, distinguindo ingestão, transformação e modelagem segundo suas responsabilidades no pipeline.

A construção do esquema estrela representou outro aprendizado relevante. A decisão de preservar todo o universo harmonizado na `gold_fact_trabalhador`, em vez de manter apenas os entregadores, mostrou-se particularmente útil quando as perguntas analíticas passaram a incluir comparações com os demais trabalhadores.

Por fim, o uso de `peso_amostral` reforçou a necessidade de distinguir permanentemente registros existentes na amostra e estimativas referentes à população. Essa distinção passou a orientar tanto a construção das agregações quanto a interpretação dos resultados apresentados pelo projeto.

## 7.3 Limitações

O MVP possui limitações que devem ser consideradas na interpretação de seus resultados.

A primeira está relacionada aos períodos selecionados. O projeto utiliza o **4º trimestre de 2022** e o **3º trimestre de 2024**. Por não se tratar dos mesmos trimestres, as diferenças encontradas entre os dois recortes não são interpretadas como uma série temporal contínua ou, isoladamente, como evidência de tendência.

A segunda limitação decorre do conjunto de variáveis incorporado nesta primeira versão. O pipeline foi desenvolvido para responder a um conjunto delimitado de perguntas sobre dimensão estimada, composição sociodemográfica, posição na ocupação, atividade principal e diferenças em relação aos demais trabalhadores. Outras dimensões existentes nos microdados não foram incorporadas ao MVP.

Também permanece uma limitação relacionada à variável `V4013`. Os códigos de atividade principal foram preservados no modelo e agregados na camada Gold, mas não foi incorporada uma descrição textual completa das atividades sem que sua correspondência estivesse integralmente validada no escopo do projeto.

Outra limitação diz respeito ao alcance das análises estatísticas. Os resultados apresentados são predominantemente descritivos e utilizam o peso amostral para produzir estimativas populacionais. O MVP não incorpora, nesta etapa, procedimentos destinados à estimação da precisão amostral ou testes de significância das diferenças observadas.

Por essa razão, pequenas diferenças entre períodos ou categorias são tratadas com cautela e não são apresentadas como evidências suficientes de mudanças substantivas.

## 7.4 Possibilidades de evolução

A arquitetura desenvolvida permite diferentes possibilidades de expansão do Observatório.

Uma primeira possibilidade consiste na incorporação de **novos períodos da PNAD Contínua**, permitindo construir uma base temporal mais extensa e realizar comparações entre períodos equivalentes.

Também poderão ser incorporadas novas variáveis relacionadas às condições e características do trabalho, desde que acompanhadas da respectiva validação documental e de sua integração às camadas Silver e Gold.

A dimensão `gold_dim_atividade_principal` poderá ser enriquecida com descrições validadas dos códigos de `V4013`, ampliando as possibilidades de análise das atividades econômicas associadas aos trabalhadores identificados.

Outra possibilidade consiste na incorporação de **novas fontes de dados**, mantendo a PNAD Contínua como uma das bases do Observatório, mas permitindo que diferentes conjuntos de informações sejam organizados e analisados de maneira integrada quando houver compatibilidade conceitual e metodológica.

As estruturas Gold e os marts também podem servir como fonte para a construção de um **dashboard**, permitindo disponibilizar indicadores de maneira mais acessível e interativa.

Essas possibilidades não foram tratadas como requisitos para a conclusão do MVP. Elas indicam caminhos de desenvolvimento a partir da infraestrutura já implementada.

## 7.5 Síntese da autoavaliação

O desenvolvimento do MVP permitiu articular conhecimentos de Engenharia de Dados a um problema de pesquisa previamente delimitado, transformando arquivos de microdados em uma estrutura organizada para consumo analítico.

O principal aprendizado não esteve apenas na utilização das ferramentas, mas na compreensão de que decisões de Engenharia de Dados interferem diretamente nas perguntas que podem ser formuladas e nos limites das respostas produzidas.

A definição da granularidade, a preservação do universo de comparação, a reconstrução de indicadores, o tratamento dos valores ausentes, o uso dos pesos amostrais e a organização das dimensões não constituem apenas decisões técnicas. Elas condicionam a forma pela qual os dados podem ser posteriormente mobilizados e interpretados.

Nesse sentido, considero que o MVP atingiu seu objetivo ao produzir uma solução funcional e reproduzível e, simultaneamente, estabelecer uma base extensível para o desenvolvimento do **Observatório de Dados dos Entregadores por Aplicativo**.


# 8. Como Reproduzir o Projeto

## 8.1 Ambiente de execução

O MVP foi desenvolvido no **Databricks**, utilizando **PySpark** para o processamento dos dados e **Delta** para a persistência das estruturas tratadas.

O código está versionado no repositório:

`dados_analytics_pucrj`

Os três notebooks que compõem o pipeline são:

1. `01_ingestao_pnad_bronze`
2. `02_transformacao_pnad_silver`
3. `03_construcao_pnad_gold`

Os notebooks devem ser executados nessa ordem, pois as estruturas produzidas em uma camada são utilizadas nas etapas posteriores.

## 8.2 Dados necessários

O projeto utiliza os microdados da PNAD Contínua referentes aos seguintes períodos:

| Período | Arquivo |
|---|---|
| 4º trimestre de 2022 | `PNADC_2022_trimestre4.txt` |
| 3º trimestre de 2024 | `PNADC_2024_trimestre3.txt` |

No ambiente utilizado para o desenvolvimento, os arquivos extraídos estão organizados em:

`/Volumes/workspace/bronze/pnad_raw/2022/extracted/PNADC_2022_trimestre4.txt`

`/Volumes/workspace/bronze/pnad_raw/2024/extracted/PNADC_2024_trimestre3.txt`

Os arquivos possuem estrutura de largura fixa. Por essa razão, a extração das variáveis depende das posições definidas para cada período no notebook da camada Silver.

## 8.3 Ordem de execução

### Etapa 1 — Bronze

Executar:

`01_ingestao_pnad_bronze`

O notebook realiza a ingestão e a validação inicial dos arquivos.

Ao final da etapa, devem ser confirmados os seguintes controles:

| Período | Registros esperados |
|---|---:|
| 4º trimestre de 2022 | 478.091 |
| 3º trimestre de 2024 | 479.778 |
| **Total** | **957.869** |

A Bronze preserva os arquivos em sua estrutura de origem e não realiza a construção dos indicadores analíticos.

### Etapa 2 — Silver

Executar:

`02_transformacao_pnad_silver`

O notebook realiza:

- extração posicional das variáveis;
- harmonização das estruturas de 2022 e 2024;
- tipagem das variáveis;
- construção de campos descritivos;
- reconstrução de `SD14001`;
- validação da informação de entrega;
- construção de `entregador_plataformizado`;
- persistência da camada Silver.

A estrutura resultante é:

`silver_entregadores_pnad`

Ao final da execução, a tabela deve conter:

**957.869 registros**

Os principais controles da população analítica são:

| Período | Observações de `entregador_plataformizado` | Estimativa ponderada |
|---|---:|---:|
| 2022 | 691 | 445.866,9 |
| 2024 | 778 | 487.284,9 |

Esses valores funcionam como referências para a validação da etapa seguinte.

### Etapa 3 — Gold

Executar:

`03_construcao_pnad_gold`

O notebook utiliza `silver_entregadores_pnad` para construir o modelo dimensional e as estruturas destinadas ao consumo analítico.

Ao final da execução, devem estar disponíveis as seguintes dimensões:

- `gold_dim_tempo`;
- `gold_dim_sexo`;
- `gold_dim_cor_raca`;
- `gold_dim_faixa_etaria`;
- `gold_dim_posicao_ocupacao`;
- `gold_dim_atividade_principal`.

A tabela fato resultante é:

`gold_fact_trabalhador`

O número esperado de registros é:

**957.869**

As dimensões devem apresentar:

| Dimensão | Registros esperados |
|---|---:|
| `gold_dim_tempo` | 2 |
| `gold_dim_sexo` | 2 |
| `gold_dim_cor_raca` | 6 |
| `gold_dim_faixa_etaria` | 8 |
| `gold_dim_posicao_ocupacao` | 8 |
| `gold_dim_atividade_principal` | 223 |

A validação de integridade deve apresentar **zero registros sem correspondência nas seis chaves dimensionais**.

## 8.4 Marts esperados

A execução da camada Gold também produz os seguintes marts:

| Mart | Registros esperados |
|---|---:|
| `gold_indicadores_gerais` | 2 |
| `gold_perfil_sexo` | 4 |
| `gold_perfil_cor_raca` | 10 |
| `gold_perfil_faixa_etaria` | 14 |
| `gold_perfil_posicao_ocupacao` | 8 |
| `gold_perfil_atividade_principal` | 32 |

Essas estruturas concentram as agregações utilizadas nas análises apresentadas no MVP.

## 8.5 Controles para validação da reprodução

Após a execução completa, alguns resultados podem ser utilizados como controles para verificar se o pipeline foi reproduzido de maneira consistente.

| Controle | Resultado esperado |
|---|---:|
| Registros processados em 2022 | 478.091 |
| Registros processados em 2024 | 479.778 |
| Total harmonizado na Silver | 957.869 |
| Total da `gold_fact_trabalhador` | 957.869 |
| Entregadores plataformizados em 2022 — amostra | 691 |
| Entregadores plataformizados em 2024 — amostra | 778 |
| Estimativa ponderada de 2022 | aproximadamente 445,9 mil |
| Estimativa ponderada de 2024 | aproximadamente 487,3 mil |
| Chaves dimensionais sem correspondência | 0 |

A correspondência desses controles não substitui as demais verificações de qualidade documentadas no projeto, mas oferece pontos de referência para identificar eventuais diferenças na reprodução do pipeline.

## 8.6 Resultado esperado

Após a execução dos três notebooks, o ambiente deverá conter uma base Silver harmonizada, um modelo dimensional Gold composto por uma tabela fato e seis dimensões, seis marts analíticos e as estruturas necessárias à reprodução das análises apresentadas neste README.

O fluxo completo pode ser resumido como:

**PNAD Contínua → `01_ingestao_pnad_bronze` → `02_transformacao_pnad_silver` → `silver_entregadores_pnad` → `03_construcao_pnad_gold` → modelo dimensional → marts → análises**

A reprodução deve respeitar essa sequência para preservar as dependências estabelecidas entre as diferentes camadas do pipeline.


# 9. Evidências da Solução

Esta seção reúne as principais evidências visuais da implementação do MVP no Databricks e de seu versionamento no Git. As imagens foram selecionadas para documentar o percurso dos dados entre as camadas, a persistência das estruturas, os controles de qualidade e os resultados analíticos produzidos.

## 9.1 Estrutura do projeto e versionamento

O pipeline está organizado em três notebooks correspondentes às camadas Bronze, Silver e Gold, além deste `README.md`.

**Evidência 1 — Estrutura do repositório**

**[INSERIR SCREENSHOT DO REPOSITÓRIO `dados_analytics_pucrj` MOSTRANDO:]**

- `README.md`
- `01_ingestao_pnad_bronze`
- `02_transformacao_pnad_silver`
- `03_construcao_pnad_gold`

A imagem documenta a presença dos três notebooks utilizados no pipeline e do arquivo de documentação no repositório do projeto.

## 9.2 Camada Bronze

A camada Bronze preserva os arquivos de origem e realiza as verificações iniciais da carga.

**Evidência 2 — Ingestão e validação dos microdados**

**[INSERIR SCREENSHOT DO NOTEBOOK `01_ingestao_pnad_bronze` MOSTRANDO OS ARQUIVOS E/OU AS CONTAGENS:]**

- 2022: 478.091 registros;
- 2024: 479.778 registros.

A evidência permite verificar a entrada dos dois arquivos utilizados no MVP e os controles iniciais estabelecidos antes das transformações.

## 9.3 Camada Silver

A Silver concentra a extração, tipagem, harmonização e construção das variáveis analíticas.

**Evidência 3 — Persistência da tabela Silver**

**[INSERIR SCREENSHOT DA `silver_entregadores_pnad` PERSISTIDA NO DATABRICKS]**

A tabela deve apresentar o universo harmonizado de **957.869 registros**.

**Evidência 4 — Validação da população de entregadores plataformizados**

**[INSERIR SCREENSHOT DO NOTEBOOK `02_transformacao_pnad_silver` MOSTRANDO A VALIDAÇÃO DE `entregador_plataformizado`]**

Resultados de controle:

| Período | Observações amostrais | Estimativa ponderada |
|---|---:|---:|
| 2022 | 691 | 445.866,9 |
| 2024 | 778 | 487.284,9 |

Essa evidência documenta a passagem entre as variáveis de origem, a reconstrução de `SD14001` e a população analítica utilizada nas etapas seguintes.

## 9.4 Modelagem da camada Gold

A camada Gold organiza os dados em um esquema estrela composto pela tabela `gold_fact_trabalhador` e seis dimensões.

**Evidência 5 — Diagrama Entidade-Relacionamento**

**[INSERIR `der_gold_entregadores.png`]**

O diagrama apresenta as relações entre:

- `gold_fact_trabalhador`;
- `gold_dim_tempo`;
- `gold_dim_sexo`;
- `gold_dim_cor_raca`;
- `gold_dim_faixa_etaria`;
- `gold_dim_posicao_ocupacao`;
- `gold_dim_atividade_principal`.

**Evidência 6 — Estruturas Gold persistidas**

**[INSERIR SCREENSHOT DO DATABRICKS MOSTRANDO A TABELA FATO E AS DIMENSÕES PERSISTIDAS]**

A tabela `gold_fact_trabalhador` deve apresentar **957.869 registros**.

As dimensões apresentam:

| Dimensão | Registros |
|---|---:|
| `gold_dim_tempo` | 2 |
| `gold_dim_sexo` | 2 |
| `gold_dim_cor_raca` | 6 |
| `gold_dim_faixa_etaria` | 8 |
| `gold_dim_posicao_ocupacao` | 8 |
| `gold_dim_atividade_principal` | 223 |

## 9.5 Catálogo de Dados

O catálogo documenta as estruturas da camada Gold, incluindo a função das tabelas, seus campos, domínios e linhagem.

**Evidência 7 — Catálogo das tabelas e campos**

**[INSERIR SCREENSHOT DO CATÁLOGO/ESTRUTURA DOCUMENTADA NO NOTEBOOK `03_construcao_pnad_gold`]**

A evidência complementa o catálogo transcrito na Seção 3 e permite relacionar a documentação apresentada neste README à implementação realizada no ambiente Databricks.

## 9.6 Controles de qualidade

Foram implementados controles em diferentes etapas do pipeline para verificar a preservação das observações, a construção da população analítica e a integridade do modelo dimensional.

**Evidência 8 — Integridade referencial da Gold**

**[INSERIR SCREENSHOT DA VERIFICAÇÃO DAS CHAVES DIMENSIONAIS]**

Resultado esperado:

**0 registros sem correspondência em cada uma das seis dimensões.**

A evidência documenta a consistência dos relacionamentos entre `gold_fact_trabalhador` e as dimensões.

## 9.7 Marts analíticos

A camada Gold também contém seis marts destinados ao consumo dos indicadores.

**Evidência 9 — Marts persistidos**

**[INSERIR SCREENSHOT MOSTRANDO OS MARTS GOLD]**

| Mart | Registros |
|---|---:|
| `gold_indicadores_gerais` | 2 |
| `gold_perfil_sexo` | 4 |
| `gold_perfil_cor_raca` | 10 |
| `gold_perfil_faixa_etaria` | 14 |
| `gold_perfil_posicao_ocupacao` | 8 |
| `gold_perfil_atividade_principal` | 32 |

A persistência dessas estruturas permite reutilizar as agregações produzidas pelo pipeline para diferentes formas de consumo analítico.

## 9.8 Evidências das análises

As análises finais foram produzidas a partir das estruturas da camada Gold e respondem às perguntas formuladas no início do MVP.

### Evidência 10 — Dimensão estimada dos entregadores

**[INSERIR GRÁFICO 1 — ESTIMATIVA DE ENTREGADORES PLATAFORMIZADOS POR PERÍODO]**

O gráfico apresenta as estimativas de aproximadamente **445,9 mil entregadores no 4º trimestre de 2022** e **487,3 mil no 3º trimestre de 2024**.

### Evidência 11 — Perfil sociodemográfico e ocupacional

**[INSERIR GRÁFICOS 2 A 5:]**

- distribuição segundo sexo;
- distribuição segundo cor ou raça;
- distribuição segundo faixa etária;
- distribuição segundo posição na ocupação.

Essas visualizações documentam a caracterização interna da população de entregadores plataformizados.

### Evidência 12 — Comparação com os demais trabalhadores

**[INSERIR GRÁFICOS 6 A 8:]**

- comparação segundo sexo;
- comparação segundo faixa etária;
- comparação segundo posição na ocupação.

As visualizações permitem observar diferenças entre os entregadores plataformizados e os demais trabalhadores pertencentes ao universo de comparação.

## 9.9 Relação entre evidências e etapas do MVP

As evidências selecionadas permitem acompanhar o desenvolvimento da solução de ponta a ponta:

| Etapa | Evidência principal |
|---|---|
| Versionamento | Estrutura do repositório |
| Carga | Arquivos e contagens da Bronze |
| Transformação | `silver_entregadores_pnad` |
| População analítica | Validação de `entregador_plataformizado` |
| Modelagem | DER e estruturas Gold |
| Catálogo | Documentação das tabelas e campos |
| Qualidade | Integridade das chaves dimensionais |
| Consumo | Marts Gold |
| Análise | Gráficos 1 a 8 |

O conjunto procura documentar não apenas os resultados finais, mas também o percurso realizado pelos dados desde os microdados de origem até as estruturas utilizadas nas análises.