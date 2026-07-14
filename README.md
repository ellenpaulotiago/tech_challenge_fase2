# Plataforma Analítica para Monitoramento da Alfabetização no Brasil
### Tech Challenge FIAP – Engenharia & Data Analytics (Fase 2)

---

## 1. Resumo Executivo
Este projeto foi desenvolvido como parte do Tech Challenge – Fase 2 da Pós-Graduação em Data Analytics da FIAP. O objetivo principal é construir uma plataforma moderna de engenharia e análise de dados capaz de consolidar, processar, monitorar e disponibilizar informações cruciais sobre os indicadores de alfabetização da educação básica brasileira.

A solução baseia-se em uma arquitetura Lakehouse moderna de dados organizada sob o paradigma da Arquitetura Medalhão (Bronze, Silver e Gold), hospedada na nuvem Amazon Web Services (AWS) e operacionalizada na plataforma Databricks. Através do processamento distribuído, governança rigorosa de dados via Unity Catalog, observabilidade contínua, práticas de otimização de custos (FinOps) e modelagem de Machine Learning, o projeto visa transformar grandes volumes de microdados públicos e dispersos em ativos de dados consistentes e valiosos para a tomada de decisões no cenário educacional do país.

---

## 2. Contexto do Problema e Desafio Educacional
A alfabetização na infância é fundamental para o desenvolvimento socioeconômico e cultural do país. O Compromisso Nacional Criança Alfabetizada visa garantir que todas as crianças brasileiras estejam alfabetizadas até o final do 2º ano do ensino fundamental. O parâmetro técnico essencial é o Indicador Criança Alfabetizada, fundamentado na Pesquisa Alfabetiza Brasil (2023), que definiu o ponto de corte crítico em 743 pontos na escala Saeb. A meta nacional é alcançar 100% de alfabetização infantil até o ano de 2030.

Embora órgãos públicos e educacionais (como o INEP) disponibilizem grandes volumes de dados de avaliações, metas municipais/estaduais e desempenho escolar, o consumo integrado e analítico dessas informações é dificultado por severos desafios práticos de engenharia de dados:

* **Dispersão crônica:** Bases públicas distribuídas em múltiplos arquivos, formatos estruturais distintos (CSV, Excel, arquivos textuais) e diferentes níveis de granularidade.
* **Ausência de padronização:** Falta de schemas comuns ou chaves de integridade entre os conjuntos de dados históricos de diferentes anos de referência.
* **Validação complexa:** Necessidade de regras de negócios educacionais e auditoria de consistência cadastral complexas (como o cruzamento de ID de Municípios, Escolas e Estados).
* **Limitação operacional:** Processos escassos de observabilidade contínua de integridade de dados e ausência de fluxos automatizados e escaláveis de ingestão incremental.

---

## 3. Objetivos e Valor Gerado

### Objetivo Geral
Desenvolver uma plataforma analítica robusta em arquitetura Lakehouse para integrar, processar, governar e disponibilizar dados públicos e históricos sobre alfabetização, apoiando a tomada de decisão por meio de indicadores confiáveis, dashboards executivos interativos e modelos analíticos preditivos.

### Objetivos Específicos
* **Pipeline Híbrido:** Implementar fluxos em lote (Batch) e tempo real (Streaming) utilizando os serviços nativos da nuvem AWS S3 e processamento Spark distribuído no Databricks.
* **Arquitetura Medalhão:** Organizar o fluxo de dados em camadas Bronze, Silver e Gold aplicando processos de refinamento progressivos.
* **Data Quality:** Garantir governança e qualidade das informações por meio de regras de validação estrutural e regras de negócio automatizadas de qualidade de dados.
* **Consumo Otimizado:** Disponibilizar dados consolidados na camada Gold otimizados para consumo direto por dashboards em Power BI e modelos de Machine Learning.
* **Observabilidade e FinOps:** Estruturar mecanismos ágeis de monitoramento (logs, métricas, dashboards de qualidade) e controle econômico-computacional para gestão de nuvem.

---

## 4. Arquitetura da Solução e Tecnologias
A solução adota a **Arquitetura Lakehouse**, unindo a flexibilidade e baixo custo de armazenamento do Data Lake com as capacidades transacionais, de performance de consulta e governança características de um Data Warehouse tradicional.

