---
title: "Apache Beam Java multi-language pipelines quickstart メモ"
emoji: "🧱"
type: "tech"
topics:
  - "apachebeam"
  - "dataflow"
  - "java"
  - "python"
published: true
published_at: "2022-11-22 09:36"
---

# 概要

Apache BeamのJavaのパイプラインからPythonの処理を呼び出すことのできるJava multi-language pipelinesの簡単な例（ https://beam.apache.org/documentation/sdks/java-multi-language-pipelines/ ）を実行したのだが、環境を作るのに少々煩雑であったので、そのメモ。

# 経緯

Apache Beamの2.40.0からRunInference transformが導入され、Apache Beam PythonではPyTorchとscikit-learnのモデルが直接サポートされるようになった。

- https://shunyaueta.com/posts/2022-08-18-1938/
- https://beam.apache.org/documentation/sdks/python-machine-learning/
- https://cloud.google.com/blog/products/data-analytics/latest-dataflow-innovations-for-real-time-streaming-and-aiml

Apache Beam 2.41.0では、Multi-language Pipelines framework という仕組みを使ってPythonのRunInference transform経由でPyTorchとscikit-learnのモデルを呼び出すことができるようになった。

今回は、まずMulti-language Pipelines frameworkにより簡単なPythonコードの呼び出しを試すところまでを確認した。

# 手順

以下、PythonDataframeWordCount（ https://github.com/apache/beam/blob/b8ca0819529e0bafaae0c08abec7c4e5682d6b50/examples/multi-language/src/main/java/org/apache/beam/examples/multilanguage/PythonDataframeWordCount.java ）をDataflowで実行するまでの手順である。 

## 環境

https://beam.apache.org/documentation/sdks/java-multi-language-pipelines/#prerequisites にあるように以下の環境が必要になるようである。

- Apache Beam Java SDK 2.41.0
- Apache Beam Python SDK 2.41.0
- Java 11
- Python 3.8
- pandas 1.4.0

## Apache Beam Java SDK

Apache Beam Java SDKは、2.41.0以降が必要だが、gradleの環境とmulti-languageのexampleが必要なため、githubのリポジトリから最新のコードをクローンしてくる

```
~/tmp
$ git clone https://github.com/apache/beam.git      
Cloning into 'beam'...
remote: Enumerating objects: 728092, done.
remote: Counting objects: 100% (136/136), done.
remote: Compressing objects: 100% (72/72), done.
remote: Total 728092 (delta 78), reused 93 (delta 52), pack-reused 727956
Receiving objects: 100% (728092/728092), 331.66 MiB | 4.14 MiB/s, done.
Resolving deltas: 100% (358643/358643), done.
Updating files: 100% (10779/10779), done.

~/tmp
$ cd beam 

~/tmp/beam
$ 
```

## Java, Python

次に、jenv, pyenvでローカルのJava, Pythonのバージョンを指定する

```
~/tmp/beam
$ jenv local oracle64-11.0.16.1               

~/tmp/beam
$ java -version
java version "11.0.16.1" 2022-08-18 LTS
Java(TM) SE Runtime Environment 18.9 (build 11.0.16.1+1-LTS-1)
Java HotSpot(TM) 64-Bit Server VM 18.9 (build 11.0.16.1+1-LTS-1, mixed mode)

~/tmp/beam
$ gradle -v

Welcome to Gradle 7.5.1!

Here are the highlights of this release:
 - Support for Java 18
 - Support for building with Groovy 4
 - Much more responsive continuous builds
 - Improved diagnostics for dependency resolution

For more details see https://docs.gradle.org/7.5.1/release-notes.html


------------------------------------------------------------
Gradle 7.5.1
------------------------------------------------------------

Build time:   2022-08-05 21:17:56 UTC
Revision:     d1daa0cbf1a0103000b71484e1dbfe096e095918

Kotlin:       1.6.21
Groovy:       3.0.10
Ant:          Apache Ant(TM) version 1.10.11 compiled on July 10 2021
JVM:          11.0.16.1 (Oracle Corporation 11.0.16.1+1-LTS-1)
OS:           Mac OS X 12.5.1 aarch64


~/tmp/beam
$ pyenv local 3.8.13                          

~/tmp/beam
$ python --version
Python 3.8.13

~/tmp/beam
$
```

## Apache Beam Python SDK

次にApache Beam Python SDKをインストールする。

https://beam.apache.org/get-started/quickstart-py/ の手順のままである。

```
~/tmp/beam
$ python -m venv .

~/tmp/beam
$ . bin/activate
(beam) 
~/tmp/beam
$ pip install apache-beam
Collecting apache-beam
  Using cached apache-beam-2.41.0.zip (2.9 MB)
  Preparing metadata (setup.py) ... done
  
:

  Running setup.py install for grpcio ... done
  Running setup.py install for apache-beam ... done
Successfully installed apache-beam-2.41.0 certifi-2022.6.15 charset-normalizer-2.1.1 cloudpickle-2.1.0 crcmod-1.7 dill-0.3.1.1 docopt-0.6.2 fastavro-1.6.0 grpcio-1.47.0 hdfs-2.7.0 httplib2-0.20.4 idna-3.3 numpy-1.22.4 orjson-3.8.0 proto-plus-1.22.1 protobuf-3.20.1 pyarrow-7.0.0 pydot-1.4.2 pymongo-3.12.3 pyparsing-3.0.9 python-dateutil-2.8.2 pytz-2022.2.1 requests-2.28.1 six-1.16.0 typing-extensions-4.3.0 urllib3-1.26.12
(beam) 
~/tmp/beam
$
```

