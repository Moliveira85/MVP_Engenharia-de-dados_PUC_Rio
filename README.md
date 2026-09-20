# MVP — Pipeline de Dados para Prescrição Individualizada em Treinamentos Hyrox

## 1. Contexto de Negócios e Perguntas

### 1.1 Contexto

Este projeto apresenta um MVP de engenharia de dados desenvolvido na plataforma Databricks para integrar informações de atletas, dados clínicos, hábitos de suplementação, hidratação, desempenho e sessões de treinamento.

O projeto foi desenvolvido para atletas de treinamento híbrido e Hyrox, considerando a necessidade de relacionar as características individuais do atleta às características de cada sessão de treinamento.

O pipeline recebe dados provenientes de:

- formulários de anamnese;
- arquivos de treinamento exportados do TrainingPeaks;
- tabelas de regras de suplementação;
- protocolos crônicos definidos para o projeto.

Após a ingestão, os dados são armazenados, transformados, padronizados e relacionados por meio de tabelas dimensionais e factuais. Como resultado, é produzida uma tabela Gold com orientações parametrizadas para as sessões de treinamento e documentos PDF destinados à revisão profissional.

O sistema não substitui a avaliação de profissionais habilitados. Seu objetivo é demonstrar a construção de um pipeline de dados reprodutível, com integração de fontes, aplicação de regras de negócio, validação e geração de uma saída operacional.

### 1.2 Problema de negócio

Informações de anamnese, características individuais e sessões de treinamento normalmente ficam distribuídas em diferentes fontes e formatos.

Essa fragmentação dificulta:

- a associação entre o atleta e seus treinamentos;
- a análise conjunta de informações clínicas e esportivas;
- a aplicação consistente de regras de hidratação e suplementação;
- a rastreabilidade das orientações produzidas;
- a revisão dos documentos antes do envio ao atleta.

O problema abordado pelo MVP é construir uma estrutura de dados capaz de integrar essas fontes e produzir uma saída organizada, individualizada e rastreável para apoiar a revisão profissional.

### 1.3 Objetivo geral

Construir um pipeline de dados em nuvem, utilizando o Databricks e PySpark, capaz de integrar dados individuais, clínicos e de treinamento, aplicar regras parametrizadas de hidratação e suplementação e produzir uma camada Gold com orientações organizadas por sessão de treinamento.

Como saída operacional adicional, o pipeline gera documentos PDF para revisão profissional.

### 1.4 Perguntas de negócio

O projeto busca responder às seguintes perguntas:

1. O pipeline consegue associar corretamente cada sessão de treinamento ao atleta correspondente?

2. Quantos atletas e sessões de treinamento foram processados?

3. Como as sessões de treinamento se distribuem por modalidade, duração, distância, intensidade e zonas?

4. Quais alertas clínicos foram identificados durante o processo de triagem?

5. Como os dados individuais do atleta e as características do treinamento são utilizados na aplicação das regras de hidratação e suplementação?

6. O pipeline consegue consolidar as informações em uma tabela Gold pronta para consumo?

7. O sistema consegue produzir uma orientação organizada por sessão de treinamento?

8. O sistema consegue gerar um documento individual para revisão profissional?

9. Quais problemas de qualidade foram encontrados nos dados?

10. Como os problemas de qualidade foram tratados no pipeline?

11. Quais limitações permanecem no MVP?

### 1.5 Escopo do MVP

O escopo deste MVP inclui:

- ingestão de dados de anamnese;
- ingestão de arquivos de treinamento;
- identificação e associação dos atletas;
- organização dos dados em tabelas analíticas;
- criação de dimensões clínicas, esportivas e nutricionais;
- criação de fatos de sessões de treinamento e alertas clínicos;
- aplicação de regras de suplementação e hidratação;
- consolidação da tabela Gold;
- geração de documentos PDF;
- registro da geração dos documentos;
- validação básica da qualidade dos dados.

A execução atual do pipeline é realizada manualmente por meio de notebooks organizados no Databricks. A orquestração por Databricks Job permanece como uma evolução futura.

## 2. Carga dos Dados

### 2.1 Plataforma utilizada

O pipeline foi desenvolvido no Databricks, uma plataforma de dados em nuvem que utiliza o Apache Spark para processamento distribuído.

Os dados foram armazenados em Volumes do ambiente Databricks e processados por notebooks utilizando PySpark.

### 2.2 Fontes de dados

As principais fontes utilizadas no MVP são:

