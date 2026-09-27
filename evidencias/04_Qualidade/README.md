
# 04. Qualidade e Integridade dos Dados

Esta pasta documenta os testes automatizados de qualidade, consistência referencial e integridade relacional executados sobre as tabelas do pipeline no Databricks, garantindo confiabilidade analítica e conformidade clínica.

---

### Evidências

- **`01_sessoes_sem_atleta.png`**: Teste de integridade referencial validando que não existem sessões de treino órfãs (`fato_sessao_treino` sem correspondente na `dim_atleta`). Resultado: 0 registros órfãos encontrados.
- **`02_duplicidade_treinos.png`**: Teste de unicidade de chave primária (`id_treino`) na tabela `fato_sessao_treino`, atestando a ausência de duplicidade nas cargas de ingestão. Resultado: 0 registros duplicados.

---

### Regras de Qualidade Implementadas

As rotinas de data quality foram incorporadas no notebook de análise final (`07_analise_final_mvp.py`) e cobrem as seguintes dimensões de qualidade de dados:

#### 1. Integridade Referencial (Foreign Key Check)
- Todas as sessões ingeridas a partir dos arquivos brutos do TrainingPeaks passam por cruzamento estrito com os cadastros validados na camada de dimensões.
- Sessões sem atleta associado não avançam para a camada Gold, prevenindo prescrições descontextualizadas ou com parâmetros antropométricos nulos.

#### 2. Unicidade de Identificadores (Primary Key Check)
- Avaliação da cardinalidade do identificador único da sessão (`id_treino`).
- Garantia de que reprocessamentos ou sobreposições de arquivos de exportação não gerem duplicidade de treinos na tabela de fatos.

#### 3. Consistência Fisiológica e Limites Operacionais
- Validação de faixas plausíveis para frequência cardíaca (mínima, média e máxima).
- Conversão padronizada de durações temporais em horas fracionárias para minutos inteiros, assegurando a precisão dos cálculos de taxa horária de carboidratos e hidratação.

#### 4. Auditoria de Segurança Clínica
- Verificação de consistência entre alertas de sensibilidade e regras de suplementação aguda, assegurando que atletas com contraindicações registradas na `fato_alerta_clinico` recebam sinalização e conduta conservadora.
