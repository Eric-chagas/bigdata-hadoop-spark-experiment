# Relatório - Experimentos com Apache Hadoop e Spark

Projeto de estudo das tecnologias de computação distribuida, Apache Hadoop e Apache Spark. Trabalho 2 desenvolvido para a disciplina **PSPD - Programação para Sistemas Paralelos e Distribuídos (Turma 02)** – **UnB/FCTE – Engenharia de Software**

- **Professor:** Prof. Fernando W. Cruz
- **Aluno:** Eric Chagas de Oliveira
- **Matrícula:** 180119508
- **Link do vídeo de apresentação**: #TODO:Adicionar Link


# 1. **Introdução**

Esse relatório apresenta os experimentos realizados com os frameworks **Apache Hadoop** e **Apache Spark**, para o trabalho 2 de Programação para sistemas paralelos e distribuídos.

O objetivo principal é explorar conceitos de Big Data, configurando um ambiente Hadoop em modo cluster e executando testes de performance e tolerância a falhas, além de desenvolver um fluxo de processamento de dados em tempo real utilizando Spark integrado a um canal de entrada baseado em fluxos de texto e apresentação gráfica.

O documento descreve a arquitetura adotada, a metodologia utilizada, os resultados obtidos e as conclusões do grupo sobre o funcionamento e comportamento dos frameworks.

---

# 2. **Experimento com Apache Hadoop**

## 2.1 Arquitetura e Configuração do Cluster

Foi criado um cluster Hadoop composto por:

* **1 master node: NameNode + ResourceManager**
* **2 worker nodes: DataNodes + NodeManagers**
* Por conta do grupo possuir apenas um integrante (Eric Chagas) a Infraestrutura foi implementada via **Docker Compose** usando a imagem docker disponibilizada em [big-data-europe/docker-hadoop](https://github.com/big-data-europe/docker-hadoop).

### Comandos utilizados:

```bash
git clone https://github.com/big-data-europe/docker-hadoop.git
cd docker-hadoop
docker compose up -d
```

### Interfaces acessadas:

* **NameNode:** [http://localhost:9870](http://localhost:9870)
* **DataNode:** [http://localhost:9864](http://localhost:9864)
* **YARN Resource Manager:** [http://localhost:8088](http://localhost:8088)

*(Inserir prints de cada dashboard acima)*

### Arquivos de configuração utilizados

Abaixo estão os arquivos de configuração extraídos do ambiente:

* `core-site.xml`
* `hdfs-site.xml`
* `yarn-site.xml`
* `mapred-site.xml`

*(Inserir conteúdo ou trechos relevantes em anexo)*



## 2.2 Testes de Comportamento do Hadoop

Foi utilizado o job clássico **WordCount**, com entradas enviadas ao HDFS.

### Criação do input:

```bash
hdfs dfs -mkdir /input
hdfs dfs -put /usr/local/hadoop/etc/hadoop/*.xml /input
```

### Execução do job WordCount:

```bash
hadoop jar \
  /opt/hadoop/share/hadoop/mapreduce/hadoop-mapreduce-examples-*.jar \
  wordcount /input /output
```

### 2.2.1 Alterações realizadas no cluster (5 mudanças)

1. **Alteração do fator de replicação** (`dfs.replication`: 3 → 1)
   *Efeito observado: [descrever]*

2. **Mudança na memória dos NodeManagers**
   Ajuste em `yarn.nodemanager.resource.memory-mb`.
   *Efeito observado: [descrever]*

3. **Alteração no número de reducers**
   *Efeito observado: [descrever]*

4. **Alteração no tamanho do buffer de ordenação** (`mapreduce.task.io.sort.mb`)
   *Efeito observado: [descrever]*

5. **Execução simultânea de dois jobs WordCount**
   *Efeito observado: [descrever]*

*(Inserir prints do YARN antes/depois de cada mudança)*



## 2.3 Teste de Tolerância a Falhas

Durante a execução contínua do WordCount, foram realizadas simulações de falha:

### Cenário 1 – Queda de um DataNode

```bash
docker stop datanode1
```

**Resultados observados:**

* Job continuou / atrasou / falhou: *[descrever]*
* Tempo de execução antes/depois: *[descrever]*
* Logs do YARN: *[inserir prints]*

### Cenário 2 – Queda de um NodeManager durante execução

*[descrever]*

### Cenário 3 – Retorno do nó ao cluster

*[descrever]*

### Conclusões do experimento Hadoop

* Impacto do aumento de nós no desempenho: *[descrever]*
* Nível de tolerância a falhas observado: *[descrever]*
* Vantagens e desvantagens observadas:

  * **Vantagens:** escalabilidade horizontal, resiliência, HDFS robusto.
  * **Desvantagens:** alta configuração, consumo de recursos, latência para jobs pequenos.



# 3. **Experimento com Apache Spark**

## 3.1 Arquitetura e Configuração

O experimento Spark foi implementado em um **Google Colab Notebook**, com:

* Spark 3.x em modo local
* Kafka Python para simulação de stream
* Biblioteca WordCloud substituindo Elastic/Kibana devido a restrições do ambiente

### Instalação realizada:

```python
!apt-get install openjdk-8-jdk-headless -qq
!wget -q http://apache.mirrors.tds.net/spark/spark-3.5.0-bin-hadoop3.tgz
!tar xf spark-3.5.0-bin-hadoop3.tgz
!pip install -q findspark pyspark kafka-python wordcloud
```



## 3.2 Configuração da Entrada (Rede Social / Simulação)

Devido à limitação do Colab e políticas de API, não foi possível integrar diretamente com uma rede social real.
Assim, foi adotado um **gerador de mensagens simuladas**, replicando um fluxo contínuo de dados:

```python
import time, random

msgs = ["spark is fast", "hadoop is reliable", "big data rocks", "data engineering"]
with open("stream_input.txt", "w") as f:
    for _ in range(1000):
        f.write(random.choice(msgs) + "\n")
        time.sleep(0.01)
```

*(Inserir justificativa formal no texto do relatório)*



## 3.3 Configuração da Saída Gráfica (substituição de Elastic/Kibana)

A biblioteca **WordCloud** foi usada para representar visualmente a frequência das palavras:

```python
from wordcloud import WordCloud
from collections import Counter
import matplotlib.pyplot as plt

words = open("stream_input.txt").read().split()
wc = WordCloud(width=800, height=400).generate_from_frequencies(Counter(words))
plt.imshow(wc)
plt.axis("off")
plt.show()
```

*(Inserir o gráfico gerado no relatório)*



## 3.4 Contador de Palavras com Spark

```python
from pyspark.sql import SparkSession
spark = SparkSession.builder.appName("WordCount").getOrCreate()

rdd = spark.read.text("stream_input.txt").rdd.flatMap(lambda x: x[0].split(" "))
counts = rdd.map(lambda w: (w,1)).reduceByKey(lambda a,b: a+b)
counts.collect()
```

### Resultados obtidos:

*(Inserir tabela do output)*



## 3.5 Dificuldades e Aprendizados

* Dificuldade de integração com APIs de redes sociais no Colab
* Restrições para execução de ElasticSearch/Kibana
* Ajustes de ambiente Spark e dependências
* Aprendizado sobre processamento distribuído, latência, paralelismo e pipelines de dados em streaming



# 4. **Conclusão**

Os experimentos permitiram compreender as diferenças entre os frameworks:

* **Hadoop** é robusto, altamente tolerante a falhas e ideal para processamento em lote.
* **Spark** é mais rápido, interativo e eficiente para workloads em streaming e análises rápidas.

### O que aprendemos:

* Como configurar clusters e interpretar dashboards do Hadoop
* Como o YARN escala jobs e como responde à falta de nós
* Como implementar pipelines de streaming e visualização de dados com Spark



# 5. **Opinião Individual dos Integrantes**

### *Integrante 1 — Nome:*

*[escrever texto curto]*

### *Integrante 2 — Nome:*

*[escrever texto curto]*

### *Integrante 3 — Nome:*

*[escrever texto curto]*

### *Integrante 4 — Nome:*

*[escrever texto curto]*

### *Integrante 5 — Nome:*

*[escrever texto curto]*



# 6. **Apêndice / Anexos**

### A.  Arquivos de Configuração Hadoop

* core-site.xml
* hdfs-site.xml
* yarn-site.xml
* mapred-site.xml

*(Inserir conteúdo ou link para pasta zip)*

### B. Prints dos Dashboards

*(Inserir todos prints aqui)*

### C. Código completo dos notebooks

*(Inserir export ou anexar no .zip)*



