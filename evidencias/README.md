
# Evidências de Execução do MVP

Este diretório reúne os registros visuais e as comprovações técnicas da execução do pipeline de dados no Databricks, organizado em etapas sequenciais que cobrem desde a infraestrutura até a entrega do resultado final.

---

### Estrutura das Evidências

- **`01_ambiente/`**: Comprovações da infraestrutura em nuvem, execução em Databricks Compute e validação do ambiente.
- **`02_tabelas/`**: Catálogo de dados no Unity Catalog, evidenciando as camadas Bronze, Silver (dimensões e fatos) e Gold.
- **`03_pipeline/`**: Execução do fluxo de ingestão, tratamento e consolidação dos dados via PySpark.
- **`04_Qualidade/`**: Testes de consistência referencial (zero sessões sem atleta) e unicidade de registros (zero duplicidades).
- **`05_Analise/`**: Tabela analítica consolidada (`gold_prescricao_diaria`) e resumo quantitativo do processamento.
- **`06_Resultados/`**: Confirmação da compilação e armazenamento dos relatórios de prescrição em PDF no Databricks Volumes.

---

*Nota: Todas as evidências foram capturadas a partir de execuções reais no ambiente de nuvem e anonimizadas para garantir a conformidade com a LGPD.*
