
# 03. Pipeline de Ingestão, Processamento e Transformação (ETL)

Esta pasta reúne as evidências da execução do pipeline de dados no Databricks, comprovando o fluxo completo de extração, limpeza, enriquecimento e consolidação dos dados esportivos e clínicos em arquitetura Medalhão (Bronze, Silver e Gold).

---

### Evidências

- **`01_validacao_tabelas.png`**: Comprovação da validação estrutural do pipeline no catálogo `workspace.default`, confirmando a existência, acessibilidade e contagem de registros das tabelas processadas na esteira analítica.
- **`02_indicadores_gerais.png`**: Resumo da execução do pipeline demonstrando a consolidação das cargas de dados: atletas cadastrados, sessões de treino ingeridas, sessões mapeadas por faixas cardíacas, alertas clínicos disparados e prescrições diárias compiladas na camada Gold.

---

### Etapas e Fluxo do Pipeline (ETL)

O pipeline foi estruturado em notebooks modulares desenvolvidos em PySpark, executando o seguinte ciclo de processamento distribuído:

#### 1. Ingestão Bruta e Isolamento Cadastral (Bronze -> Silver)
- Leitura dos arquivos de anamnese depositados no Databricks Volumes (`dados_raw`).
- Aplicação de rotinas de higienização de tipos, normalização de textos e cálculo de idade.
- Geração de identificadores pseudonimizados com função hash criptográfica SHA-256 (`id_atleta`), segregando dados sensíveis e pessoais (PII) em conformidade com as diretrizes da LGPD.
- Carga das tabelas dimensionais: `dim_atleta`, `dim_atleta_nome_restrito`, `dim_condicao_clinica`, `dim_desempenho_corrida`, `dim_habitos_suplementacao` e `dim_hidratacao_nutricao`.

#### 2. Processamento das Sessões de Treino (TrainingPeaks -> Silver)
- Ingestão dos arquivos de exportação do TrainingPeaks armazenados no volume `treinos_trainingpeaks`.
- Limpeza de colunas temporais, tratamento de durações em horas para minutos, padronização de distâncias e cálculo de métricas de intensidade (TSS, FC Média, FC Máxima).
- Extração dos minutos acumulados por faixas cardíacas (Z1 a Z5) e carga da tabela `fato_sessao_treino_zonas`.
- Persistência das sessões transacionais na tabela gerenciada `fato_sessao_treino` em formato Delta Lake.

#### 3. Motor de Regras e Alertas Fisiológicos (Silver -> Gold)
- Cruzamento dimensional das sessões com o peso corporal do atleta e as matrizes de recomendação esportiva (`dim_regra_nutricao_treino` e `dim_regra_suplementacao_zona`).
- Cálculo das taxas horárias e totais de carboidratos, hidratação hídrica basal e reposição de eletrólitos (sódio), adotando princípios de margem de segurança clínica conservadora.
- Avaliação de contraindicações e interações (ex.: sensibilidade a estimulantes versus dosagem aguda de cafeína), alimentando a tabela `fato_alerta_clinico`.
- Consolidação dos registros finais na tabela analítica `gold_prescricao_diaria`.
