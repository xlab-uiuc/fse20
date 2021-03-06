# cDep: Configuration Dependency Analysis

cDep is a tool for discovering configuration dependencies both within and across software components. cDep analyzes Java bytecode of the target software programs (supporting both Java and Scala code). It outputs specific types of dependencies and the corresponding interdependent configuration parameters. 

cDep currently supports:
 
<div align="left">
  <img src="https://github.com/xlab-uiuc/cdep-fse/blob/master/figure/sw_supported.png" width="750">
</div>

<br>

The whole artifact is accessbile from https://doi.org/10.5281/zenodo.3877326 and a github repository https://github.com/xlab-uiuc/cdep-fse.git.

The repository contains **all the artifacts** (including all the code and datasets) of the paper

* **Understanding and Discovering Software Configuration Dependencies in Cloud and Datacenter Systems** <br>
Qingrong Chen, Teng Wang, Owolabi Legunsen, Shanshan Li, and Tianyin Xu <br>
In Proceedings of the ACM Joint European Software Engineering Conference and Symposium on the Foundations of Software Engineering (ESEC/FSE 2020), November 8-13, 2020 Sacramento, CA.

----

## 1. Building and Running cDep

<div align="left">
  <img src="https://github.com/xlab-uiuc/cdep-fse/blob/master/figure/build.png" width="250">
</div>

### 1.1 Docker Container Image

We prepared a Docker container image, with which you can directly interact with the pre-built cDep.

The cDep Docker image is hosted on Docker hub and it will be automatically downloaded when you run the following command.

To run the Docker image, there is one CLI option:

* `-a <arg>`: where `<arg>` is a comma-separated list of elements in `hdfs`, `mapreduce`, `yarn`, `hadoop_common`, 
                     `hadoop_tools`, `hbase`, `alluxio`, `zookeeper`, `spark`

An example running command is as follows:
```
$ git clone https://github.com/xlab-uiuc/cdep-fse.git
$ cd cdep-fse
$ ./dockerrun.sh -a hdfs,mapreduce
```
The results will be stored at `/tmp/output/cDep_result.csv`.

**The analysis could take several tens of minutes (so be patient).**

### 1.2 Build Docker Image Locally

We provide the Dockerfile as well, with which you could build the docker image locally and run the program.

To build the docker image:
```
$ git clone https://github.com/xlab-uiuc/cdep-fse.git
$ cd cdep-fse
$ docker build -t cdep/cdep:1.0 .
```

Then the running command is same as above. An example running command is:
```
$ ./dockerrun.sh -a hdfs,mapreduce
```

### 1.3 Building cDep in Your Own Environment

We build cDep using Java(TM) SE Runtime Environment (build 12.0.2+10) and Apache Maven 3.6.1.
We did not test on other Java versions.

First, clone the repository,
```
$ git clone https://github.com/xlab-uiuc/cdep-fse.git
$ cd cdep-fse
```

Second, build cDep (we use Maven as the build tool for cDep)
```
$ mvn compile
```
After compiling, `cDep.class` should be generated at `target/classes/cdep/cDep.class`.

Third, use the script `run.sh`. One example running command is as follows:
```
$ ./run.sh -a hdfs,mapreduce
```

### 1.4 Machine Specifications

To run cDep with Maven, we recommend you have at least 8GB memory since all the jars which will be analyzed takes 1.4 GB already. 

Our evaluation result is done on the server with with CentOS 7.8.2003, 62GB memory and 40 Intel(R) Xeon(R) CPU E5-2660 v3 @ 2.60GHz. 

----

## 2. Reproducibility
<div align="left">
  <img src="https://github.com/xlab-uiuc/cdep-fse/blob/master/figure/repro.png" width="150">
</div>

<br>
 
**All the results in the paper, including both the study dataset and the cDep results can be reproduced.**

The cDep results can be reproduced by running cDep and it could take several hours:
```
$ ./dockerrun.sh  -a  hdfs,mapreduce,yarn,hadoop_common,hadoop_tools,hbase,alluxio,zookeeper,spark
```

The `cDep_result.csv` is in the format of:
`["parameter A","parameter B","dependency type","java class","java method","jimple stmt"]`

The output means `parameter A` and `parameter B` have a `dependency type`. And that dependency relation is identified in the `jimple stmt` of a certain `java method` and `java class`.

