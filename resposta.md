# 📝 Resposta do Laboratório: A Wiki Perdida dos Arquivos Corporativos

> Preencha este arquivo com a sua proposta de solução.
>
> Sua resposta deve explicar como transformar os documentos brutos da pasta `raw/` em uma Wiki Corporativa Inteligente, pesquisável e segura usando apenas serviços da AWS.

---

## 👤 Identificação

**Nome:**  
Fredy Salazar

**Data:**  
10/10/2026

**Link do repositório:**  
(https://github.com/fredsalas87/laboratorio-wiki-aws)

---

# ✅ Quest 1: O Mapa dos Arquivos Perdidos

## 1.1 Formatos encontrados na pasta `raw/`

Descreva quais tipos de arquivos existem dentro da pasta `raw/`.

```md
Exemplo de como responder, com o formato e o que ele implica:
- <extensao>: <nasce digital ou precisa de OCR?>, <o que da para extrair>
```

> Abra a pasta e liste o que voce encontrou de fato. Esta quest avalia a sua
> leitura do acervo, entao a resposta certa e a que corresponde aos arquivos.

**Sua resposta:**

```md
- `.pdf` (`ata_reuniao_vendas_sa.pdf`): Nasce digital (com camada de texto). É possível extrair o texto corrido diretamente sem gastar com OCR.
- `.png` (`ata_resultados_vendas_novos_dados.png`): Imagem digitalizada (apenas pixels). Precisa obrigatoriamente de OCR para extrair as letras da imagem.
- `.csv` (`vendas_sa_dados_ficticios_laboratorio.csv`): Dados estruturados em tabela (CRM). É possível extrair colunas e linhas para transformar em textos explicativos ou JSON.
```

---

## 1.2 Principais desafios encontrados

Explique quais dificuldades esses documentos podem apresentar.

```md
Exemplo:
- Arquivos sem padrão de nomenclatura
- Documentos escaneados com baixa qualidade
- Textos manuscritos ou parcialmente ilegíveis
- Atas com estruturas diferentes
- Informações importantes espalhadas em vários formatos
```

**Sua resposta:**

```md
- Mistura de formatos diferentes dentro da mesma pasta sem organização prévia.
- Arquivos de imagem (`.png`) que dependem de leitura de caracteres por computador e podem conter falhas de nitidez.
- Arquivos de tabela (`.csv`) que não possuem texto corrido e precisam ser convertidos em frases para a IA entender.
- Documentos longos em PDF com quebras de páginas, tópicos e listas de participantes espalhados.
```

---

## 1.3 Informações importantes a serem extraídas

Liste quais informações precisam ser identificadas para transformar os documentos em conhecimento pesquisável.

**Sua resposta:**

```md
- Datas das reuniões e eventos.
- Lista de participantes e áreas envolvidas.
- Temas e tópicos principais discutidos.
- Decisões tomadas e aprovadas.
- Pessoas responsáveis por cada tarefa.
- Prazos, pendências, riscos e próximos passos.
- Valores e status de oportunidades do CRM (no caso do CSV).
```

---

## 1.4 Estratégia de classificação inicial

Como você classificaria os documentos sem depender de subpastas dentro de `raw/`?

**Sua resposta:**

```md
A classificação será feita de forma automática na entrada usando **Tags do Amazon S3** e metadados lidos pela extensão do arquivo via **AWS Lambda**:
- Arquivos `.pdf` recebem a Tag `Tipo=Documento-PDF`.
- Arquivos `.png` recebem a Tag `Tipo=Imagem-OCR`.
- Arquivos `.csv` recebem a Tag `Tipo=Tabela-CRM`.
```

---

# ✅ Quest 2: O Portal de Entrada na AWS

## 2.1 Armazenamento dos arquivos brutos

Explique como os arquivos da pasta `raw/` seriam enviados e armazenados na AWS.

Serviços que você pode considerar:

- Amazon S3
- AWS IAM
- AWS KMS
- Amazon S3 Versioning
- Amazon S3 Lifecycle

**Sua resposta:**

```md
Os arquivos serão enviados diretamente para um bucket do **Amazon S3** chamado `s3://wiki-corporativa-dados/raw/`. O acesso a esse bucket será protegido por políticas do **AWS IAM** para que apenas pessoas autorizadas façam o upload, e todos os dados serão criptografados em repouso usando o **AWS KMS**.
```

---

## 2.2 Preservação dos arquivos originais

Explique como garantir que os arquivos originais sejam mantidos intactos e rastreáveis.

**Sua resposta:**

```md
A pasta `raw/` no Amazon S3 será mantida como uma área somente de leitura para processamentos. Ativaremos o **Amazon S3 Versioning** (Versioning de objetos) para evitar que arquivos sejam sobrescritos ou apagados acidentalmente. Qualquer arquivo alterado ou limpo será salvo em outra pasta separada (`s3://wiki-corporativa-dados/processed/`).
```

---

## 2.3 Extração de texto dos documentos

Explique como cada tipo de arquivo seria processado.

Considere:

- PDFs escaneados;
- Imagens;
- PDFs digitais;
- Arquivos `.txt`;
- Arquivos `.docx`;
- Arquivos `.md`.

Serviços que você pode considerar:

- Amazon Textract
- AWS Lambda
- AWS Step Functions
- Amazon S3
- Amazon CloudWatch

**Sua resposta:**

```md
Assim que um arquivo chega no S3, um evento dispara uma função **AWS Lambda** operada em um fluxo do **AWS Step Functions**:
- **PDFs e Imagens escaneadas (`.png`)**: O fluxo envia o arquivo para o **Amazon Textract**, que usa visão computacional para ler a imagem e retornar o texto digitado.
- **PDFs digitais (`.pdf`)**: A função Lambda usa bibliotecas Python simples para ler a camada de texto diretamente do PDF, sem precisar acionar o Textract, reduzindo custos.
- **Arquivos `.csv`**: O Lambda lê as linhas da tabela e as formata em textos curtos estruturados (ex: "A oportunidade X do cliente Y está com o status Z").
- **Arquivos `.txt`, `.docx`, `.md`**: O Lambda faz a leitura nativa do texto diretamente.
```

---

## 2.4 Tratamento de falhas

Explique como sua solução identificaria e registraria erros de processamento.

**Sua resposta:**

```md
Todas as execuções do Lambda e Step Functions serão monitoradas pelo **Amazon CloudWatch**. Caso um arquivo esteja corrompido ou o OCR falhe, a mensagem de erro será gravada nos logs do CloudWatch e o arquivo com falha será enviado para uma fila de refazer/análise chamada **Amazon SQS DLQ (Dead-Letter Queue)**.
```

---

# ✅ Quest 3: A Relíquia dos Metadados

## 3.1 Padronização dos textos processados

Explique como os textos extraídos seriam limpos, normalizados e preparados para consulta.

**Sua resposta:**

```md
Uma função **AWS Lambda** receberá o texto bruto extraído e executará um script de limpeza para:
- Remover quebras de linha desnecessárias e espaços duplos.
- Corrigir caracteres estranhos gerados na leitura de imagens.
- Salvar o resultado final padronizado no formato `.txt` dentro de `s3://wiki-corporativa-dados/processed/`.
```

---

## 3.2 Metadados propostos

Defina quais metadados você extrairia de cada documento.

| Metadado | Por que ele é importante? |
|---|---|
| Nome do documento | Preencha aqui |
| Tipo do documento | Preencha aqui |
| Data identificada | Preencha aqui |
| Tema principal | Preencha aqui |
| Participantes | Preencha aqui |
| Decisões tomadas | Preencha aqui |
| Responsáveis | Preencha aqui |
| Próximos passos | Preencha aqui |
| Nível de confidencialidade | Preencha aqui |
| Caminho do arquivo original | Preencha aqui |

Adicione outros metadados, se necessário.

---

## 3.3 Uso de IA para enriquecimento dos documentos

Explique como o Amazon Bedrock poderia ajudar a identificar temas, decisões, responsáveis, pendências e resumos dos documentos.

**Sua resposta:**

```md
O **Amazon Bedrock** receberá o texto limpo e, através de um prompt instruído, um modelo de linguagem extrairá automaticamente um resumo executivo, identificará os nomes dos responsáveis, datas, temas principais e as decisões tomadas. Essa resposta da IA é estruturada em formato JSON pronto para indexação.
```

---

## 3.4 Armazenamento dos metadados

Explique onde os metadados seriam armazenados e como seriam conectados aos documentos originais.

Serviços que você pode considerar:

- Amazon S3
- Amazon DynamoDB
- AWS Glue Data Catalog
- Amazon Bedrock Knowledge Bases

**Sua resposta:**

```md
Os metadados extraídos em JSON serão armazenados no **Amazon DynamoDB**, um banco de dados rápido e chave-valor. A chave primária da tabela será o próprio caminho do arquivo no S3 (`s3://wiki-corporativa-dados/raw/nome_do_arquivo`), criando uma conexão direta entre os metadados e o arquivo original.
```

---

# ✅ Quest 4: O Oráculo da Wiki Inteligente

## 4.1 Estratégia de indexação

Explique como os documentos seriam divididos em trechos menores e preparados para busca semântica.

**Sua resposta:**

```md
Usaremos o **Amazon Bedrock Knowledge Bases**, que divide automaticamente os arquivos limpos da pasta `processed/` em pedaços menores (*chunks*) de aproximadamente 300 a 500 palavras. Essa divisão garante que as buscas encontrem o parágrafo exato onde uma dúvida é respondida.
```

---

## 4.2 Busca semântica e base vetorial

Explique como embeddings seriam gerados e onde seriam armazenados.

Serviços que você pode considerar:

- Amazon Bedrock Knowledge Bases
- Amazon OpenSearch Serverless
- Amazon Aurora PostgreSQL com pgvector
- Amazon S3 Vectors
- Modelos de embeddings no Amazon Bedrock

**Sua resposta:**

```md
O **Amazon Bedrock Knowledge Bases** utiliza o modelo **Amazon Titan Text Embeddings** para converter cada trecho de texto em um vetor numérico (vetor que representa o significado das palavras). Esses vetores são armazenados automaticamente em uma base de dados vetorial usando o **Amazon OpenSearch Serverless**.
```

---

## 4.3 Geração de respostas com IA

Explique como a Wiki responderia perguntas em linguagem natural com base nos documentos originais.

Considere explicar:

- Como a pergunta do usuário seria recebida;
- Como os trechos relevantes seriam recuperados;
- Como o Amazon Bedrock geraria a resposta;
- Como a resposta indicaria as fontes utilizadas.

**Sua resposta:**

```md
1. O usuário digita uma pergunta (ex: *"Quais os próximos passos do projeto de vendas?"*).
2. O **Amazon Bedrock Knowledge Bases** busca no OpenSearch Serverless os trechos de texto mais parecidos semanticamente com a pergunta.
3. Os trechos encontrados e a pergunta original são enviados para o modelo de linguagem no **Amazon Bedrock**.
4. O Bedrock lê apenas aqueles documentos recuperados e gera uma resposta clara em português.
5. A resposta final é exibida ao usuário acompanhada do link/nome do documento original no S3 utilizado como fonte.
```

---

## 4.4 Interface de consulta

Proponha como os usuários acessariam essa Wiki Inteligente.

Serviços que você pode considerar:

- Amazon Q Business
- AWS Amplify
- Amazon API Gateway
- AWS Lambda
- Amazon Cognito

**Sua resposta:**

```md
Para simplificar sem precisar codificar uma aplicação do zero, podemos disponibilizar a consulta através do **Amazon Q Business**, criando um portal de chat corporativo onde os funcionários fazem perguntas diretamente no navegador e o sistema responde usando a base do Bedrock.
```

---

## 4.5 Segurança, auditoria e monitoramento

Explique como controlar acesso, proteger dados, auditar consultas e monitorar custos, erros e qualidade das respostas.

Serviços que você pode considerar:

- AWS IAM
- AWS KMS
- Amazon Cognito
- AWS CloudTrail
- Amazon CloudWatch
- Amazon Macie
- AWS Cost Explorer

**Sua resposta:**

```md
- **Controle de Acesso e Autenticação**: **Amazon Cognito** ou integração do **Amazon Q Business** com o login corporativo da empresa via **AWS IAM**.
- **Proteção dos Dados**: Criptografia de dados em repouso e em trânsito com chaves gerenciadas no **AWS KMS**.
- **Auditoria**: O **AWS CloudTrail** registrará todas as ações, chamadas de API e consultas feitas.
- **Monitoramento e Custos**: **Amazon CloudWatch** para alertas de erros e **AWS Cost Explorer** para controlar os gastos com chamadas de IA.
```

---

# 🧩 Arquitetura Final da Solução

Agora reúna tudo em uma visão única.

## 1. Visão geral

Explique em poucas linhas a ideia central da sua arquitetura.

**Sua resposta:**

```md
A solução consiste em uma esteira *serverless* totalmente gerenciada pela AWS. Os documentos brutos enviados ao Amazon S3 são processados pelo Amazon Textract ou Lambda, limpos e enriquecidos com metadados via Amazon Bedrock. O conteúdo é vetorizado pelo Bedrock Knowledge Bases e disponibilizado em uma interface inteligente no Amazon Q Business com segurança total.
```

---

## 2. Serviços AWS utilizados

| Serviço AWS | Papel na solução |
|---|---|
| Amazon S3 | Preencha aqui |
| Amazon Textract | Preencha aqui |
| Amazon Bedrock | Preencha aqui |
| Amazon Bedrock Knowledge Bases | Preencha aqui |
| AWS Lambda | Preencha aqui |
| AWS Step Functions | Preencha aqui |
| Amazon CloudWatch | Preencha aqui |
| AWS IAM | Preencha aqui |
| AWS KMS | Preencha aqui |

Adicione, remova ou ajuste os serviços conforme sua proposta.

---

## 3. Fluxo de dados de ponta a ponta

Descreva o caminho dos dados desde a pasta `raw/` até a Wiki Inteligente.

```md
Exemplo de estrutura:

1. Arquivos estão inicialmente na pasta raw/
2. Arquivos são enviados para o Amazon S3
3. Documentos escaneados passam pelo Amazon Textract
4. Arquivos digitais têm seus textos extraídos
5. Textos são limpos e padronizados
6. Metadados são extraídos
7. Conteúdos são indexados em uma base pesquisável
8. Usuário pesquisa na Wiki
9. IA responde com base nos documentos originais
```

**Sua resposta:**

```md
Preencha aqui.1. Os arquivos brutos chegam na pasta `raw/` do Amazon S3.
2. Um evento do S3 aciona o AWS Step Functions que executa o AWS Lambda.
3. Se for imagem/PDF escaneado, o arquivo é enviado ao Amazon Textract. Se for digital ou CSV, o Lambda faz o processamento.
4. O texto resultante é limpo e salvo em `s3://.../processed/`.
5. O Amazon Bedrock analisa o texto e gera metadados em JSON, que são salvos no Amazon DynamoDB.
6. O Amazon Bedrock Knowledge Bases lê o texto limpo, divide em partes, gera vetores e salva no Amazon OpenSearch Serverless.
7. O usuário faz uma pergunta no chat do Amazon Q Business.
8. A IA consulta a base de vetores, recupera as fontes no S3 e responde citando os documentos originais.```

---

## 4. Diagrama textual da arquitetura

Crie um diagrama simples usando texto.

```md
Exemplo:

raw/ → Amazon S3 → Lambda/Step Functions → Textract → S3 Processado → Bedrock Knowledge Bases → Interface de Consulta → Usuário Final
```

**Sua resposta:**

```md
`raw/ (Amazon S3)` ➡️ `AWS Lambda / Step Functions` ➡️ `Amazon Textract (OCR)` ➡️ `S3 Processado` ➡️ `Amazon Bedrock (Metadados -> DynamoDB)` ➡️ `Bedrock Knowledge Bases (OpenSearch)` ➡️ `Amazon Q Business (Chat)` ➡️ `Usuário Final`
```

---

## 5. Riscos e limitações

Liste possíveis desafios da sua solução.

```md
Exemplo:
- Documentos ilegíveis podem prejudicar a extração de texto.
- OCR pode gerar erros em documentos com baixa qualidade.
- Custos podem aumentar conforme o volume de documentos.
- Metadados inferidos por IA podem precisar de validação humana.
- Respostas geradas por IA devem sempre referenciar documentos de origem.
```

**Sua resposta:**

```md
- Imagens em baixíssima resolução podem gerar erros na leitura do Textract.
- Arquivos em tabela (CSV) muito grandes exigem regras bem definidas no Lambda para não poluir o modelo de IA.
- Custos de uso da API do Amazon Bedrock e Textract devem ser acompanhados com alertas no CloudWatch.
- Respostas geradas por IA exigem que os documentos originais estejam atualizados para evitar alucinações.
```

---

## 6. Melhorias futuras

Descreva como a solução poderia evoluir.

```md
Exemplo:
- Criar uma interface web para consulta.
- Criar um chat interno para perguntas sobre atas.
- Adicionar controle de acesso por departamento.
- Criar dashboard de decisões e pendências.
- Gerar alertas automáticos sobre ações em aberto.
- Integrar com ferramentas corporativas.
```

**Sua resposta:**

```md
- Adicionar um fluxo de aprovação humana para revisar os metadados extraídos de atas confidenciais.
- Integrar a Wiki diretamente no Slack ou Microsoft Teams da empresa.
- Criar alertas automáticos no e-mail para avisar responsáveis sobre pendências e prazos de reuniões perto do vencimento.
- Criar dashboards executivos no Amazon QuickSight mostrando os temas mais discutidos nas reuniões do mês.
```

---

# 🧠 Checklist Final

Antes de entregar, confirme se sua solução responde:

- [ ] Como transformar documentos escaneados em texto?
- [ ] Como lidar com diferentes formatos dentro da mesma pasta `raw/`?
- [ ] Como armazenar os documentos originais?
- [ ] Como preservar a rastreabilidade entre resposta e documento fonte?
- [ ] Como organizar metadados?
- [ ] Como criar busca semântica?
- [ ] Como usar Amazon Bedrock na solução?
- [ ] Como proteger documentos sensíveis?
- [ ] Como monitorar falhas?
- [ ] Como a empresa usaria essa Wiki no dia a dia?

---

# 🏁 Conclusão

Escreva uma breve conclusão defendendo sua solução como se estivesse apresentando para uma liderança técnica ou de negócio.

**Sua resposta:**

```md
A arquitetura proposta transforma dados brutos e inacessíveis em um ativo estratégico de conhecimento para a empresa. Utilizando o ecossistema Serverless da AWS e recursos gerenciados de IA como Amazon Textract e Amazon Bedrock, eliminamos a necessidade de gerenciar servidores e garantimos baixa manutenção, alta eficiência e custo proporcional ao uso. Com esta Wiki Inteligente, a empresa reduz drasticamente o tempo gasto procurando informações em arquivos antigos, garante segurança e rastreabilidade total das decisões corporativas e eleva a produtividade das equipes.
```
