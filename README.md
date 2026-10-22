# Análise de viabilidade — Monitoramento de vegetação em rodovias

## 1. Capa / Identificação da proposta

| Campo | Informação |
| --- | --- |
| Nome do projeto | Monitoramento de vegetação em rodovias: apoio à priorização de inspeção e roçada |
| Organização | Motiva/CCR |
| Equipe responsável | Felipe Souza Carvalho (RM 564779); Gustavo Hackime Costa (RM 563751); Luiz Henrique Macedo Graça (RM 564704); Riquelme Santos da Mata (RM 565053) |
| Responsável pela análise | Felipe Souza Carvalho |
| Data | 22/10/2026 |
| Versão | 1.1 |

**Escopo.** Este documento consolida apenas as Sprints 1, 2 e 3 de Data Science and Analytics (classificação de imagens sintéticas e proxy tabular com dados meteorológicos). Não trata do projeto desenvolvido no challenge.

## 2. Resumo executivo

### 2.1 Problema

A Motiva/CCR precisa decidir quando e onde roçar as margens das rodovias para preservar visibilidade, sinalização e faixas de rolamento. Hoje essa decisão depende de inspeções manuais, o que limita a escala e a precisão (Sprint 1). Custo, tempo e taxa de erro desse processo ainda não foram medidos.

### 2.2 Proposta de solução

Uma ferramenta de **classificação em três níveis (baixo, médio, alto)** para priorizar trechos de inspeção e roçada, sempre com decisão final humana. O projeto explorou duas frentes:

- **Imagens (Sprints 1–2):** classificação da densidade de vegetação com KNN, Random Forest e CNN, em 120 imagens sintéticas, com execução em Edge AI.
- **Dados tabulares (Sprint 3):** índice de favorabilidade ao crescimento calculado com dados meteorológicos da NASA POWER, por segmento e mês. É um *proxy* para priorizar inspeções, não uma medição de vegetação.

Esta análise recomenda tratar a frente tabular como caminho principal da prova de conceito e manter a frente de imagens condicionada à obtenção de imagens reais. As duas frentes não são equivalentes e não devem ser combinadas antes de comparadas com observações de campo.

### 2.3 Objetivo

Verificar se variáveis meteorológicas e, se disponíveis, imagens de satélite ou de campo conseguem antecipar os trechos que precisarão de inspeção ou roçada com desempenho superior ao processo atual, de forma explicável e com custo operacional aceitável.

### 2.4 Conclusão da análise de viabilidade

Há viabilidade técnica para investigar: os dados meteorológicos são públicos, o volume é pequeno e o processamento roda em computador comum. Não há, porém, evidência de desempenho em campo. As métricas de 100% das Sprints 1–2 vêm de imagens geradas por regras simples, e as classes da Sprint 3 vêm de um score definido pela equipe, sem medição independente de vegetação. Viabilidade operacional e econômica também não foram avaliadas.

**Situação recomendada para a proposta:**

- [] Viável
- [x] Viável com condições
- [] Necessita de investigação adicional
- [] Inviável nas condições atuais

**Principais condicionantes:**

- Obter dados de referência independentes (histórico autorizado de inspeção/roçada, avaliação de campo ou NDVI) para validar qualquer modelo.
- Confirmar a procedência do dataset da Sprint 3 e auditar a série meteorológica bruta (seção 4.3).
- Definir com a operação a tarefa, as métricas, o volume aceitável de alertas e os critérios de aprovação antes de qualquer piloto.
- Manter revisão humana em toda decisão de roçada; nenhuma automação de decisão nesta fase.

## 3. Descrição do problema

### 3.1 Contexto

O estudo usa como referência um trecho de rodovia concedida em SP (km 23–165, cerca de 142 km), representado por 12 segmentos georreferenciados entre Perus e Leme. Vegetação alta ou densa na faixa de domínio reduz a visibilidade dos motoristas, dificulta a sinalização e pode invadir faixas de rolamento (Sprint 2). A Sprint 1 propôs câmeras com inferência na borda; a Sprint 3 mudou para uma abordagem tabular, por falta de imagens reais.