### Pilares Tecnológicos da Plataforma
* **AWS S3:** Atua como o Data Lake principal do projeto, provendo armazenamento de alta durabilidade, escalabilidade ilimitada e baixo custo. Os dados são estruturados fisicamente de forma particionada por domínio e ano.
* **Databricks (Serverless):** Plataforma de processamento central para notebooks Apache Spark (PySpark), garantindo escalabilidade automática sob demanda para grandes cargas de dados sem a necessidade de gerenciamento manual de infraestrutura de cluster.
* **Unity Catalog:** Atua como a camada centralizada de governança corporativa de dados. Utiliza o recurso de *External Volumes* e *External Locations* no Databricks, permitindo ler e escrever diretamente de e para o S3 de maneira controlada, sem expor chaves de segurança estáticas no código do projeto.
* **Structured Streaming:** Utilizado para a ingestão e transformação incremental eficiente de indicadores municipais de metas, assegurando reprocessamento dinâmico otimizado.
* **Python/Pandas:** Aplicado de forma complementar para auditorias pontuais, investigações exploratórias específicas e validações matemáticas fora do cluster distribuído Spark.

### 📌 Decisões Arquiteturais & Trade-offs
* **Batch vs. Streaming:** O processamento em lote (Batch) é aplicado para grandes volumes históricos de microdados do INEP devido ao foco em eficiência de custo. O processamento em Streaming simula a atualização incremental frequente de metas de planejamento.
* **Data Lake vs. Data Warehouse:** O armazenamento no S3 com arquivos estruturados em formato colunar Parquet compactados com Snappy garante consultas extremamente velozes a uma fração do custo de um DW tradicional.
* **Custo vs. Performance:** A infraestrutura Serverless do Databricks aliada à eliminação de clusters ociosos otimiza o faturamento computacional.

---

## 5. Arquitetura Medalhão e Processamento
O ciclo de vida dos dados educacionais é estruturado logicamente em três camadas progressivas de maturidade dentro do Lakehouse, mitigando a propagação de inconsistências:

1. **Camada Bronze (Dados Brutos):** Porta de entrada das bases oficiais originais (INEP). Preserva a fidelidade original dos dados no bucket S3, enriquecendo os registros com metadados técnicos de controle corporativo, como data de ingestão (`_ingestion_timestamp`) e arquivo de origem (`_source_file`) para fins de auditoria e versionamento.
2. **Camada Silver (Dados Tratados):** Camada de padronização estrutural, limpeza e conformidade técnica. Realiza conversão estrita de tipos, deduplicação com base em chaves naturais, tratamento de valores ausentes (imputação ou descarte) e validações robustas de regras de negócio. Registros inconsistentes são direcionados para uma área isolada de rejeição para auditoria detalhada.
3. **Camada Gold (Dados Analíticos):** Camada de apresentação de dados consolidados e agregados espacialmente (por município, estado, região) e temporalmente. As tabelas analíticas Gold servem como fonte otimizada para o Power BI e para as esteiras de Machine Learning.
![alt text](imagem-1.png)

---

## 6. Estrutura Avançada de Data Quality
O motor de qualidade do pipeline de dados é implementado de forma automatizada na transição para a camada Silver, garantindo confiabilidade analítica através de diferentes frentes de integridade:

* **Campos Obrigatórios:** Verificação e bloqueio de registros que possuam valores nulos em colunas essenciais, tais como ID do Município, Código do Estado, Ano de Referência ou Código da Escola. Registros ausentes nessas chaves fundamentais são imediatamente rejeitados.
* **Padronização de Tipos (Cast):** Garantia de que colunas numéricas, datas, dados categóricos e lógicos sejam devidamente tipados como Inteiros, Double, Date ou Booleanos, impedindo inconsistências em modelos de inteligência artificial e agregações no Power BI.
* **Tratamento de Nulos de Negócio:** Aplicação de estratégias diferenciadas como preenchimento com valor padrão (`-1` ou `'Não Informado'` para dados cadastrais secundários), descarte controlado ou direcionamento para auditoria.
* **Consistência de Regras Educacionais:** Validação de integridade entre Estados e Municípios (confrontamento cadastral), verificação de valores de notas/pontuações Saeb dentro de limites de escala reais e datas de vigência coerentes de metas pactuadas.
* **Tratamento de Registros Rejeitados (Quarentena):** Todo registro inválido é gravado separadamente com metadados detalhados de diagnóstico: data/hora da rejeição, código do erro, dataset de origem e etapa do pipeline. Isso permite à equipe de engenharia analisar sistematicamente a qualidade dos dados de origem sem interromper o processamento produtivo.

