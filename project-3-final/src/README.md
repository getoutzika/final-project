# Instruções básicas para os procedimentos computacionais

## Analise de rede de vias enriquecias

Para realizar os procedimentos da análise de vias enriquecias, executar o Jupyter-notebook disponível em [aqui](../pipelines/notebooks/analise_vias_enriquecidas.ipynb).

Em seguida, foi utilizado o CytoScape para visualizar a rede gerada e assim realizar as análises biológicas. O arquivo do CytoScape para essa análise pode ser acessado [aqui](./ZIKV_POL_vs_CTRL_72hpi_pathways.cys).

## Analise de rede de interação proteína-proteína

O código Python disponivel na análise de vias enriquecidas faz o tratamento dos dados e gera uma lista com nomes dos genes diferencialmente expressos:
- [ZIKV_POL_vs_CTRL_72hpi_genes.tsv](../data/interim/ZIKV_POL_vs_CTRL_72hpi_genes.tsv)
- [ZIKV_POL_vs_CTRL_72hpi_genes-names-only.tsv](../data/interim/ZIKV_POL_vs_CTRL_72hpi_genes-names-only.tsv)

De posse de tal arquivo com os nomes dos genes diferencialmente expressos, no site String foi utilizada a funcionalidade de busca [Multiple Proteins by Names / Identifiers](https://string-db.org/cgi/input?sessionId=b83UhkDkzkcG&input_page_active_form=multiple_identifiers) para se contruir uma rede de interação proteica com as seguintes caracteristias:
- network type: full STRING network
- active interaction sources: textmining, experiments and database
- meaning of network edges: confidence
- minimum required interaction score: medium confidence (0.400)

Resultando em uma rede de interação proteica com [582 nós](../data/raw/ZIKV_POL_vs_CTRL_72hpi-string_node_degrees-582nodes_full_text_exp_db_0400.tsv) e [4389 arestas](../data/raw/ZIKV_POL_vs_CTRL_72hpi-string_interactions_short-582nodes_full_text_exp_db_0400.tsv).

No Orange, foi criado um [primeiro fluxo de trabalho](../pipelines/workflows/join-string-genes-log2foldchange.ows) para juntar ao nós da rede então criada, o valor da expressão diferencial (log2FoldChange) relativa a cada gene estudado nas amostras extraídas do GEO. E por fim foram gerados dois novos arquivos para serem importados no Neo4J.
- [ZIKV_POL_vs_CTRL_72hpi-string_nodes-log2foldchange.csv](../data/interim/ZIKV_POL_vs_CTRL_72hpi-string_nodes-log2foldchange.csv)
- [ZIKV_POL_vs_CTRL_72hpi-string_interactions.csv](../data/interim/ZIKV_POL_vs_CTRL_72hpi-string_interactions.csv)

Para a rede de genes associados à microcefalia, buscamos no website String por `microcefalia` na funcionalidade de busca [Geneset by Pathway / Process / Disease / Publication](https://string-db.org/cgi/input?sessionId=b83UhkDkzkcG&input_page_active_form=single_term) onde foram retornadas diversas posibilidades de bancos de dados. Escolhemos então o banco de dados de Human genes for microcephaly, que resultou em uma rede de interação proteica com [1080 nós](../data/raw/Microcephaly-HP_0000252-string_node_degrees-1080nodes.tsv) e 13368 arestas.

No Orange, foi criado um [segundo fluxo de trabalho](../pipelines/workflows/add-microcephaly-related-feature.ows) para filtrar os genes comuns à primeira rede criada e à segunda de forma a reduzir o tamanho da rede de genes relacionados à doença.

Em seguida buscou-se no website String uma lista reduzida de genes que resultou em uma rede de interação proteica com [27 nós](../data/raw/Microcephaly-HP_0000252-string_node_degrees-27nodes_default_config.tsv) e [41 arestas](../data/raw/Microcephaly-HP_0000252-string_node_degrees-27nodes_default_config.tsv). Por fim, novamente foi utilizado o Orange para se adicionar uma feature informativa de forma a discriminar os nós associados à microcefalia. Então foram gerados outros dois novos arquivos para serem importados no Neo4J.
- [Microcephaly_HP_0000252-string_nodes_microcephalyrelated.csv](../data/interim/Microcephaly_HP_0000252-string_nodes_microcephalyrelated.csv)
- [Microcephaly_HP_0000252-string_interactions.csv](../data/interim/Microcephaly_HP_0000252-string_interactions.csv)

De posse dos arquivos gerados, foi utilizado o Neo4J para importar as redes de interação proteica com base no modelo lógico disponível em [neo4j_importer_model_2024-06-16.json](../src/neo4j_importer_model_2024-06-16.json) resultando em uma rede com 582 nós e 4389 arestas mesclando as redes de genes diferencialmente expressos e genes associados à microcefalia.

O próximo passo foi exportar a rede para visualização no CytoScape, porém a quantidade de relacionamenos tornou inviável a exportação. Então foi necessário filtrar a rede para reduzir a quantidade de relacionamentos. Para isso, foi utilizado o seguinte código Cypher:

```
MATCH (p1)-[r]-(p2)
WHERE p2.MicrocephalyRelated IS NOT NULL
RETURN p1.Name, p1.Log2FoldChange, p1.MicrocephalyRelated, r.combined_score, p2.Name, p2.Log2FoldChange, p2.MicrocephalyRelated
```
Dessa forma fica garantido que pelo menos um dos nós da relação será associado à microcefalia. Assim, foi possível reduzir a rede para [233 nós e 667 arestas](../data/processed/neo4j-nodes(233)-relationships(667)-query_table_data.csv) para posterior visualização e analise no CytoScape.

Em seguida, foi utilizado o CytoScape para visualizar a rede de interação proteica final gerada e assim realizar as análises biológicas. O arquivo do CytoScape para essa análise pode ser acessado [aqui](./PPI-ZIKV_POL_vs_CTRL_72hpi-Microcephaly.cys).