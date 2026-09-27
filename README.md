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

### 2.6 Tratamento e publicação de dados pessoais

O MVP utiliza dados de anamnese esportiva e clínica, incluindo informações pessoais e de saúde. O autor informa ter autorização das pessoas participantes para a publicação dos dados utilizados nesta demonstração. A documentação dessas autorizações e seu alcance devem ser conferidos antes da entrega e de futuras publicações.

O pipeline utiliza `id_atleta` como chave de associação entre as tabelas. Esse identificador não torna anônimos os registros que contenham, na mesma linha ou nas saídas dos notebooks, nomes ou outros dados que permitam identificar uma pessoa.

Na execução publicada do notebook de análise final, a tabela `gold_prescricao_diaria` inclui a coluna `nome_completo`, e as saídas do notebook exibem dados individuais e nomes de arquivos PDF. Portanto, o repositório **não deve ser descrito como integralmente anonimizado**. A publicação desses elementos exige atenção específica à autorização concedida, ao alcance da exposição e à necessidade de cada dado exibido.

Os relatórios PDF são gerados no ambiente Databricks para revisão profissional; sua existência no Volume não significa aprovação clínica nem envio automático aos atletas.
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
- `dim_atleta_nome_restrito`: Chave relacional vinculada aos dados cadastrais.
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

### 4.1 Estrutura dos notebooks publicados

Os notebooks do MVP estão no diretório [`Notebooks/`](Notebooks/). Os nomes abaixo correspondem aos arquivos publicados no repositório:

1. [`00_validacao_pipeline.ipynb`](Notebooks/00_validacao_pipeline.ipynb) — validações do pipeline e do ambiente.
2. [`02_transformacao_silver_hyrox.ipynb`](Notebooks/02_transformacao_silver_hyrox.ipynb) — transformações da camada Silver.
3. [`03_dimensaoes_prescricao.ipynb`](Notebooks/03_dimensaoes_prescricao.ipynb) — dimensões utilizadas na prescrição.
4. [`04_ingestao_trainingpeaks.ipynb`](Notebooks/04_ingestao_trainingpeaks.ipynb) — ingestão e tratamento dos dados de treinamento.
5. [`05_regras_clinicas_prescricao.ipynb`](Notebooks/05_regras_clinicas_prescricao.ipynb) — regras e alertas aplicados à prescrição.
6. [`06_geracao_pdf_prescricao.ipynb`](Notebooks/06_geracao_pdf_prescricao.ipynb) — geração dos documentos PDF para revisão.
7. [`07_analise_final_mvp.ipynb`](Notebooks/07_analise_final_mvp.ipynb) — consultas de validação e apresentação dos resultados do MVP.

Os números identificam os arquivos publicados; não devem ser interpretados, por si só, como garantia de execução automática ou agendada. A execução agendada por Databricks Workflows/Jobs permanece como evolução futura.

### 4.2 Fluxo funcional do MVP

1. Os dados de anamnese e os arquivos de treinamento são disponibilizados no ambiente Databricks.
2. Os notebooks de transformação e ingestão organizam as informações dos atletas e das sessões em tabelas relacionáveis por `id_atleta`.
3. As dimensões e regras clínicas apoiam o processamento das orientações por sessão.
4. Os resultados são consolidados em `gold_prescricao_diaria`, que inclui características dos treinos, textos de orientação e campos de alerta/revisão.
5. O notebook de geração de PDF produz documentos no ambiente para revisão profissional.
6. O notebook de análise final consulta tabelas e saídas do pipeline, apresentando contagens, verificações de chaves, características dos treinos e alertas.

A execução documentada ainda não demonstrou uma tabela de controle de documentos: `fato_documento_prescricao` não foi encontrada na validação publicada. Os PDFs encontrados no Volume não devem ser confundidos com registros nessa tabela.

## 5. Análise dos resultados e respostas às perguntas de negócio

Os resultados a seguir correspondem à execução registrada em 20/09/2026 no notebook [`07_analise_final_mvp.ipynb`](Notebooks/07_analise_final_mvp.ipynb). Eles representam uma execução do MVP, não um resultado generalizável para todos os atletas ou para futuras cargas de dados.

1. **Associação entre sessão e atleta.** As quatro sessões registradas têm `id_atleta` preenchido; uma pessoa cadastrada aparece com sessões. Isso demonstra a associação na amostra analisada, mas não testa, por si só, todos os possíveis arquivos ou atletas.

2. **Volume consolidado.** A execução mostrou três atletas na dimensão principal, quatro registros de sessões, dois registros na tabela de zonas, oito alertas clínicos e quatro registros na Gold. Esses números representam registros de tabelas: quatro linhas na Gold não significam quatro atletas atendidos.

