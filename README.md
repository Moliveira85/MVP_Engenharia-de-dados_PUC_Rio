# MVP — Pipeline de Dados para Prescrição Individualizada em Treinamentos Hyrox

## 1. Contexto de Negócios e Perguntas

### 1.1 Contexto

Este projeto apresenta um MVP de engenharia de dados desenvolvido na plataforma Databricks para integrar informações de atletas, dados clínicos, hábitos de suplementação, hidratação, desempenho e sessões de treinamento.

O projeto foi desenvolvido para atletas de treinamento híbrido e Hyrox, considerando a necessidade de relacionar as características individuais do atleta às características de cada sessão de treinamento.

O pipeline recebe dados provenientes de:

- formulários de anamnese esportiva e clínica;
- arquivos de treinamento exportados do TrainingPeaks;
- tabelas dimensionais de regras nutricionais balizadas pela literatura esportiva;
- protocolos crônicos basais de suplementação.

Após a ingestão, os dados são armazenados, transformados, padronizados e relacionados por meio de tabelas dimensionais e factuais em arquitetura Medalhão (Bronze, Silver e Gold). Como resultado, é produzida uma tabela analítica Gold com orientações parametrizadas para as sessões de treinamento e relatórios operacionais em PDF destinados à revisão profissional.

O sistema não substitui a avaliação de profissionais habilitados. Seu objetivo é demonstrar a construção de um pipeline de dados reprodutível, com integração de fontes, aplicação de regras de negócio, validação e geração de uma saída operacional escalável.

### 1.2 Problema de Negócio

Informações de anamnese, características individuais e sessões de treinamento normalmente encontram-se distribuídas em diferentes fontes e formatos não estruturados ou semiestruturados.

Essa fragmentação dificulta:

- a associação automatizada entre o atleta e seus respectivos treinamentos;
- a análise conjunta de informações clínicas, restrições médicas e métricas esportivas;
- a aplicação consistente e segura de regras de hidratação, calorias e suplementação aguda/crônica;
- a rastreabilidade e governança das orientações produzidas;
- a revisão padronizada dos documentos antes do repasse final ao atleta.

O problema abordado pelo MVP é construir uma arquitetura de dados em nuvem capaz de integrar essas fontes heterogêneas e produzir uma saída organizada, individualizada, rastreável e segura para apoiar a revisão técnica especializada.

### 1.3 Objetivo Geral

Construir um pipeline de dados em nuvem, utilizando a plataforma Databricks e processamento distribuído com Apache Spark (PySpark), capaz de integrar dados cadastrais, clínicos e de treinamento, aplicar regras parametrizadas de hidratação e suplementação com margem de segurança conservadora, e consolidar uma camada Gold pronta para consumo analítico e geração automatizada de relatórios em PDF.

### 1.4 Perguntas de Negócio

O projeto responde às seguintes perguntas ao longo da esteira analítica:

1. O pipeline consegue associar corretamente cada sessão de treinamento ao atleta correspondente de forma automatizada?
2. Quantos atletas, sessões de treinamento, zonas de esforço e prescrições foram consolidados no MVP?
3. Como as sessões de treinamento se distribuem por modalidade, duração, distância, intensidade e faixas cardíacas (Z1 a Z5)?
4. Quais alertas clínicos e potenciais contraindicações foram identificados durante o processo de triagem?
5. Como os parâmetros individuais do atleta (peso, tolerância) e as demandas da sessão são combinados nas regras de suplementação?
6. O pipeline consolida as informações em uma tabela Gold íntegra e pronta para relatórios?
7. O sistema produz orientações individualizadas e detalhadas por sessão de treinamento?
8. O sistema gera documentos executivos individuais em PDF para validação e auditoria profissional?
9. Quais critérios e testes de qualidade de dados foram aplicados nas tabelas?
10. Como o pipeline garante a integridade referencial e a ausência de duplicidades?
11. Quais limitações técnicas permanecem no MVP e quais as evoluções futuras recomendadas?

### 1.5 Escopo do MVP

O escopo deste MVP contempla:

- ingestão de dados de anamnese e isolamento cadastral;
- ingestão e enriquecimento de planilhas de treinamento do TrainingPeaks;
- identificação determinística e associação dos atletas via chaves relacionais;
- organização dos dados sob modelagem dimensional (Kimball) e armazenamento em Delta Lake;
- criação de dimensões clínicas, esportivas, nutricionais e regras de negócio;
- criação de tabelas de fatos transacionais (sessões de treino, distribuição de zonas cardíacas e alertas clínicos);
- motor de inferência para suplementação aguda, crônica e reposição de eletrólitos/carboidratos;
- consolidação analítica na tabela gerenciada `gold_prescricao_diaria`;
- geração automatizada de relatórios consolidados em PDF via biblioteca ReportLab;
- validação de qualidade dos dados (integridade referencial e unicidade de chaves primárias).

