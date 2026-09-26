
# 06. Resultados e Artefatos Finais

Esta pasta reúne as evidências do resultado final do MVP, demonstrando a geração automatizada dos arquivos de prescrição a partir dos dados consolidados na camada Gold.

## Evidências

### `01_pdfs_prescricao_armazenados.png`

Este print comprova que os arquivos PDF de prescrição foram localizados e armazenados no Databricks Volume configurado para os resultados do pipeline.

A evidência demonstra:

- existência dos arquivos de prescrição em formato PDF;
- organização dos artefatos no armazenamento em nuvem;
- utilização do diretório de prescrições pendentes para revisão;
- integração entre o processamento analítico e a geração dos relatórios finais.

### `02_pdf_prescricao_gerado.png`

Este print comprova a execução bem-sucedida do processo de geração do PDF consolidado.

A evidência apresenta:

- confirmação de que o PDF único foi gerado;
- quantidade de sessões incluídas no documento;
- nome do arquivo produzido;
- caminho de armazenamento no Databricks Volume.

## Resultado do MVP

O pipeline executa o fluxo completo:

1. ingestão dos dados de anamnese e treinamento;
2. tratamento e transformação dos dados;
3. criação das tabelas dimensionais e de fatos;
4. consolidação das prescrições na tabela Gold;
5. geração automatizada do relatório em PDF;
6. armazenamento do artefato para revisão técnica.

Os arquivos PDF não são disponibilizados neste repositório. O resultado é comprovado pelos prints de execução e armazenamento, preservando a privacidade dos atletas e evitando a exposição de dados pessoais ou clínicos.

## Observação de privacidade

As evidências publicadas no repositório devem estar anonimizadas. Nomes completos, identificadores pessoais, caminhos que contenham dados identificáveis e outras informações sensíveis devem ser ocultados ou recortados antes da publicação.