3. **Distribuição e métricas dos treinos.** As quatro sessões da amostra estão classificadas como `Run`. Duração e distância aparecem preenchidas nas quatro linhas. FC média, FC máxima e TSS não apresentam valores no resumo publicado; portanto, essa execução não sustenta uma discussão quantitativa dessas três métricas. A disponibilidade de apenas dois registros de zonas também limita a análise de intensidade por faixa cardíaca.

4. **Alertas clínicos.** Foram registrados oito alertas: três de segurança cardiovascular, três relacionados a fármacos e dois de alergia/intolerância. Cinco estão classificados como críticos e três como altos; todos aparecem como pendentes de revisão farmacêutica. São sinais para avaliação profissional, não diagnósticos clínicos produzidos pelo pipeline.

5. **Combinação de parâmetros individuais e sessão.** A Gold apresenta características individuais e da sessão ao lado de textos de orientação de carboidratos, sódio, cafeína e protocolo crônico. Essa saída demonstra a consolidação de dados e regras no MVP, mas não comprova, isoladamente, adequação clínica ou liberação das orientações para uso.

6. **Integridade da Gold.** A tabela `gold_prescricao_diaria` foi encontrada e continha quatro registros na execução publicada. Ela serve como saída analítica e operacional para revisão. A presença da tabela não elimina a necessidade de conferir qualidade, duplicidade lógica e alertas antes de utilizar seus registros.

7. **Orientações por sessão.** Há textos de orientação associados às linhas da Gold. Nas linhas com `RISCO_CRITICO`, o parecer informa pendência de validação clínica individual; esses textos não devem ser apresentados como prescrições aprovadas ou entregues automaticamente.

8. **Documentos PDF.** O notebook encontrou arquivos PDF no Volume de prescrições pendentes, demonstrando a existência de saídas documentais no ambiente. A tabela `fato_documento_prescricao` não foi encontrada naquela validação; portanto, não há, nessa evidência, comprovação de rastreabilidade dos PDFs por uma tabela de documentos.

9. **Qualidade dos dados.** O notebook verifica a existência e a contagem de tabelas, a presença de `id_atleta` nas sessões e a unicidade de `id_treino`. Esses testes cobrem aspectos importantes do MVP, mas não equivalem a uma validação completa de todos os atributos clínicos, métricas e regras.

10. **Integridade referencial e duplicidades.** A verificação encontrou zero sessões sem `id_atleta` e quatro `id_treino` distintos para quatro linhas. Entretanto, a amostra da Gold exibe sessões com mesma data, título, duração e distância sob identificadores diferentes. Assim, a ausência de IDs repetidos **não comprova** ausência de sessões duplicadas do ponto de vista de negócio; essa ocorrência permanece para investigação.

11. **Limitações e evolução.** A execução envolve três atletas cadastrados, mas sessões associadas a apenas um deles; não demonstra variedade de modalidades e não apresenta valores de FC média, FC máxima ou TSS no resumo publicado. Permanecem como pontos de evolução a verificação de duplicidade lógica, a rastreabilidade dos PDFs em tabela, a ampliação das validações e a eventual orquestração agendada.

## 6. Autoavaliação

**Objetivo e problema de negócio — atendido na documentação.** O projeto explicita o problema de integrar anamnese, dados de treino e regras de orientação, além de apresentar perguntas que direcionam a análise.

**Armazenamento em nuvem — demonstrado no ambiente do MVP.** A implementação utiliza Databricks, tabelas no catálogo e Volumes para arquivos de entrada e documentos de saída. O README deve distinguir os componentes descritos daqueles efetivamente verificados nas evidências publicadas.

**Modelagem de dados — implementada, com ajustes documentais necessários.** Há dimensões, fatos e uma tabela Gold. O inventário de tabelas e as afirmações sobre segregação de dados pessoais precisam corresponder ao esquema e às saídas efetivamente publicados.

**Pipeline, transformação e carga — implementados em notebooks.** Os arquivos publicados documentam etapas de ingestão, transformação, aplicação de regras, geração de PDF e validação. A execução agendada em Jobs não faz parte do escopo demonstrado nesta versão.

**Análise e discussão — demonstradas em amostra limitada.** O notebook apresenta contagens, verificações e amostras de saída. A interpretação dos resultados deve respeitar a cobertura observada: somente um atleta com sessões na execução, quatro registros de treino, métricas cardíacas ausentes no resumo e necessidade de investigar possível repetição lógica de sessões.

**Segurança da saída operacional — depende de revisão profissional.** A existência de alertas críticos e pareceres pendentes reforça que os textos de orientação e os PDFs são material para conferência, não validação clínica automática nem comprovante de entrega ao atleta.

**Síntese.** O MVP demonstra um pipeline funcional e um caso aplicado de engenharia de dados. Seus principais pontos de melhoria são alinhar a documentação à implementação observada, ampliar os testes de qualidade, esclarecer o tratamento das sessões aparentemente repetidas e tornar explícito o controle das orientações diante de alertas críticos.