A execução atual do pipeline é realizada de forma orquestrada por meio de notebooks modulares no Databricks. A orquestração agendada via Databricks Workflows/Jobs permanece mapeada como evolução futura.

---

## 2. Ingestão, Governança e Carga de Dados

### 2.1 Plataforma e Nuvem

O pipeline foi implementado no ecossistema Databricks, usufruindo da capacidade de computação elástica do Apache Spark (PySpark) e do armazenamento em tabelas Delta Lake com garantias transacionais ACID.

Os arquivos de entrada brutos e os relatórios compilados são persistidos em volumes gerenciados pelo Databricks Unity Catalog (`workspace.default`).

### 2.2 Fontes de Dados

As fontes primárias utilizadas na solução são:

- Formulários de anamnese clínica e esportiva (dados semiestruturados de perfil, rotina e histórico);
- Arquivos tabulares (CSV) exportados da plataforma TrainingPeaks contendo telemetria esportiva;
- Tabelas internas de mapeamento de identificadores de arquivos e atletas;
- Tabelas dimensionais de regras nutricionais e alocação de suplementos esportivos por zonas de esforço;
- Diretrizes padronizadas para protocolos de uso crônico basal.

### 2.3 Dados de Anamnese e Triagem Clínica

Os dados de anamnese fornecem atributos fundamentais para a individualização do cálculo metabólico:

- características antropométricas (peso corporal, altura, data de nascimento/idade);
- histórico de condições médicas preexistentes e intolerâncias;
- métricas de desempenho de corrida (referenciais de 5 km e 10 km);
- hábitos de uso de suplementos e sensibilidade gástrica/estimulantes;
- perfil de sudorese percebida e queixas gastrointestinais em treinos prévios.

### 2.4 Dados de Sessões de Treino (TrainingPeaks)

Os treinos contêm métricas objetivas de carga externa e interna registradas por relógios e monitores cardíacos:

- data do treino, título e modalidade esportiva (Run, Functional/Strength);
- duração planejada e duração realizada (padronizadas em minutos);
- distância percorrida planejada e executada (em metros e km);
- frequência cardíaca média e máxima atingidas na sessão;
- métricas de carga e estresse de treinamento (TSS, IF);
- tempo acumulado em cada zona de frequência cardíaca (HRZone1Minutes a HRZone5Minutes);
- percepção subjetiva de esforço (RPE) e percepção de sensação relatada.

### 2.5 Associação de Atletas e Integridade

A vinculação entre os arquivos do TrainingPeaks e os perfis cadastrais é gerenciada via tabela de mapeamento:

`workspace.default.dim_atleta_trainingpeaks`

A rotina avalia os padrões textuais dos arquivos depositados nos Volumes e associa a sessão ao respectivo `id_atleta` de forma determinística, impedindo a injeção de treinos anônimos ou descontextualizados na camada Gold.

### 2.6 Privacidade e Conformidade com a LGPD

Considerando a natureza sensível dos dados clínicos e esportivos:

- Nomes civis completos e identificadores diretos foram segregados na tabela de acesso restrito `dim_atleta_nome_restrito`.
- As tabelas de dimensões públicas, tabelas de fatos e a tabela Gold utilizam exclusivamente identificadores pseudonimizados gerados por algoritmo hash criptográfico SHA-256 (`id_atleta`).
- Nenhum dado pessoal identificável (PII), arquivo de treino bruto real ou documento PDF contendo nomes de atletas é versionado no repositório público do GitHub.
- Todas as evidências visuais incluídas na documentação foram tratadas e anonimizadas.

---

## 3. Modelagem e Catálogo de Dados

### 3.1 Estratégia de Modelagem

O projeto adota a modelagem dimensional (Kimball) integrada à arquitetura Medalhão:

- **Camada Bronze:** Cargas dos arquivos de anamnese em estado bruto (`bronze_anamnese_raw`).
- **Camada Silver (Dimensões):** Entidades descritivas, histórico do atleta e tabelas de parametrização de regras de negócio.
- **Camada Silver (Fatos):** Eventos transacionais granulares com chaves substitutas e métricas numéricas acumuladas.
- **Camada Gold:** Visão analítica materializada agregando perfil do atleta, características da sessão, dosagens calculadas e alertas de segurança.

### 3.2 Principais Tabelas Criadas no Unity Catalog