---

## 7. Monitoramento, Observabilidade e FinOps

### Monitoramento e Observabilidade
O pipeline implementa uma camada transversal de acompanhamento técnico que coleta métricas cruciais de processamento em tempo real:
* Indicadores operacionais de tempo de início e término de execuções de notebooks, volume de dados lidos e gravados por etapa, status da tarefa (sucesso/erro) e mensagens detalhadas de falha.
* Criação de dashboards operacionais consolidados no Databricks monitorando taxas de rejeição por dataset para detecção rápida de anomalias em cargas públicas recorrentes.

### FinOps e Otimização de Custos em Nuvem
Com base na união dos métodos técnicos dos arquivos oficiais e a proposta do projeto, a gestão econômico-computacional foi estruturada em 4 frentes principais:
* **Separação de Armazenamento e Computação:** Utilização do S3 como armazenamento estático de baixo custo, desligando os clusters Databricks de processamento assim que as cargas de trabalho forem concluídas.
* **Otimização de Particionamento & Compacidade:** O particionamento temporal das tabelas (por Ano de Referência) aliado à compactação colunar Snappy em Parquet evita varreduras de tabelas inteiras (*full scans*), reduzindo custos de I/O em consultas subsequentes.
* **Varredura Preventiva de Arquivos Pequenos:** Rotinas de monitoramento e otimização automatizada (como `auto-optimize`, `optimize` e `coalesce` no Spark) para mitigar o problema de fragmentação excessiva de arquivos pequenos no S3, que encarecem as operações de leitura.
* **Validações Amostrais Leves:** Execução de rotinas rápidas de amostragem de dados (como validação de cabeçalhos e limites de registros) antes de disparar processamentos pesados de microdados (como dados nacionais de alunos), evitando gastos desnecessários com processamento em arquivos corrompidos.

---

## 8. Inteligência Artificial e Aprendizado de Máquina
A camada Gold estruturada provê dados devidamente preparados e limpos (features de alta fidelidade) para modelos de Data Science e IA:

* **Modelagem Preditiva de Metas:** Implementação de modelo baseado em *Random Forest Classifier* para estimar e classificar a probabilidade de um município brasileiro não atingir a meta crítica do Indicador Criança Alfabetizada até 2030, baseando-se em variáveis históricas de desempenho educacional.
* **Clusterização Espacial de Desigualdade:** Aplicação do algoritmo de clusterização *K-Means* para segmentar municípios semelhantes, agrupando-os por vulnerabilidade socioeconômica e educacional para orientar a alocação prioritária de investimentos públicos pelo Ministério da Educação e estados.
* **Suporte MLOps e Integrações Futuras:** A arquitetura está preparada para expansão futura utilizando modelos preditivos mais complexos de séries temporais e orquestração de ciclo de vida com MLflow (MLOps).

---

## 9. Organização Estrutural do Repositório
A organização lógica e física do repositório de dados segue rigorosamente as melhores práticas internacionais de desenvolvimento e engenharia de software:

```text
tech-challenge-fiap/
├── README.md
├── requirements.txt
├── docs/
│   ├── arquitetura/         # Diagramas e topologia de rede
│   ├── referencias/         # Documentos e links técnicos de apoio
│   └── documentacao_tecnica/
├── notebooks/
│   ├── 00_setup/            # Configuração do ambiente e Unity Catalog
│   ├── 01_bronze/           # Ingestão batch e controle de metadados
│   ├── 02_silver/           # Limpeza, padronização e Data Quality
│   ├── 03_gold/             # Agregações analíticas e tabelas de negócios
│   ├── 04_monitoring/       # Logs, métricas e observabilidade contínua
│   ├── 05_machine_learning/ # Modelagem preditiva e clusterização
│   └── 06_power_bi/         # Preparação de fontes para Dashboards
├── data/                    # Volumes locais estruturados para testes rápidos
└── scripts/                 # Utilitários e validações de CI/CD
```

