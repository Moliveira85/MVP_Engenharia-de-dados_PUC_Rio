
# 05. Camada Gold e Análise Consolidada

Esta pasta documenta a camada analítica final do projeto (`gold_prescricao_diaria`), evidenciando a materialização das regras de negócio, a consolidação dos dados de treino com as diretrizes nutricionais e os indicadores gerais do MVP.

---

### Evidências Fotográficas

- **`01_tabela_gold_prescricao.png`**: Exibição da tabela analítica gerenciada `gold_prescricao_diaria`, demonstrando os registros enriquecidos com data, modalidade, duração em minutos, peso corporal, zona cardíaca predominante, taxas de carboidratos (`cho_g_h` e `cho_total_g`), eletrólitos, cafeína, suplementação crônica diária e os alertas clínicos vinculados.
- **`02_resumo_executivo_mvp.png`**: Resumo executivo com os quantitativos globais do pipeline consolidado: contagem de atletas cadastrados, volume total de sessões ingeridas, sessões mapeadas por zonas de intensidade, alertas clínicos processados e prescrições diárias compiladas.

---

### Aspectos Analíticos e Regras de Negócio

A camada Gold representa o ponto de convergência de todo o pipeline, agregando valor aos dados brutos por meio de:

#### 1. Cálculo Energético e Nutricional Personalizado
- **Carboidratos intra-treino:** Determinação da taxa de carboidratos por hora baseada na duração e na zona de frequência cardíaca predominante (Z1 a Z5), adotando abordagem de dosagem conservadora inicial para evitar desconfortos gastrointestinais.
- **Eletrólitos e Hidratação:** Estimativa de reposição de sódio e volume hídrico em função da duração e condições climáticas/intensidade.
- **Suplementação Aguda e Crônica:** Mapeamento de dosagens de cafeína (com ajuste por kg de peso), bicarbonato de sódio, creatina e beta-alanina, organizados por protocolo diário.

#### 2. Segurança e Farmacovigilância Esportiva
- Cruzamento automatizado com a tabela `fato_alerta_clinico`.
- Sinalização visual e parecer farmacêutico em caso de contraindicações relatadas pelo atleta (ex.: sensibilidade gástrica a estimulantes ou restrições renais), priorizando a segurança clínica.

#### 3. Governança e Anonimização
- Todas as chaves analíticas e relacionais utilizam identificadores criptográficos pseudonimizados (`id_atleta` via hash SHA-256 e `id_treino`), garantindo a preservação total da privacidade dos dados pessoais (LGPD).