### 3.2 Problema a ser resolvido

Indicar, com antecedência e por trecho, onde a vegetação tende a exigir inspeção ou roçada, para que a equipe de manutenção priorize deslocamentos e reduza o risco de vegetação fora de controle.

### 3.3 Impacto atual do problema

As entregas não medem o impacto atual. Sem esses números não é possível estimar retorno financeiro nem redução de risco. Indicadores a levantar com a concessionária:

| Indicador | Para que serve | Fonte provável |
| --- | --- | --- |
| Horas e custo de inspeção e roçada por segmento/ano | Linha de base de custo | Manutenção / financeiro |
| Intervalo real entre roçadas e ordens corretivas | Baseline de calendário e rótulo | Ordens de serviço |
| Ocorrências ligadas a vegetação (visibilidade, sinalização, invasão de pista) | Tamanho do risco | Segurança viária / registros operacionais |
| Trechos inspecionados sem necessidade de intervenção | Taxa de deslocamento improdutivo | Relatórios de inspeção |

### 3.4 Usuários e partes interessadas

| Stakeholder | Interesse | Impacto esperado |
| --- | --- | --- |
| Operação e manutenção rodoviária | Planejar inspeções e priorizar trechos | Menos deslocamentos improdutivos, se confirmado em piloto |
| Equipes de campo | Receber indicações verificáveis | Melhor planejamento; a ferramenta não substitui a avaliação local |
| Segurança viária | Reduzir exposição a vegetação que prejudique visibilidade ou sinalização | Apoio à priorização; efeito ainda não medido |
| Tecnologia e dados | Integrar, monitorar e manter dados e modelos | Nova rotina de dados, validação e suporte |
| Jurídico, privacidade e segurança da informação | Autorizar fontes, acessos e uso de dados | Controles proporcionais a coordenadas e sistemas envolvidos |

## 4. Viabilidade dos dados

### 4.1 Dados necessários

| Dado | Fonte | Volume | Frequência | Qualidade conhecida | Disponibilidade |
| --- | --- | --- | --- | --- | --- |
| Precipitação, temperatura, umidade e radiação | NASA POWER API | 12 segmentos × 24 meses = 288 observações mensais (série diária 2023–2024) | Diária, agregada por mês | Grade de ~0,5°; tratamento de ausentes (-999) sem relatório de completude | Sim (API pública); procedência dos 288 registros a confirmar |
| Segmento, km, coordenadas, mês | Pontos definidos pela equipe | 12 pontos, km 23–165 | Estático | Coordenadas aproximadas dos municípios, não do eixo da rodovia | Sim; precisão e autorização a validar |
| Score e classe de risco | Derivado das variáveis meteorológicas | 288 registros, classes em tercis | Mensal | Pesos definidos pela equipe, sem validação por especialista | Sim, mas não é rótulo real de vegetação |
| Histórico de inspeção, roçada e ocorrências | Concessionária | Não informado | A definir | Desconhecida | **Pendente de autorização** |
| NDVI (referência auxiliar) | Sentinel-2 / Google Earth Engine | Por segmento e data | ~5 dias, sujeito a nuvens | Mede verde/biomassa, não altura nem necessidade de roçada | Dados abertos; não utilizado ainda |
| Imagens reais das margens | Captura autorizada em campo | Não existe | A definir | — | Não disponível |

### 4.2 Disponibilidade

Os dados meteorológicos e de satélite são abertos e acessíveis sem custo, o que viabiliza a fase exploratória. A parte decisiva, os dados operacionais da concessionária, depende de autorização e ainda não foi solicitada. Termos de uso das fontes devem ser verificados antes de qualquer uso comercial.

### 4.3 Qualidade