---

## Estrutura do Repositório
```bash
.
│
├── README.md
├── LICENSE
├── .gitignore
├── requirements.txt
├── pyproject.toml
│
├── config/
│   ├── config.json
│   ├── finops_parameters.json
│   ├── streaming_metadata.json
│
├── notebooks/
│   │
│   ├── 00_setup/
│   │   └── 00_setup_ambiente.ipynb
│   │
│   ├── 01_bronze/
│   │   ├── 01_0_bronze_planejador.ipynb
│   │   ├── 01_1_bronze_alunos.ipynb
│   │   ├── 01_2_bronze_estados.ipynb
│   │   ├── 01_3_bronze_municipios.ipynb
│   │   ├── 01_4_bronze_metas_municipios.ipynb
│   │   └── 01_5_bronze_metas_ufs.ipynb
│   │
│   ├── 02_silver/
│   │   ├── 02_0_silver_planejador.ipynb
│   │   ├── 02_1_silver_alunos.ipynb
│   │   ├── 02_2_silver_estados.ipynb
│   │   ├── 02_3_silver_municipios.ipynb
│   │   ├── 02_4_silver_metas_municipios.ipynb
│   │   └── 02_5_silver_metas_ufs.ipynb
│   │
│   ├── 03_gold/
│   │   ├── 03_0_gold_planejador.ipynb
│   │   ├── 03_gold_orquestrador.ipynb
│   │   ├── 03_1_gold_alunos.ipynb
│   │   ├── 03_2_gold_municipios.ipynb
│   │   ├── 03_3_gold_estados.ipynb
│   │   ├── 03_4_gold_indicadores.ipynb
│   │   ├── 03_5_gold_powerbi.ipynb
│   │   └── 03_6_gold_machine_learning.ipynb
│   │
│   ├── 04_streaming/
│   │   ├── 04_0_streaming_planejador.ipynb
│   │   ├── 04_1_streaming_simulador_eventos.ipynb
│   │   ├── 04_2_streaming_bronze.ipynb
│   │   ├── 04_3_streaming_silver.ipynb
│   │   └── 04_4_streaming_gold.ipynb
│   │
│   ├── 05_quality/
│   │   ├── 05_0_quality_planejador.ipynb
│   │   ├── 05_0_quality_orquestrador.ipynb
│   │   ├── 05_1_quality_bronze.ipynb
│   │   ├── 05_2_quality_silver.ipynb
│   │   ├── 05_3_quality_gold.ipynb
│   │   └── 05_4_quality_dashboard.ipynb
│   │
│   ├── 06_monitoring/
│   │   └── 06_monitoring.ipynb
│   │
│   ├── 07_finops/
│   │   └── 07_finops.ipynb
│   │
│   └── 08_ia_modelagem/
│       └── 08_ia_modelagem.ipynb
│
├── docs/
│   ├── FinOps.png
│   ├── Gold.png
│   ├── Job Pipeline.png
│   ├── Job_pipeline_execução.mp4
│   ├── Job_pipeline_streaming.mp4
│   ├── Pipeline executada completa.png
│   ├── Pipeline Streaming executando.png
│   ├── Pipeline Streaming executada.png
│   ├── Possivéis Dashboards.png
│
├── presentation/
│   ├── TechChallenge_Apresentacao_Executiva.pptx

```
---


## 10. Governança e Boas Práticas de Git
O ciclo de desenvolvimento do pipeline de dados é regido por padrões modernos de engenharia de software para garantir a reprodutibilidade e a governança das alterações de código e dados:

* **Git Flow:** Fluxo de ramificação estrito para separação clara entre código de desenvolvimento (`develop`), homologação (`release`) e produção estável (`main`).
* **Revisões por Pull Requests (PRs):** Todas as alterações e correções de bugs propostas passam obrigatoriamente por revisão técnica de pares, mitigando riscos de degradação da performance ou segurança.
* **Commits Semânticos:** Padronização rígida de descrições de commits para garantir rastreabilidade histórica completa das evoluções do ecossistema.

