# Tabelas

# 02. Catálogo de Tabelas e Modelagem Dimensional

Esta pasta documenta a estrutura do catálogo de dados gerenciado via Databricks Unity Catalog (`workspace.default`), comprovando a modelagem dimensional (Kimball) e a organização dos dados em camadas (Medalhão).

---

### Evidências 

- **`01_tabelas_dimensoes_unity_catalog.png`**: Visualização do esquema no Unity Catalog apresentando a camada de entrada bruta (`bronze_anamnese_raw`) e as tabelas de dimensões cadastrais, clínicas e de hábitos esportivos.
- **`02_tabelas_fatos_gold_volumes.png`**: Visualização das tabelas de regras de negócio, tabelas de fatos transacionais de treinos e alertas, a tabela analítica consolidada (`gold_prescricao_diaria`) e os Databricks Volumes configurados.

---

### Arquitetura e Estrutura de Tabelas

A modelagem foi concebida para atender tanto à rastreabilidade analítica quanto às normas de privacidade (LGPD):

#### 1. Camada Bronze
- `bronze_anamnese_raw`: Tabela de ingestão direta dos dados brutos de anamnese.

#### 2. Dimensões (Camada Silver)
- `dim_atleta`: Contém atributos antropométricos (peso, altura, data de nascimento), tempo de experiência e objetivos de prova.
- `dim_atleta_nome_restrito`: Tabela com isolamento de PII (nome completo associado ao identificador pseudonimizado).
- `dim_condicao_clinica`: Registros de histórico médico e condições preexistentes.
- `dim_desempenho_corrida`: Marcas de tempo de referência (5 km e 10 km) e ritmos basais.
- `dim_habitos_suplementacao`: Histórico de suplementos em uso e sensibilidade a estimulantes.
- `dim_hidratacao_nutricao`: Estratégia hídrica declarada, sintomas em provas longas e percepção de taxa de sudorese.
- `dim_regra_nutricao_treino`: Matriz de dosagens nutricionais balizada pela literatura esportiva.
- `dim_regra_suplementacao_zona`: Regras de alocação de suplementos conforme zonas fisiológicas de esforço.

#### 3. Tabelas de Fatos (Camada Silver)
- `fato_sessao_treino`: Sessões individuais de treino contendo distância, tempo de movimento, métricas de carga (TSS) e frequências cardíacas (média e máxima).
- `fato_sessao_treino_zonas`: Distribuição temporal das sessões estratificada por faixas cardíacas (Z1 a Z5).
- `fato_alerta_clinico`: Identificação automatizada de potenciais contraindicações e avisos de segurança farmacológica.

#### 4. Camada Analítica (Gold)
- `gold_prescricao_diaria`: Tabela analítica consolidada que combina as demandas energéticas da sessão com o perfil individual do atleta, gerando o protocolo diário recomendado.

#### 5. Storage Não Estruturado (Databricks Volumes)
- `dados_raw`: Repositório de arquivos brutos de entrada.
- `treinos_trainingpeaks`: Volume contendo as planilhas de treino e as pastas de armazenamento dos relatórios em PDF (`prescricoes_pendentes` e `prescricoes_aprovadas`).