- **Procedência do dataset da Sprint 3.** O repositório da Sprint 3 não versiona os CSV (`dados_brutos_rodovia.csv` e `dataset_final.csv` constam no `.gitignore`), e seu README descreve o `dataset_final.csv` como demonstração do pipeline, que só passa a conter valores reais da API ao reexecutar a coleta com internet. Um slide menciona "período simulado". É preciso confirmar se os 288 registros vêm da API e arquivar o CSV bruto com data de coleta.
- **Completude não auditada.** O script descarta dias com menos de 4 das 6 variáveis e interpola linearmente o restante, mas a proporção descartada e interpolada não está documentada.
- **Resolução espacial.** Pontos adjacentes distam de 7 a 23 km, bem menos que a célula da grade (~55 km); vários segmentos tendem a compartilhar os mesmos valores, o que enfraquece a comparação entre trechos.
- **Rótulo circular.** O score é função das próprias variáveis de entrada; a correlação entre elas e o score na análise exploratória é consequência da fórmula, não evidência de efeito sobre a vegetação. Os tercis forçam classes balanceadas e a normalização min-max usa todo o conjunto, o que causaria vazamento em uma divisão treino/teste.
- **Imagens sintéticas.** As 120 imagens (40 por classe, 224 × 224 px) são círculos verdes sobre fundo fixo, com classes definidas por 10, 50 ou 120 manchas. São separáveis por regra trivial e não incluem iluminação, sombra, estação, perspectiva nem espécies.

### 4.4 Rotulagem

Nenhum dado atual tem rótulo independente. Opções para a prova de conceito:

| Fonte de rótulo | Vantagem | Limitação |
| --- | --- | --- |
| Ordens de serviço e histórico de roçada | Reflete a decisão real do negócio; custo baixo | Depende de autorização e da qualidade do registro; pode refletir calendário, não necessidade |
| Avaliação de campo por especialista | Padrão de referência | Custo e prazo altos; exige protocolo e deslocamento |
| NDVI Sentinel-2 | Barato e escalável | Mede verde, não necessidade de roçada; falhas por nuvens |

- **Quantidade estimada de exemplos rotulados:** até 288 pares segmento-mês, alinhados aos dados existentes. É referência de alinhamento, não garantia de suficiência estatística; o tamanho final e a cobertura sazonal devem ser definidos por análise de poder.
- **Tempo estimado:** horas de especialista = nº de avaliações × tempo por avaliação. Como hipótese a validar, 288 avaliações remotas de 15 min correspondem a cerca de 72 h. Avaliações em campo dependem de deslocamento e não foram estimadas.
- **Custo estimado:** a cotar com a Motiva/CCR; proporcional às horas de especialista e aos deslocamentos. Se o histórico de ordens de serviço for suficiente, o custo de rotulagem tende a ser pequeno.

### 4.5 Dados sensíveis

Variáveis meteorológicas e imagens de satélite públicas não são dados pessoais. Coordenadas de ativos, imagens de campo (que podem captar pessoas ou veículos) e registros operacionais podem exigir controle de acesso, anonimização de imagens e avaliação jurídica. Medidas: minimização, acesso por função, armazenamento controlado e retenção limitada.

## 5. Modelagem

### 5.1 Modelo ou abordagem

Tarefa: classificação supervisionada ordinal (baixo, médio, alto) ou binária (exige/não exige intervenção no mês seguinte), por segmento-mês. O alvo exato deve ser definido com a operação.

- **Score atual:** mantido como indicador descritivo e baseline transparente.
- **Regressão logística e Random Forest** sobre as variáveis meteorológicas, treinados com rótulos independentes. São adequados a poucos dados, rodam em CPU e permitem explicar o resultado à equipe.
- **CNN / KNN:** só voltam a ser considerados com imagens reais em volume e diversidade suficientes. Na Sprint 2 o Random Forest foi preferido por empatar com a CNN sendo mais simples.
- **Edge AI:** adiada. A conversão da CNN para TFLite e a execução em Raspberry Pi ou módulos IoT foram propostas, mas nunca medidas.

### 5.2 Baseline

Três referências, em ordem crescente de sofisticação:

1. **Processo atual** (calendário fixo de roçada/inspeção): é o que a solução precisa superar para justificar-se.
2. **Sazonalidade** (mês ou estação do ano): mostra se a meteorologia acrescenta algo além do calendário.
3. **Score atual da Sprint 3.**