- dados de anamnese preenchidos pelos atletas;
- arquivos CSV exportados do TrainingPeaks;
- tabela de associação entre arquivos de treino e atletas;
- tabelas internas de regras de suplementação;
- tabelas internas de protocolos crônicos.

### 2.3 Dados de anamnese

Os dados de anamnese são utilizados para compor informações como:

- características gerais do atleta;
- condições clínicas relatadas;
- desempenho de corrida;
- hábitos de suplementação;
- hidratação e nutrição;
- informações necessárias para a triagem.

Os dados identificáveis não devem ser publicados no repositório público.

### 2.4 Dados do TrainingPeaks

Os treinamentos são recebidos em arquivos CSV exportados do TrainingPeaks.

Entre os campos disponíveis nos arquivos estão:

- título do treino;
- tipo de treino;
- descrição;
- duração planejada;
- distância planejada;
- data do treino;
- duração realizada;
- distância realizada;
- frequência cardíaca média;
- frequência cardíaca máxima;
- TSS;
- zonas de frequência cardíaca;
- zonas de potência;
- percepção de esforço;
- sensação relatada.

Os arquivos são armazenados no Volume do Databricks antes da transformação.

### 2.5 Associação entre arquivo e atleta

A associação entre os arquivos de treinamento e os atletas é feita por meio da tabela de mapeamento:

`workspace.default.dim_atleta_trainingpeaks`

Essa tabela utiliza um termo de busca relacionado ao nome ou padrão presente no caminho do arquivo.

Após a associação, o treinamento recebe o respectivo `id_atleta`, que é utilizado nas demais tabelas do pipeline.

### 2.6 Privacidade

Os dados utilizados no projeto contêm informações pessoais e potencialmente sensíveis.

Por esse motivo:

- os dados brutos não serão publicados no GitHub;
- os arquivos reais de anamnese não serão disponibilizados;
- os arquivos reais do TrainingPeaks não serão disponibilizados;
- os PDFs reais não serão publicados;
- dados identificáveis devem permanecer restritos ao ambiente de processamento;
- o repositório público conterá somente código, documentação e evidências anonimizadas.

## 3. Modelagem e Catálogo de Dados

### 3.1 Estratégia de modelagem

O projeto utiliza uma organização baseada em dimensões, fatos e tabelas consolidadas.

As dimensões armazenam informações descritivas dos atletas, suas características e regras de negócio.

As tabelas fato armazenam eventos ou ocorrências, como:

- sessões de treinamento;
- alertas clínicos;
- documentos gerados.

A tabela Gold consolida as informações necessárias para a geração das orientações por sessão.

### 3.2 Identificação analítica do atleta

O pipeline utiliza `id_atleta` como identificador analítico.

A tabela com nomes restritos é mantida separadamente da estrutura analítica:

`workspace.default.dim_atleta_nome_restrito`

Essa separação reduz a exposição de informações identificáveis nas tabelas utilizadas para análise.

### 3.3 Principais tabelas

As principais tabelas criadas no MVP são:

- `dim_atleta`;
- `dim_atleta_nome_restrito`;
- `dim_atleta_trainingpeaks`;
- `dim_condicao_clinica`;
- `dim_desempenho_corrida`;
- `dim_habitos_suplementacao`;
- `dim_hidratacao_nutricao`;
- `fato_sessao_treino`;
- `fato_sessao_treino_zonas`;
- `fato_alerta_clinico`;
- `dim_regra_suplementacao_zona`;
- `dim_protocolo_suplementacao_base`;
- `gold_prescricao_diaria`;
- `fato_documento_prescricao`.

O catálogo detalhado de cada tabela será apresentado no arquivo:

`docs/catalogo_de_dados.md`

## 4. Pipeline de Dados

### 4.1 Notebooks utilizados

O pipeline foi organizado nos seguintes notebooks:

1. `00_validacao_pipeline`
2. `02_transformacao_silver_hyrox`
3. `03_dimensoes_prescricao`
4. `04_ingestao_trainingpeaks`
5. `05_regras_clinicas_prescricao`
6. `06_geracao_pdf_prescricao`

### 4.2 Fluxo lógico

```text
Arquivos brutos
      ↓
Volumes do Databricks
      ↓
Validação inicial
      ↓
Limpeza e padronização
      ↓
Dimensões clínicas, esportivas e nutricionais
      ↓
Ingestão dos treinos
      ↓
Associação entre atleta e treino
      ↓
Zonas e características da sessão
      ↓
Alertas clínicos
      ↓
Regras de suplementação e hidratação
      ↓
Gold de prescrição
      ↓
Geração do PDF
      ↓
Registro de rastreabilidade
