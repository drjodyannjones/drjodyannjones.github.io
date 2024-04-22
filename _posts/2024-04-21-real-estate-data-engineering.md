---
title: "Streaming Real Estate Data Engineering Application"
layout: post
---

In this project, I designed and implemented a real-time streaming end-to-end data engineering pipeline that captures real estate listings from [Zoopla](https://www.zoopla.co.uk/){:target="\_blank"} using the [BrightData API](https://brightdata.com/){:target="\_blank"}. The data flows through a Kafka cluster, a message broker, which effectively manages the movement of data from the source to the storage system (sink), in this case, Cassandra DB. Utilizing Apache Spark, the pipeline handles large-scale data processing efficiently. This setup is specifically engineered to optimize real estate market analysis, providing a robust tool for dynamic and precise market evaluation. For an in-depth look at the project, you are welcome to visit my [GitHub repository](https://github.com/drjodyannjones/RealEstateDataEngineering){:target="\_blank"}.

### Project Overview

The **Real Estate Data Engineering** project leverages cutting-edge data processing technologies to generate actionable insights from comprehensive real estate datasets. This initiative enhances decision-making capabilities for stakeholders by providing sophisticated tools for analyzing market trends, property valuations, and investment opportunities in real-time. By integrating real-time data streaming with advanced analytical processes, this project supports a wide range of applications, including:

- **Market Trend Analysis**: Analysts can detect market shifts and emerging trends as they happen, enabling faster strategic responses.
- **Investment Decision Support**: Investors gain access to up-to-date information on property values and market conditions, aiding in better investment choices.
- **Portfolio Management**: Real estate companies can manage their assets more effectively by having current data at their fingertips, facilitating better operational and financial decisions.
- **Risk Management**: By analyzing current and historical data trends, risks associated with investments can be better assessed and mitigated.

This project exemplifies the power of integrating multiple technologies to transform raw data into a valuable strategic asset, driving forward the capabilities of real estate market analytics.

### Technologies Employed

- **Python:** Used for scripting data collection, transformation, and aggregation processes.
- **Apache Spark:** Employed to efficiently manage large-scale data processing.
- **Docker:** Applied to containerize the development environment to ensure consistency across different platforms.
- **Apache Kafka** A cluster of Apache Kafka brokers used as the conduit to transmit data from the source to sink with low latency.
- **Brightdata API** I setup a Brightdata web scraper to seamlessly web scrape real estate listings in the UK from Zoopla's website.
- **CassandraDB** A NoSQL database designed to manage large amounts of data. This is our sink
- **OpenAI** This helps with the querying of property listings on Zoopla's website.

## Getting Started

### Installation

Step 1. Clone the repository to your local machine:

```bash
git clone https://github.com/drjodyannjones/RealEstateDataEngineering.git
```

Step 2. Building Docker Image:

```bash
docker build -t my-custom-spark:3.5.0 .
```

Step 3. Start Docker Container (make sure the Docker client is up and running on your machine first!)

```bash
docker compose up -d
```

Step 3. Start Data Ingestion process:

```bash
python main.py
```

Step 4. Start Spark Consumer:

```bash
docker exec -it realestatedataengineering-spark-master-1 spark-submit \
    --packages com.datastax.spark:spark-cassandra-connector_2.12:3.4.1,org.apache.spark:spark-sql-kafka-0-10_2.12:3.3.0 \
    jobs/spark-consumer.py
```

### Sample Code Snippet: spark-consumer.py

In this example, I showcase how Apache Spark serves as an efficient consumer by extracting data from Apache Kafka and subsequently storing it in CassandraDB:

{% highlight python %}
import logging
from cassandra.cluster import Cluster
from pyspark.sql import SparkSession
from pyspark.sql.functions import from_json, col
from pyspark.sql.types import StructType, StructField, StringType, FloatType, ArrayType

# Configure logging

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(**name**)

def get_cassandra_session():
"""Retrieve or create a Cassandra session."""
if 'cassandra_session' not in globals():
cluster = Cluster(["cassandra"])
globals()['cassandra_session'] = cluster.connect()
return globals()['cassandra_session']

def setup_cassandra(session):
"""Setup the keyspace and table in Cassandra."""
session.execute("""
CREATE KEYSPACE IF NOT EXISTS property_streams
WITH replication = {'class': 'SimpleStrategy', 'replication_factor': '1'};
""")
logger.info("Keyspace created successfully!")

    session.execute("""
        CREATE TABLE IF NOT EXISTS property_streams.properties (
            price text, title text, link text, pictures list<text>, floor_plan text,
            address text, bedrooms text, bathrooms text, receptions text, epc_rating text,
            tenure text, time_remaining_on_lease text, service_charge text,
            council_tax_band text, ground_rent text, PRIMARY KEY (link)
        );
    """)
    logger.info("Table created successfully!")

def insert_data(\*\*kwargs):
"""Insert data into Cassandra table using a session created at the executor."""
session = get_cassandra_session()
session.execute("""
INSERT INTO property_streams.properties (
price, title, link, pictures, floor_plan, address, bedrooms, bathrooms,
receptions, epc_rating, tenure, time_remaining_on_lease, service_charge, council_tax_band, ground_rent
) VALUES (%s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s)
""", (
kwargs['price'], kwargs['title'], kwargs['link'], kwargs['pictures'],
kwargs['floor_plan'], kwargs['address'], kwargs['bedrooms'], kwargs['bathrooms'],
kwargs['receptions'], kwargs['epc_rating'], kwargs['tenure'], kwargs['time_remaining_on_lease'],
kwargs['service_charge'], kwargs['council_tax_band'], kwargs['ground_rent']
))
logger.info("Data inserted successfully!")

def define_kafka_to_cassandra_flow(spark):
"""Define data flow from Kafka to Cassandra using Spark."""
schema = StructType([
StructField("price", FloatType(), True),
StructField("title", StringType(), True),
StructField("link", StringType(), True),
StructField("pictures", ArrayType(StringType()), True),
StructField("floor_plan", StringType(), True),
StructField("address", StringType(), True),
StructField("bedrooms", StringType(), True),
StructField("bathrooms", StringType(), True),
StructField("receptions", StringType(), True),
StructField("epc_rating", StringType(), True),
StructField("tenure", StringType(), True),
StructField("time_remaining_on_lease", StringType(), True),
StructField("service_charge", StringType(), True),
StructField("council_tax_band", StringType(), True),
StructField("ground_rent", StringType(), True)
])

    kafka_df = (spark
                .readStream
                .format("kafka")
                .option("kafka.bootstrap.servers", "kafka-broker:9092")
                .option("subscribe", "properties")
                .option("startingOffsets", "earliest")
                .load()
                .selectExpr("CAST(value AS STRING) as value")
                .select(from_json(col("value"), schema).alias("data"))
                .select("data.*"))

    kafka_df.writeStream.foreachBatch(
        lambda batch_df, _: batch_df.foreach(
            lambda row: insert_data(**row.asDict())
        )
    ).start().awaitTermination()

def main():
spark = SparkSession.builder.appName("RealEstateConsumer").config(
"spark.cassandra.connection.host", "cassandra"
).config(
"spark.jars.packages",
"com.datastax.spark:spark-cassandra-connector_2.12:3.4.1,org.apache.spark:spark-sql-kafka-0-10_2.12:3.3.0"
).getOrCreate()

    session = get_cassandra_session()
    setup_cassandra(session)
    define_kafka_to_cassandra_flow(spark)

if **name** == "**main**":
main()
{% endhighlight %}