| Tipo de Commit | Uso / Aplicação Prática no Projeto |
| :--- | :--- |
| **feat** | Inclusão de nova funcionalidade ou nova lógica de transformação de dados no pipeline. |
| **fix** | Resolução de bug lógico, erro de tipagem ou falha de leitura em bases do INEP. |
| **refactor** | Refatoração de código de notebooks para melhor legibilidade sem alteração de funcionalidade. |
| **perf** | Melhorias direcionadas a ganho de desempenho no Spark (`coalesce`, particionamentos). |
| **docs** | Documentação do README, documentação técnica de arquitetura e arquivos de ajuda. |
| **test** | Ajustes ou novas inclusões de testes automatizados de qualidade de dados. |
| **chore** | Atividades administrativas rotineiras, manutenção de repositórios ou dependências. |
| **config** | Modificação em arquivos de configuração de ambiente, schemas JSON ou credenciais. |
| **security** | Melhorias de segurança e ajustes de acesso baseados em políticas de controle do S3. |
| **data** | Alterações de engenharia relacionadas puramente a estruturas de tabelas ou modelos de dados. |

---

## 11. Instalação e Execução do Projeto

### Pré-requisitos Operacionais
* Conta ativa na AWS com bucket Amazon S3 estruturado.
* Workspace Databricks ativo e integrado com o Unity Catalog habilitado.
* *External Location* e credenciais de storage devidamente configurados no Unity Catalog.
* Ambiente local ou cluster com Python 3.11+ e Apache Spark compatível com o runtime Databricks.

### Ordem de Execução Recomendada
1. **Setup de Ambiente:** Execute o notebook `00_setup_ambiente` para instanciar as tabelas, schemas e locais externos no Unity Catalog.
2. **Ingestão Bronze:** Dispare o notebook `01_bronze_orquestrador` para carregar as bases cruas do S3 e salvá-las na camada Bronze com metadados.
3. **Saneamento Silver:** Execute o notebook `02_silver_orquestrador` para aplicar as regras críticas de Data Quality e gerar os registros rejeitados.
4. **Consolidação Gold:** Execute o notebook `03_gold_orquestrador` para gerar as agregações espaciais analíticas otimizadas para consumo.
5. **Analytics e Modelagem:** Utilize os notebooks de `05_machine_learning` para treinar os algoritmos preditivos e `06_power_bi` para alimentar os dashboards.
6. **Auditoria e Monitoramento:** Execute o módulo `04_monitoring` para verificar as métricas consolidadas de processamento e faturamento de recursos.

---

## 12. Roadmap de Evolução Técnica
* **Curto Prazo:** Automação de pipelines de ingestão diária de novos datasets públicos. Integração de novos KPIs educacionais municipais. Aprimoramento de dashboards analíticos executivos.
* **Médio Prazo:** Implantação de pipelines de CI/CD robustos para notebooks Databricks. Incorporação de Delta Live Tables (DLT) para gerenciamento simplificado de fluxos. Automatização completa de testes de integridade.
* **Longo Prazo:** Implementação de esteira de MLOps de ponta a ponta para controle de versão de modelos de IA com MLflow. Disponibilização de dados públicos de alfabetização via APIs seguras. Integração automatizada com bases educacionais correlacionadas.

---

## 13. Lições Aprendidas e Conclusão
A construção desta plataforma demonstrou de forma contundente que a adoção da Arquitetura Lakehouse baseada na Arquitetura Medalhão simplifica a governança de dados massivos e garante a confiabilidade de análises educacionais complexas. Os recursos integrados de qualidade de dados na camada Silver e a otimização de custos com princípios FinOps e amostragem leve tornam o projeto viável e escalável para cenários reais de grandes proporções no setor público.

### Principais Referências Técnicas
* Documentação Oficial Databricks, Spark e Delta Lake.
* AWS Well-Architected Framework & Lake House Architecture.
* *Designing Data-Intensive Applications* – Martin Kleppmann.
* *Fundamentals of Data Engineering* – Joe Reis & Matt Housley.
* Bases de dados e publicações oficiais do INEP e Ministério da Educação.