Evidência disponível (Sprint 2, 18 imagens de teste, 6 por classe, imagens sintéticas):

| Modelo | Acerto no teste | IC 95% (binomial exato) | F1 macro |
| --- | --- | --- | --- |
| KNN | 17/18 (94%) | 73% a 99,9% | 0,94 |
| Random Forest | 18/18 (100%) | 81% a 100% | 1,00 |
| CNN | 18/18 (100%) | 81% a 100% | 1,00 |

Com 18 exemplos, um erro equivale a 5,6 pontos percentuais e o limite inferior do intervalo de confiança do "100%" é 81%. Os resultados não distinguem os modelos e descrevem apenas o conjunto artificial. Na Sprint 3 não há classificador treinado e validado contra observações de vegetação.

### 5.3 Requisitos computacionais estimados

Para 288 registros tabulares, bastam 4 núcleos de CPU e 8 GB de RAM como referência inicial, sem GPU; confirmar com o dataset real. Treino de CNN com imagens reais, inferência embarcada e latência em dispositivo de borda não foram medidos e exigem benchmark próprio.

### 5.4 Desempenho esperado

Valores iniciais, **a aprovar com a operação** (não são resultados já atingidos), medidos em dados independentes com rótulos reais:

| Métrica | Mínimo aceitável |
| --- | --- |
| Recall da classe de maior risco | ≥ 0,90 |
| F1 macro | ≥ 0,75 |
| Ganho sobre o melhor baseline | ≥ 5 p.p. em F1 macro, com IC 95% (bootstrap por segmento) que exclua ganho zero |
| Volume de alertas | Executável pela equipe de campo, definido com a operação |
| Latência e disponibilidade | A definir após escolher o ambiente de execução; ainda sem requisito que justifique valor |

### 5.5 Justificativa

O pior erro é o falso negativo da classe de maior risco (trecho crítico não sinalizado), por isso o recall dessa classe é a métrica principal. O F1 macro evita que uma classe domine a avaliação, e a comparação com baselines mede se a IA acrescenta valor sobre o processo atual. O volume de alertas controla o falso positivo, que gera deslocamento desnecessário.

### 5.6 Critério de sucesso

A prova de conceito é bem-sucedida se, cumulativamente:

- a procedência e a qualidade da série meteorológica forem confirmadas em auditoria;
- houver rótulos independentes para os 12 segmentos cobrindo ao menos dois ciclos sazonais;
- o melhor modelo atingir as métricas da seção 5.4 em divisão temporal e por segmento, evitando que observações correlacionadas apareçam em treino e teste;
- a operação confirmar que o volume de alertas é executável e que a saída é compreensível.

## 6. Análise de riscos

Nível = combinação qualitativa de probabilidade e impacto.

| Categoria | Risco | Prob. | Impacto | Nível | Mitigação |
| --- | --- | --- | --- | --- | --- |
| Técnico | Proxy meteorológico não representar crescimento ou necessidade real de roçada | Alta | Alto | Alto | Validar contra histórico e campo; comunicar a saída como prioridade de inspeção, não diagnóstico |
| Dados | Procedência, completude ou resolução espacial inadequadas | Alta | Alto | Alto | Reexecutar a coleta, arquivar CSV bruto, auditar ausentes e comparar com NDVI ou medições locais |
| Dados | Falta de autorização ou baixa qualidade dos dados operacionais | Média | Alto | Alto | Acordo formal com finalidade, responsável, acesso e retenção antes da coleta |
| Técnico | Desempenho aparente alto por dados sintéticos ou divisão inadequada | Alta | Alto | Alto | Descartar métricas sintéticas para decisão; separação temporal/por segmento; avaliação independente |
| Ético | Viés do score (pesos sem especialista, tercis forçados) | Alta | Médio | Alto | Revisar pesos com agrônomo; avaliar classes, segmentos e estações separadamente |
| Econômico | Custos e retorno não estimados | Alta | Médio | Alto | Medir custos do processo atual e da coleta; análise econômica antes do piloto |
| Operacional | Excesso de alertas ou baixa adoção | Média | Médio | Médio | Co-desenhar o fluxo com a operação; piloto supervisionado medindo carga e utilidade |
| Operacional | Integração, conectividade e manutenção de dispositivos de borda | Média | Médio | Médio | Adiar Edge AI; estudo técnico de infraestrutura, segurança e custo total |
| Segurança / jurídico | Exposição de coordenadas, imagens ou acesso indevido aos sistemas; desrespeito a termos de uso | Baixa | Alto | Médio | Minimização, acesso por função, revisão de segurança e checagem de licenças |
| Segurança viária | Confiança excessiva na ferramenta e decisão sem validação humana | Baixa | Alto | Médio | Manter humano na decisão; registrar falsos negativos; comunicar limites da ferramenta |

