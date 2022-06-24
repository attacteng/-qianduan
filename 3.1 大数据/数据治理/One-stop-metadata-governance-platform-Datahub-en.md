*   [One-stop metadata governance platform - Datahub](https://www.cnblogs.com/tree1123/p/15743253.html)

As digital transformation advances, data governance has been put on the agenda by more and more companies. As a new generation of metadata management platform, Datahub has grown rapidly in the past year, and has the potential to replace the old metadata management tool Atlas. Most companies want to use Datahub as their metadata management platform, but there are too few references.

So this document has been collated for everyone to learn and use. This document is based on**Datahub's latest version 0.8.20**, compiled from some of the official website content, various blogs and practice processes. The article is longer and recommended to be collected. For the new version of the document, please pay attention to the official account **Big data flows**, will continue to update ~

Through this document, you can quickly get started with Datahub, successfully set up Datahub, and obtain metadata information from the database. is from**Getting started documentation from 0 to 1**For more advanced features of Datahub, you can pay attention to subsequent article updates.

The document is divided into 6 parts, and the hierarchy is shown in the following figure.

![img](https://img2020.cnblogs.com/blog/1089984/202112/1089984-20211228222843096-35062279.png)

## 1. Data governance and metadata management

### background

Why data governance?  There are many businesses, many data, and business data is constantly iterating. Personnel flow, incomplete documentation, unclear logic, difficult to intuitively understand the data, difficult to maintain in the later stage.

In the development of big data, the original data has a lot of databases, data tables.

After the aggregation of data, there will be many dimension tables.

In recent years, the magnitude of data has been growing wildly, which has led to a series of problems. As a data underpinning for AI teams, the question we hear the most is "the right data set" and they need the right data for their analysis. We began to realize that while we built highly scalable data storage, real-time computing, and more, our team was still wasting time looking for the right data set to analyze.

That is, we lack the management of data assets. In fact, there are many companies that offer open source solutions to solve these problems, which is also known as data discovery and metadata management tools.

### Metadata management

Simply put, metadata management is about the efficient organization of data assets. It uses metadata to help manage their data. It also helps data professionals collect, organize, access, and enrich metadata to support data governance.

Thirty years ago, a data asset might have been a table in an Oracle database. However, in the modern enterprise, we have a dizzying array of different types of data assets. This could be tables in a relational database or NoSQL store, live streaming data, features in an AI system, metrics in a metrics platform, dashboards in a data visualizer.

Modern metadata management should include all of these types of data assets and enable data workers to do their jobs more efficiently.

Therefore, the functions that metadata management should have are as follows:

*   Search and discovery: Data tables, fields, labels, usage information
*   Access Control: Access control groups, users, policies
*   \*\*Data lineage:\*\*Pipeline execution, query.\*\*
*   \*\*Compliance:\*\*Classification of data privacy/compliance annotation types
*   Data Management: Data source configuration, ingestion configuration, retention configuration, data cleanup policy
*   \*\*AI interpretability, reproducibility: \*\*Feature definition, model definition, training run execution, problem statement
*   \*\*Data operations:\*\* Pipeline execution, processing data partitioning, data statistics
*   \*\*Data quality:\*\*Data quality rule definition, rule execution results, data statistics

### Architecture and open source solutions

The following describes the architectural implementation of metadata management, and different architectures correspond to different open source implementations.

The following diagram depicts the first-generation metadata schema. It is usually a classic monolithic front-end (possibly a Flask application), connected to the primary store for queries (usually MySQL/Postgres), a search index to serve search queries (usually Elasticsearch), and for the 1.5th generation of this architecture, perhaps once the "recursive query" limit for relational databases is reached, a graph index that processes genealogical (usually Neo4j) graph queries is used.

![img](https://img2020.cnblogs.com/blog/1089984/202112/1089984-20211228222918780-1404050909.jpg)

Soon, the second generation of architecture appeared. The monolithic application has been split into services that sit in front of the metadata store database. The service provides an API that allows metadata to be written to the system using a push mechanism.

![img](https://img2020.cnblogs.com/blog/1089984/202112/1089984-20211228222925623-326530673.jpg)

The third-generation architecture is an event-based metadata management architecture that allows customers to interact with the metadata database in different ways according to their needs.

Low-latency lookups for metadata, full-text and ranked searches of metadata properties, graphical queries of metadata relationships, and full scan and analysis capabilities.

![img](https://img2020.cnblogs.com/blog/1089984/202112/1089984-20211228222930805-14980056.jpg)

Datahub is one such architecture.

The following diagram is a simple visual representation of today's metadata landscape:

(Including some non-open source solutions)

![img](https://img2020.cnblogs.com/blog/1089984/202112/1089984-20211228223623121-5702114.jpg)

Other options can be used as the main direction of the survey, but are not the focus of this article.

## Second, Datahub Introduction

First of all, Alibaba Cloud also has a product called DataHub, which is a streaming platform, and DataHub described in this article has nothing to do with it.

Data governance is a hot topic that the big guys have been talking about lately. Both at the national level and at the corporate level, this issue is now being paid more and more attention. Data governance addresses data quality, data management, data assets, data security, and more. And the key to data governance is here**Metadata management**We must know the ins and outs of the data in order to carry out all-round management, monitoring, and insight of the data.

DataHub is a tool open sourced by LinkedIn's data team that provides metadata search and discovery.

When it comes to LinkedIn, we have to think of the famous Kafka, kafka is LinkedIn open source. LinkedIn's open source Kafka has directly impacted the entire real-time computing landscape, and LinkedIn's data team has also been exploring issues of data governance, constantly striving to expand its infrastructure to meet the needs of the growing big data ecosystem. As the volume and richness of data grows, it becomes increasingly challenging for data scientists and engineers to discover available data assets, understand their provenance, and take appropriate action based on insights. To help grow while continuing to scale productivity and data innovation, DataHub, a common metadata search and discovery tool, was created.

There are several common metadata management systems on the market:
a) linkedin datahub:
https://github.com/linkedin/datahub
b) apache atlas:
https://github.com/apache/atlas
c) lyft amundsen
https://github.com/lyft/amundsen

Atlas has also been introduced before, and has very good support for hive, but it is very difficult to deploy. Amundsen is still an emerging framework, there is no release version yet, and it may be slowly observed that the future development may be carried out.

In summary, datahub is currently a new star, but the current data on datahub is still small, and we will continue to pay attention to and update more information about datahub in the future.

At present, the number of github stars on datahub has reached 4.3k.

![img](https://img2020.cnblogs.com/blog/1089984/202112/1089984-20211228222940538-2111610741.png)

Datahub official website

Datahub describes it as Data ecosystems are diverse — too diverse. DataHub's  extensible metadata platform enables data discovery, data observability  and federated governance that helps you tame this complexity.

The data ecosystem is diverse, and DataHub provides a scalable metadata management platform that can meet the needs of data discovery, data observation and governance. This also greatly solves the problem of data complexity.

![img](https://img2020.cnblogs.com/blog/1089984/202112/1089984-20211228222949969-553769986.png)

Datahub provides rich data source support and bloodline display.

![img](https://img2020.cnblogs.com/blog/1089984/202112/1089984-20211228223001569-528078135.png)

When obtaining the data source, you only need to write a simple yml file to complete the acquisition of metadata.

![img](https://img2020.cnblogs.com/blog/1089984/202112/1089984-20211228223009985-1737509630.png)

In terms of data source support, DataHub supports data sources such as druid, hive, kafka, mysql, oracle, postgres, redash, metabase, superset, etc., and supports data lineage acquisition through airflow. It can be said that it was realized from**Full link of data sources to BI tools**Data lineage opened.

![img](https://img2020.cnblogs.com/blog/1089984/202112/1089984-20211228223025830-1417755340.png)

## Third, the Datahub interface

Through the Datahub page, we can briefly understand the functions that Datahub can meet.

### 3.1 Home

First, after logging in to Datahub, you enter the Datahub homepage, which provides a menu bar, search box, and a list of metadata information on The homepage. This is so that everyone can quickly manage the metadata.

Metadata information is categorized by dataset, dashboard, chart, etc.

![img](https://img2020.cnblogs.com/blog/1089984/202112/1089984-20211228223040025-1933685859.png)

Further down is the platform information, which includes the collection of platform information such as Hive, Kafka, and Airflow.

![img](https://img2020.cnblogs.com/blog/1089984/202112/1089984-20211228223047266-648084613.png)

Below are actually some of the search statistics. Used to count the most recent and popular search results.

Includes some label and glossary information.

![img](https://img2020.cnblogs.com/blog/1089984/202112/1089984-20211228223053444-1749291050.png)

### 3.2 Analyzing Pages

The analysis page is a statistic about metadata information, as well as statistics on users who use datahub.

It can be understood as a display page, which is still very necessary for the understanding of the overall situation.

![img](https://img2020.cnblogs.com/blog/1089984/202112/1089984-20211228223059667-353166676.png)

Other functions are basically controls over users and permissions.

![img](https://img2020.cnblogs.com/blog/1089984/202112/1089984-20211228223104835-1476178203.png)

## Fourth, the overall structure

In order to learn Datahub well, you must understand the overall architecture of Datahub.

Through Datahub's architecture diagram, you can clearly understand the architectural composition of Datahub.

![img](https://img2020.cnblogs.com/blog/1089984/202112/1089984-20211228223113440-1903906242.png)

The architecture of DataHub has three main parts.

The front end is the Datahub frontend as a page display for the front end.

The rich front-end showcase gives Datahub the ability to support most of its features. Its front-end is based on react framework research and development, for companies with secondary research and development intentions, pay attention to the matching of this technology stack.

The backend Datahub serves to provide storage services for the backend.

Datahub's back-end development language is Python, and storage is based on ES or Neo4J.

Datahub ingestion is used to extract metadata information.

Datahub provides proactive pull methods based on API metadata and real-time metadata acquisition based on Kafka. This is very flexible for obtaining metadata.

These three parts are also the main focus of our deployment process, let's deploy Datahub from scratch and get the metadata information of a database.

## Fifth, rapid installation and deployment

Deploying datahub has certain requirements for the system. This article is installed based on CentOS7.

Install docker, jq, docker-compose first. Also ensure that the python version of the system is Python 3.6+.

### 5.1, install docker, docker-compose, jq

Docker is an open source application container engine that allows developers to package their applications and dependencies into a portable container and then publish to any popular Linux or Windows operating system machine, or virtualization, containers are completely sandboxed and will not have any interface with each other.

Docker can be quickly installed by yum

    yum -y install docker

![img](https://img2020.cnblogs.com/blog/1089984/202112/1089984-20211228223123960-291893834.png)

Check the version by docker -v when you're done.

    # docker -v
    Docker version 1.13.1, build 7d71120/1.13.1

The following command allows you to start and stop docker

    systemctl start docker // 启动docker
    systemctl stop docker // 关闭docker

Docker Compose is then installed

[Docker Compose](https://github.com/docker/compose)is a command-line tool provided by Docker to define and run applications that consist of multiple containers. Using compose, we can define the individual services of an application declaratively through a YAML file, and create and launch the application with a single command.

    sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

Modify execute permissions

    sudo chmod +x /usr/local/bin/docker-compose

Establish a soft connection

    ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose

Review the version and verify that the installation was successful.

    docker-compose --version
    docker-compose version 1.29.2, build 5becea4c

Install jq

Install the EPEL source first, the Enterprise Linux Add-On Package (EPEL) is a Fedora Special Interest Group that creates, maintains, and manages a high-quality add-on package set for Enterprise Linux, including but not limited to Red Hat Enterprise Linux (RHEL), CentOS, Scientific Linux (SL), and Oracle Linux (OL).

EPEL packages typically do not conflict with or replace files with packages from the official Enterprise Linux repositories. The EPEL project is basically the same as Fedora and includes a complete build system, upgrade manager, image manager, and so on.

Install the EPEL source

    yum install epel-release 

After installing the EPEL source, you can check whether the following jq package exists:

    yum list jq

Install jq:

    yum install jq

### 5.2. Install Python3

Install dependencies

    yum -y groupinstall "Development tools"
    yum -y install zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gdbm-devel db4-devel libpcap-devel xz-devel libffi-devel

Download the installation package

    wget https://www.python.org/ftp/python/3.8.3/Python-3.8.3.tgz
    tar -zxvf  Python-3.8.3.tgz

Compile the installation

    mkdir /usr/local/python3 
    cd Python-3.8.3
    ./configure --prefix=/usr/local/python3
    make && make install

Modify the system default python pointer

    rm -rf /usr/bin/python
    ln -s /usr/local/python3/bin/python3 /usr/bin/python

Modify the system default pip pointer

    rm -rf /usr/bin/pip
    ln -s /usr/local/python3/bin/pip3 /usr/bin/pip

verify

    python -V

**Fix yum**

Python3 will cause yum not to work properly

    vi /usr/bin/yum 
    把 #! /usr/bin/python 修改为 #! /usr/bin/python2 
    vi /usr/libexec/urlgrabber-ext-down 
    把 #! /usr/bin/python 修改为 #! /usr/bin/python2
    vi /usr/bin/yum-config-manager
    #!/usr/bin/python 改为 #!/usr/bin/python2
    没有的不用修改

### 5.3. Install and start datahub

Start by upgrading pip

    python3 -m pip install --upgrade pip wheel setuptools

You need to see a successful return below.

     Attempting uninstall: setuptools
        Found existing installation: setuptools 57.4.0
        Uninstalling setuptools-57.4.0:
          Successfully uninstalled setuptools-57.4.0
      Attempting uninstall: pip
        Found existing installation: pip 21.2.3
        Uninstalling pip-21.2.3:
          Successfully uninstalled pip-21.2.3

Check the environment

    python3 -m pip uninstall datahub acryl-datahub || true  # sanity check - ok if it fails

There is no problem with receiving such a prompt to explain.

    WARNING: Skipping datahub as it is not installed.
    WARNING: Skipping acryl-datahub as it is not installed.

Install datahub, this step takes a long time, wait patiently.

    python3 -m pip install --upgrade acryl-datahub

Receiving a prompt indicates that the installation was successful.

    Successfully installed PyYAML-6.0 acryl-datahub-0.8.20.0 avro-1.11.0 avro-gen3-0.7.1 backports.zoneinfo-0.2.1 certifi-2021.10.8 charset-normalizer-2.0.9 click-8.0.3 click-default-group-1.2.2 docker-5.0.3 entrypoints-0.3 expandvars-0.7.0 idna-3.3 mypy-extensions-0.4.3 progressbar2-3.55.0 pydantic-1.8.2 python-dateutil-2.8.2 python-utils-2.6.3 pytz-2021.3 pytz-deprecation-shim-0.1.0.post0 requests-2.26.0 stackprinter-0.2.5 tabulate-0.8.9 toml-0.10.2 typing-extensions-3.10.0.2 typing-inspect-0.7.1 tzdata-2021.5 tzlocal-4.1 urllib3-1.26.7 websocket-client-1.2.3

Finally we see the version of datahub.

    [root@node01 bin]# python3 -m datahub version
    DataHub CLI version: 0.8.20.0
    Python version: 3.8.3 (default, Aug 10 2021, 14:25:56)
    [GCC 4.8.5 20150623 (Red Hat 4.8.5-44)]

Then launch datahub

    python3 -m datahub docker quickstart

Will go through a long download process, wait patiently.

![img](https://img2020.cnblogs.com/blog/1089984/202112/1089984-20211228223144345-693133501.png)

Start booting and watch for errors. If the internet speed is not good, you need to do it several times.

![img](https://img2020.cnblogs.com/blog/1089984/202112/1089984-20211228223150317-1436821699.png)

If you can see the following display, the installation is successful.

![img](https://img2020.cnblogs.com/blog/1089984/202112/1089984-20211228223200260-791947381.png)

Visit ip:9002 enter the datahub datahub login

![img](https://img2020.cnblogs.com/blog/1089984/202112/1089984-20211228223205807-1089312742.png)

## 6. Acquisition of metadata information

After logging in to Datahub, there will be a friendly welcome page. to suggest how to crawl metadata.

![img](https://img2020.cnblogs.com/blog/1089984/202112/1089984-20211228223213625-1820040877.png)

Metadata ingestion uses a plugin schema, and you only need to install the plugins you need.

There are many sources of intake

```text
插件名称	   安装命令	                          提供功能
mysql	pip install 'acryl-datahub[mysql]'	   MySQL source
```

Two plugins are installed here:

Source: mysql

Sink: datahub-rest

```text
pip install 'acryl-datahub[mysql]'
```

![img](https://img2020.cnblogs.com/blog/1089984/202112/1089984-20211228223220693-1750337153.png)

There are many packages installed, and the following prompts are obtained to prove that the installation was successful.

    Installing collected packages: zipp, traitlets, pyrsistent, importlib-resources, attrs, wcwidth, tornado, pyzmq, pyparsing, pycparser, ptyprocess, parso, nest-asyncio, jupyter-core, jsonschema, ipython-genutils, webencodings, pygments, prompt-toolkit, pickleshare, pexpect, packaging, nbformat, matplotlib-inline, MarkupSafe, jupyter-client, jedi, decorator, cffi, backcall, testpath, pandocfilters, nbclient, mistune, jupyterlab-pygments, jinja2, ipython, defusedxml, debugpy, bleach, argon2-cffi-bindings, terminado, Send2Trash, prometheus-client, nbconvert, ipykernel, argon2-cffi, numpy, notebook, widgetsnbextension, toolz, ruamel.yaml.clib, pandas, jupyterlab-widgets, jsonpointer, tqdm, termcolor, scipy, ruamel.yaml, jsonpatch, ipywidgets, importlib-metadata, altair, sqlalchemy, pymysql, greenlet, great-expectations
    Successfully installed MarkupSafe-2.0.1 Send2Trash-1.8.0 altair-4.1.0 argon2-cffi-21.3.0 argon2-cffi-bindings-21.2.0 attrs-21.3.0 backcall-0.2.0 bleach-4.1.0 cffi-1.15.0 debugpy-1.5.1 decorator-5.1.0 defusedxml-0.7.1 great-expectations-0.13.49 greenlet-1.1.2 importlib-metadata-4.10.0 importlib-resources-5.4.0 ipykernel-6.6.0 ipython-7.30.1 ipython-genutils-0.2.0 ipywidgets-7.6.5 jedi-0.18.1 jinja2-3.0.3 jsonpatch-1.32 jsonpointer-2.2 jsonschema-4.3.2 jupyter-client-7.1.0 jupyter-core-4.9.1 jupyterlab-pygments-0.1.2 jupyterlab-widgets-1.0.2 matplotlib-inline-0.1.3 mistune-0.8.4 nbclient-0.5.9 nbconvert-6.3.0 nbformat-5.1.3 nest-asyncio-1.5.4 notebook-6.4.6 numpy-1.21.5 packaging-21.3 pandas-1.3.5 pandocfilters-1.5.0 parso-0.8.3 pexpect-4.8.0 pickleshare-0.7.5 prometheus-client-0.12.0 prompt-toolkit-3.0.24 ptyprocess-0.7.0 pycparser-2.21 pygments-2.10.0 pymysql-1.0.2 pyparsing-2.4.7 pyrsistent-0.18.0 pyzmq-22.3.0 ruamel.yaml-0.17.19 ruamel.yaml.clib-0.2.6 scipy-1.7.3 sqlalchemy-1.3.24 termcolor-1.1.0 terminado-0.12.1 testpath-0.5.0 toolz-0.11.2 tornado-6.1 tqdm-4.62.3 traitlets-5.1.1 wcwidth-0.2.5 webencodings-0.5.1 widgetsnbextension-3.5.2 zipp-3.6.0

Then check the installed plugins, Datahub is a plug-in installation method. You can check the data source to get the plug-in Source, convert the plug-in transformer, and get the plug-in Sink.

     python3 -m datahub check plugins

![img](https://img2020.cnblogs.com/blog/1089984/202112/1089984-20211228223229124-1907560779.png)

It can be seen that the Mysql plugin and the Rest interface plugin have been installed, and the following configuration is to obtain metadata from MySQL and use the Rest interface to store data on DataHub.

    vim mysql_to_datahub_rest.yml
    # A sample recipe that pulls metadata from MySQL and puts it into DataHub
    # using the Rest API.
    source:
      type: mysql
      config:
        username: root
        password: 123456
        database: cnarea20200630

    transformers:
      - type: "fully-qualified-class-name-of-transformer"
        config:
          some_property: "some.value"

    sink:
      type: "datahub-rest"
      config:
        server: "http://ip:8080"

    # datahub ingest -c mysql_to_datahub_rest.yml

This is followed by a lengthy data acquisition process.

![img](https://img2020.cnblogs.com/blog/1089984/202112/1089984-20211228223241028-721609965.png)

After receiving the following prompts, the certificate is successful.

    {datahub.cli.ingest_cli:83} - Finished metadata ingestion

![img](https://img2020.cnblogs.com/blog/1089984/202112/1089984-20211228223248703-1273920148.png)

    Sink (datahub-rest) report:
    {'records_written': 356,
     'warnings': [],
     'failures': [],
     'downstream_start_time': datetime.datetime(2021, 12, 28, 21, 8, 37, 402989),
     'downstream_end_time': datetime.datetime(2021, 12, 28, 21, 13, 10, 757687),
     'downstream_total_latency_in_seconds': 273.354698}
    Pipeline finished with warnings

Refresh the datahub page here, and the metadata information of mysql has been successfully obtained.

![img](https://img2020.cnblogs.com/blog/1089984/202112/1089984-20211228223254823-783477836.png)

Go to the table to view the metadata situation, table field information.
![img](https://img2020.cnblogs.com/blog/1089984/202112/1089984-20211228223301349-1940431176.png)

The previous presentation of the metadata analysis page has also been shown in detail.

![img](https://img2020.cnblogs.com/blog/1089984/202112/1089984-20211228223306038-633461159.png)

At this point, we completed the construction of Datahub from 0 to 1, and basically did not carry out any code development work except for simple installation and configuration during the whole process. But datahub has more functions, such as the acquisition of data lineage, conversion operations in the process of metadata acquisition, and so on. Tutorials updating these features will also be included in future articles.