The following shows an example of a dependency cDep extracts from MapReduce:

```
(
  'mapreduce.output.fileoutputformat.compress',
  'mapreduce.output.fileoutputformat.compress.type',
  'control dependency',
  'org.apache.hadoop.mapred.MapFileOutputFormat',
  '<org.apache.hadoop.mapred.MapFileOutputFormat:org.apache.hadoop.mapred.RecordWriter getRecordWriter(org.apache.hadoop.fs.FileSystem,org.apache.hadoop.mapred.JobConf,java.lang.String,org.apache.hadoop.util.Progressable)>', 
  'if $z0 == 0 goto $r7 = new org.apache.hadoop.io.MapFile$Writer'
)
```

The two parameters, `mapreduce.output.fileoutputformat.compress` and `mapreduce.output.fileoutputformat.compress.type`, have a control dependency. And that relation is found from class `org.apache.hadoop.mapred.MapFileOutputFormat`.

----

## 3. Datasets
<div align="left">
  <img src="https://github.com/xlab-uiuc/cdep-fse/blob/master/figure/dataset.png" width="120">
</div
 
<br>
 
We also release all the dataset included in the paper under the `dataset` directory.

### 3.1 Configuration Dependency Dataset

It contains the following five files:
* `hadoop_intra.csv` : Intra-component dependencies in each individual component of the Hadoop-based stack;
* `hadoop_inter.csv` : Inter-component dependencies across components of the Hadoop-based stack;
* `openstack_intra.csv` : Intra-component dependencies in each individual component of OpenStack;
* `openstack_inter.csv` : Inter-component dependencies across components of OpenStack;
* `one_off_dep.csv` : One-off dependencies described in Section 4.3.

All the data sheets are in the format of CSV, with the first row describing the meaning of each column.

The data sheets provide detailed labels of the analysis results presented in our study.

### 3.2 cDep Findings

The found dependency cases from cDep can be found at `cDep_result`.
It contains the following two files:
* `intra.csv` : Intra-component dependencies in each individual component of the Hadoop-based stack;
* `inter.csv` : Inter-component dependencies across components of the Hadoop-based stack;

All the data sheets are in the format of CSV, with the first row describing the meaning of each column.

----

## 4. Code Structure

The following graph shows the end-to-end workflow of cDep:

<div align="left">
  <img src="https://github.com/xlab-uiuc/cdep-fse/blob/master/figure/cdep_overview.png" width="750">
</div>

<br>

The source code of `cdep` is placed under the `src/main/java` directory.

It contains the following main modules:
* `configinterface` implements the configuration interface methods to read configuration values in different projects;
* `dataflow` implements the inter-procedure and intra-procedure taint tracking;
* `handlingdep` implements the methods to capture different types of configuration dependencies;
* `utility` implements utility methods.

----

## 5. Verification and Validation

We show some configuration dependency cases (found by cDep) and explain why they are dependent on each other.

### 5.1 Control Dependency
If the first parameter is true, the second parameter will work.
1. `fs.client.resolve.topology.enabled`
2. `net.topology.node.switch.mapping.impl`

Code snippets:
```java
private void initTopologyResolution(Configuration config) {
 topologyResolutionEnabled = config.getBoolean(
      FS_CLIENT_TOPOLOGY_RESOLUTION_ENABLED,
     FS_CLIENT_TOPOLOGY_RESOLUTION_ENABLED_DEFAULT);
  if (!topologyResolutionEnabled) {
    return;
  }
 DNSToSwitchMapping dnsToSwitchMapping = ReflectionUtils.newInstance(
      config.getClass(
CommonConfigurationKeys.NET_TOPOLOGY_NODE_SWITCH_MAPPING_IMPL_KEY, ScriptBasedMapping.class, DNSToSwitchMapping.class), config);
}
```

### 5.2 Value Relationship Dependency

If the first parameter is not `null`, then the second parameter has to be `kerberos` to enable authentication.
1. `hbase.thrift.security.qop`
2. `hadoop.security.authentication`

Code snippets:
```java
if (qop != null) {
    ...
    if (!securityEnabled) {
        throw new IOException("Thrift server must run in secure mode to support authentication");
    }
}
```
(`qop` stores values of the first parameter, while `securityEnabled` takes the value from the second parameter.)


### 5.3 Overwrite Dependency