## pandas

pandas 1.4.0以降が必要とあるので、venvの環境下でインストールする。

```
(beam) 
~/tmp/beam
$ pip install pandas
Collecting pandas
  Using cached pandas-1.4.4-cp38-cp38-macosx_11_0_arm64.whl (10.3 MB)
Requirement already satisfied: numpy>=1.20.0 in ./lib/python3.8/site-packages (from pandas) (1.22.4)
Requirement already satisfied: pytz>=2020.1 in ./lib/python3.8/site-packages (from pandas) (2022.2.1)
Requirement already satisfied: python-dateutil>=2.8.1 in ./lib/python3.8/site-packages (from pandas) (2.8.2)
Requirement already satisfied: six>=1.5 in ./lib/python3.8/site-packages (from python-dateutil>=2.8.1->pandas) (1.16.0)
Installing collected packages: pandas
Successfully installed pandas-1.4.4
(beam) 
~/tmp/beam
$
```

## ビルド・実行

gradleでPythonDataframeWordCountを実行する。

https://beam.apache.org/documentation/sdks/java-multi-language-pipelines/#run-with-dataflow-runner の `export PYTHON_VERSION=<version>` は不要だったのと、Dataflow Shuffleを無効にするために `--experiments=shuffle_mode=appliance` を追加している。

```
(beam) 
~/tmp/beam
$ export OUTPUT_BUCKET=<bucket>
export GCP_REGION=asia-northeast1
export TEMP_LOCATION=gs://$OUTPUT_BUCKET/tmp

./gradlew :examples:multi-language:pythonDataframeWordCount --args=" \
--runner=DataflowRunner \
--output=gs://${OUTPUT_BUCKET}/count \
--gcpTempLocation=${TEMP_LOCATION} \
--experiments=shuffle_mode=appliance \
--region=${GCP_REGION}"

Configuration on demand is an incubating feature.

> Task :examples:multi-language:pythonDataframeWordCount
9月 01, 2022 2:01:13 午後 org.apache.beam.sdk.extensions.gcp.options.GcpOptions$DefaultProjectFactory create

:
```

初回はbeamのビルドが実行されるため時間が掛かる。

以下のようなログが出てきたら、Dataflowにジョブが登録され実行された状態になっている。

```
:

9月 01, 2022 2:01:24 午後 org.apache.beam.runners.dataflow.DataflowRunner run
情報: To access the Dataflow monitoring console, please navigate to https://console.cloud.google.com/dataflow/jobs/asia-northeast1/2022-08-31_22_01_23-3571696049549206263?project=XXXXXXXX
9月 01, 2022 2:01:24 午後 org.apache.beam.runners.dataflow.DataflowRunner run
情報: Submitted job: 2022-08-31_22_01_23-3571696049549206263
9月 01, 2022 2:01:24 午後 org.apache.beam.runners.dataflow.DataflowRunner run
情報: To cancel the job using the 'gcloud' tool, run:
> gcloud dataflow jobs --project=XXXXXXXX cancel --region=asia-northeast1 2022-08-31_22_01_23-3571696049549206263
9月 01, 2022 2:01:27 午後 org.apache.beam.runners.dataflow.util.MonitoringUtil$LoggingHandler process
情報: 2022-09-01T05:01:25.205Z: Autoscaling is enabled for job 2022-08-31_22_01_23-3571696049549206263. The number of workers will be between 1 and 1000.

:
```

起動してから最初のtransform（ReadLinesと書いてあるもの）が起動するまで5分以上掛かるので心配になるが、しばらく待つ。

DataflowのコンソールでJob statusがSuccessになり、ターミナルで以下のように表示されれば正常終了である。

```
　:

9月 01, 2022 2:08:11 午後 org.apache.beam.runners.dataflow.util.MonitoringUtil$LoggingHandler process
情報: 2022-09-01T05:08:09.301Z: Worker pool stopped.
9月 01, 2022 2:08:15 午後 org.apache.beam.runners.dataflow.DataflowPipelineJob logTerminalState
情報: Job 2022-08-31_22_01_23-3571696049549206263 finished with status DONE.

Deprecated Gradle features were used in this build, making it incompatible with Gradle 8.0.

You can use '--warning-mode all' to show the individual deprecation warnings and determine if they come from your own scripts or plugins.

See https://docs.gradle.org/7.4/userguide/command_line_interface.html#sec:command_line_warnings

BUILD SUCCESSFUL in 7m 5s
74 actionable tasks: 1 executed, 73 up-to-date
(beam) 
~/tmp/beam
$ 
```


# その他気になったところ

## dataflowrunnerでのPython expansion service

今回起動したPythonDataframeWordCountでは、Python expansion serviceのURLを指定するexpansionServiceのオプションがあるが、これを省略した場合（今回の実行ケース）、PythonExternalTransformでこのサービスを起動しているようである。サービスを起動する時間が掛かったりして、最初のtransformの起動までに時間が掛かったりしているかもしれない。

https://github.com/apache/beam/blob/b8ca0819529e0bafaae0c08abec7c4e5682d6b50/sdks/java/extensions/python/src/main/java/org/apache/beam/sdk/extensions/python/PythonExternalTransform.java#L421-L427 

## dataflowrunner以外のrunner

dataflowrunner以外のdirectrunnerとportablerunnerを試したが、Python expansion serviceと連携するところで躓いているようで上手く行かなかった。