## 7. Conclusão da análise

### 7.1 Síntese dos resultados

| Sprint | Entrega | O que demonstra | O que não demonstra |
| --- | --- | --- | --- |
| 1 | Definição do problema, dataset sintético, EDA, pré-processamento | Estrutura do problema e pipeline reprodutível | Qualquer característica de imagens reais |
| 2 | KNN, Random Forest e CNN em 120 imagens sintéticas | Fluxo de treino e avaliação funciona; modelos leves resolvem o problema artificial | Generalização, latência em borda ou viabilidade da câmera |
| 3 | Pipeline NASA POWER e score de favorabilidade (288 registros) | Coleta e agregação meteorológica são factíveis e baratas | Relação com vegetação real; o score é proxy não validado |

O projeto é tecnicamente investigável a baixo custo, mas as evidências atuais não sustentam piloto, implantação nem promessa de redução de custo ou risco.

### 7.2 Decisão sobre a próxima etapa

**Recomendação de encaminhamento:**

- [] Encerrar a proposta
- [] Realizar investigação adicional
- [x] **Desenvolver prova de conceito** (em duas fases, com ponto de decisão entre elas)
- [] Executar projeto-piloto
- [] Prosseguir para implementação

**Justificativa:** o custo de uma prova de conceito tabular é baixo e as fontes de dados existem, mas ela só gera conhecimento se começar pela **Fase 0**, que fecha as incertezas de dados (procedência, auditoria, acesso a rótulos). Se a Fase 0 não for concluída (sem acesso a dados operacionais ou de referência, ou série meteorológica não confirmada), a proposta passa a **inviável nas condições atuais** e deve ser reavaliada ou encerrada.

### 7.3 Condições para implementação

- Acordo com a Motiva/CCR para uso de histórico de inspeção, roçada e ocorrências, com identificadores espaciais e temporais.
- Protocolo de rotulagem validado por especialista e definição do alvo com a operação.
- Série meteorológica reproduzível e auditada.
- Métricas, baseline, limite de alertas e critérios de aprovação formalizados com os usuários.
- Revisão humana obrigatória das recomendações; nenhuma decisão de roçada automatizada.

### 7.4 Próximos passos

| Fase | Ação | Resultado esperado |
| --- | --- | --- |
| 0 | Reexecutar `collect_data.py` e `process_dataset.py` com internet; arquivar CSV bruto e registrar completude, ausentes e interpolações | Dataset confirmado e auditável |
| 0 | Solicitar à operação histórico de inspeção/roçada/ocorrências e levantar os indicadores da seção 3.3 | Baseline operacional e candidatos a rótulo |
| 0 | Definir alvo, protocolo de rotulagem e métricas com operação e especialista; revisar os pesos do score | Critérios de aprovação acordados |
| 0 | Testar NDVI (Sentinel-2) como referência auxiliar para validar o proxy | Primeira checagem independente do score |
| 1 | Treinar baselines e modelos simples com divisão temporal e por segmento; reportar erros por classe, segmento e estação | Decisão go/no-go para piloto |
| 1 | Se aprovado, estimar custo total, integração, segurança e suporte antes de propor piloto | Proposta de piloto fundamentada |