The second parameter overwrites the first parameter. 
1. `hadoop.security.service.user.name.key`
2. `mapreduce.jobhistory.principal`

Code snippets:
```java
private Configuration addSecurityConfiguration(Configuration conf) {
    ...
    conf.set(CommonConfigurationKeys.HADOOP_SECURITY_SERVICE_USER_NAME_KEY,
             conf.get(JHAdminConfig.MR_HISTORY_PRINCIPAL, ""));
    return conf;
}
```

### 5.4 Default Value Dependency 

If the value of the first parameter is not available, the second parameter will serve as its default value.
1. `yarn.resourcemanager.fail-fast`
2. `yarn.fail-fast`

Code snippets:
```java
public static boolean shouldRMFailFast(Configuration conf) {
    return conf.getBoolean(YarnConfiguration.RM_FAIL_FAST,
        conf.getBoolean(YarnConfiguration.YARN_FAIL_FAST,
            YarnConfiguration.DEFAULT_YARN_FAIL_FAST));
}
```

### 5.5 Behavioral Dependency

The first and second parameters work together to determine an IP address.
1. `fs.ftp.host`
2. `fs.ftp.host.port`

Code snippets:
```java
private FTPClient connect() throws IOException {
    FTPClient client = null;
    Configuration conf = getConf();
    String host = conf.get(FS_FTP_HOST);
    int port = conf.getInt(FS_FTP_HOST_PORT, FTP.DEFAULT_PORT);
    ...
    client.connect(host, port);
}
```
### 5.6 Sanity Check
The fastest job cDep could finish is over the application `hdfs`. The command is as follows,
```
$ ./run.sh -a hdfs
```
It takes around 15 minutes to finish this job over a Macbook Pro with 16GB memory and 2.6 GHz 6-Core Intel Core i7.

### 5.7 Demo Case
We provide one running case which could finish in one minute to further validate our tools. To run the demo case,
First, build cDep (we use Maven as the build tool for cDep)
```
$ mvn compile
```
Second, run cDep
```
$ ./demo.sh
```
The result contains the case from 5.4.

### 6. Reusability
1. First, cDep could be well adapted for more applications and no more dependencies are needed for that. 
We implement one [configuration interface](https://github.com/xlab-uiuc/cdep-fse/blob/master/src/main/java/configinterface/ConfigInterface.java). The main reason is because different projects have different configuration interfaces to get configuration values. So, users just need to implement the interface for their own project such as an exmaple for [Hadoop](https://github.com/xlab-uiuc/cdep-fse/blob/master/src/main/java/configinterface/HadoopInterface.java). Another thing they need to do is to download their own project source codes and put it under the [app directory](https://github.com/xlab-uiuc/cdep-fse/tree/master/app). The last thing they need to do is to add their application to the current [command line interface](https://github.com/xlab-uiuc/cdep-fse/blob/62568ad2f2cb488b96a8243dbc77e4a4e17ef96a/src/main/java/cdep/cDep.java#L31). 
The interface will be passed into dataflow and control flow analysis as in the [code](https://github.com/xlab-uiuc/cdep-fse/blob/042c8f109229c188e3aaa453b95eaced3030bc96/src/main/java/dataflow/DataTransform.java#L31). Later, this interface will be used to recognize getter and setter functions (i.e. starting points for confiugration analysis) in the speciifc project such as the [code](https://github.com/xlab-uiuc/cdep-fse/blob/042c8f109229c188e3aaa453b95eaced3030bc96/src/main/java/dataflow/DataTransform.java#L163).

2. Second, although not all projects need to do configuration dependency analysis, we believe the taint tracking of configuration parameters will benefit a lot of configuration related projects. The [file](https://github.com/xlab-uiuc/cdep-fse/blob/master/src/main/java/dataflow/DataTransform.java) and [file](https://github.com/xlab-uiuc/cdep-fse/blob/master/src/main/java/dataflow/IntraProcedureAnalysis.java) implement all the logic for doing inter/intra data/control flow analysis for configuration parameters. The results of the taint tacking are stored in the [object](https://github.com/xlab-uiuc/cdep-fse/blob/62568ad2f2cb488b96a8243dbc77e4a4e17ef96a/src/main/java/dataflow/DataTransform.java#L132). So, if other projects want to analyze the results, they could just run the analysis, get the object and do the analysis they want. We use this object to do dependency analysis while it could be used for other analysis for sure.