#### Dimensões e Regras de Negócio
- `dim_atleta`: Atributos antropométricos basais (peso, altura, idade) e objetivos competitivos.
- `dim_atleta_nome_restrito`: Chave relacional vinculada aos dados cadastrais sensíveis (PII isolada).
- `dim_atleta_trainingpeaks`: Mapeamento de termos e diretórios para associação determinística dos treinos.
- `dim_condicao_clinica`: Histórico de diagnósticos médicos e restrições funcionais.
- `dim_desempenho_corrida`: Referenciais de ritmo (pace) e tempos basais em distâncias padrão.
- `dim_habitos_suplementacao`: Histórico de suplementação em uso e tolerância a substâncias estimulantes.
- `dim_hidratacao_nutricao`: Comportamento hídrico, histórico de sintomas térmicos e desconfortos gastrointestinais.
- `dim_regra_nutricao_treino`: Matriz científica parametrizada de taxas de carboidrato e hidratação por intensidade e tempo.
- `dim_regra_suplementacao_zona`: Regras de indicação de cafeína, eletrólitos e tamponantes conforme zonas de esforço predominantes.
- `dim_protocolo_suplementacao_base`: Diretrizes para suplementação de saturação e uso crônico contínuo.

#### Tabelas de Fatos
- `fato_sessao_treino`: Sessões individuais com duração, distância, FC média/máxima, métricas de carga e modalidade.
- `fato_sessao_treino_zonas`: Estratificação precisa dos minutos de permanência em cada faixa cardíaca (Z1 a Z5).
- `fato_alerta_clinico`: Ocorrências de incompatibilidade fisiológica ou contraindicações farmacológicas detectadas.

#### Camada Analítica
- `gold_prescricao_diaria`: Tabela analítica consolidada integrando treinos, recomendações nutricionais individualizadas (g/h de carboidratos, sódio, volume hídrico e cafeína por kg de peso), suplementação crônica diária e pareceres de segurança.

---

## 4. Pipeline de Dados e Engenharia

### 4.1 Estrutura Modular de Notebooks

O pipeline é composto por 7 notebooks modulares desenvolvidos em PySpark, localizados no diretório `notebooks/`:

1. `00_validacao_ambiente.py`: Teste de conectividade com o cluster, validação do catálogo, schema e volumes de armazenamento.
2. `02_silver_anamnese_dimensoes.py`: Ingestão dos dados de anamnese, higienização, pseudonimização com hash SHA-256 e carga das tabelas de dimensões.
3. `03_silver_treinos_fatos.py`: Ingestão dos arquivos brutos do TrainingPeaks, conversão de unidades, tratamento de durações e criação da `fato_sessao_treino`.
4. `04_silver_treinos_zonas_fc.py`: Decomposição e processamento das distribuições de frequências cardíacas, gerando a `fato_sessao_treino_zonas`.
5. `05_gold_prescricao_diaria.py`: Execução das regras de negócio esportivas, cruzamento com o perfil do atleta e consolidação da tabela Gold.
6. `06_geracao_pdf_prescricao.py`: Compilação dinâmica de relatórios em PDF com formatação técnica via ReportLab e persistência em Volumes gerenciados.
7. `07_analise_final_mvp.py`: Validação cruzada de ponta a ponta, auditoria de integridade referencial, testes de unicidade e apresentação de indicadores consolidados.

### 4.2 Fluxo Lógico do Pipeline

```text
[ Formulários de Anamnese ]           [ Exportações TrainingPeaks ]
            │                                      │
            ▼                                      ▼
[ Volume: dados_raw ]                 [ Volume: treinos_trainingpeaks ]
            │                                      │
            ▼                                      ▼
  [ 02_silver_anamnese ]                [ 03_silver_treinos_fatos ]
   - Limpeza e Normalização              - Padronização de Métricas
   - Hashing SHA-256 (LGPD)              - Conversão Horas -> Minutos
            │                                      │
            ├──────────────────────┬───────────────┤
            ▼                      ▼               ▼
     [ Dimensões ]     [ 04_silver_zonas ]   [ Fato Sessões ]
   - dim_atleta        - fato_sessao_zonas   - fato_sessao_treino
   - dim_clinica, etc.         │                   │
            │                  └───────────┬───────┘
            │                              │
            └──────────────┬───────────────┘
                           ▼
              [ 05_gold_prescricao_diaria ]
               - Cruzamento de Peso e Zonas Predominantes
               - Taxas de Carboidrato (g/h) e Eletrólitos
               - Auditoria e fato_alerta_clinico
                           │
                           ▼
              [ 06_geracao_pdf_prescricao ]
               - Compilação dos Relatórios em PDF (ReportLab)
               - Armazenamento em Volumes (prescricoes_pendentes)
                           │
                           ▼
              [ 07_analise_final_mvp ]
               - Testes de Integridade (Órfãos = 0, Duplicatas = 0)
               - Relatório de Métricas Globais do MVP
