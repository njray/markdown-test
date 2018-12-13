Batch scoring of Python models on Azure
=======================================

This reference architecture shows how to build a scalable solution for batch
scoring many models on a schedule in parallel using Batch AI. The solution can
be used as a template and can generalize to different problems. A reference
implementation for this architecture is available
on [GitHub](https://github.com/Azure/BatchAIAnomalyDetection).

![](media/23c95f0a97549d4778f978ba9eab85be.png)

**Scenario**: This solution monitors the operation of a large number of devices
in an IoT setting where each device sends sensor readings continuously. Each
device is assumed to have pre-trained anomaly detection models that need to be
used to predict whether a series of measurements, that are aggregated over a
predefined time interval, correspond to an anomaly or not. In real-world
scenarios, this could be a stream of sensor readings that need to be filtered
and aggregated before being used in training or real-time scoring. For
simplicity, the solution uses the same data file when executing scoring jobs.

Architecture
------------

This architecture consists of the following components:

[Azure Event
Hubs](https://docs.microsoft.com/en-us/azure/event-hubs/event-hubs-geo-dr). This
message ingestion service can ingest millions of event messages per second. In
this architecture, sensors send a stream of data to the event hub.

[Azure Stream
Analytics](https://docs.microsoft.com/en-us/azure/stream-analytics/). An
event-processing engine. A Stream Analytics job reads the data streams from the
event hub and performs stream processing.

Azure [Batch](https://docs.microsoft.com/en-us/azure/batch-ai/) AI. This
distributed computing engine is used to train and test machine learning and AI
models at scale in Azure. Batch AI creates virtual machines on demand with an
automatic scaling option, where each node in the Batch AI cluster runs a scoring
job for a specific sensor. The scoring
Python [script](https://github.com/Azure/BatchAIAnomalyDetection/blob/master/batchai/predict.py) runs
in Docker containers that are created on each node of the cluster, where it
reads the relevant sensor data, generates predictions and stores them in Blob
storage.

Azure Blob Storage. Blob containers are used to store the pretrained models, the
data, and the output predictions. The models are uploaded to Blob storage in the
[create\_resources.ipynb](https://github.com/Azure/BatchAIAnomalyDetection/blob/master/create_resources.ipynb)
notebook. These [one-class
SVM](http://scikit-learn.org/stable/modules/generated/sklearn.svm.OneClassSVM.html)
models are trained on data that represents values of different sensors for
different devices. This solution assumes that the data values are aggregated
over a fixed interval of time.

[Azure Logic
Apps](https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-overview).
This solution creates a Logic App that runs hourly Batch AI jobs. Logic Apps
provide an easy way to create the runtime workflow and scheduling for the
solution. The Batch AI jobs are submitted using a Python
[script](https://github.com/Azure/BatchAIAnomalyDetection/blob/master/sched/submit_jobs.py)
that also runs in a Docker container.

[Azure Container
Registry](https://docs.microsoft.com/en-us/azure/container-registry/container-registry-intro).
The Docker images used in both Batch AI and Logic Apps are created in the
[create\_resources.ipynb](https://github.com/Azure/BatchAIAnomalyDetection/blob/master/create_resources.ipynb)
notebook and pushed to Azure Container Registry. A container registry provides a
convenient way to host images and instantiate containers through other Azure
services—Logic Apps and Batch AI in this solution.

Performance considerations
--------------------------

For standard Python models, it's generally accepted that CPUs are sufficient to
handle the workload. This architecture uses CPUs. However, for [deep learning
workloads](https://docs.microsoft.com/en-us/azure/architecture/reference-architectures/ai/batch-scoring-deep-learning),
GPUs generally outperform CPUs by a considerable amount—a sizeable cluster of
CPUs is usually needed to get comparable performance.

### Parallelizing across VMs vs cores

When running scoring processes of many models in batch mode, the jobs need to be
parallelized across VMs. Two approaches are possible: create a larger cluster
using low cost VMs, or create a smaller cluster using high performing VMs with
more cores available on each.

In general, scoring of standard Python models is not as demanding as scoring of
deep learning models, and a small cluster should be able to handle a large
number of queued models efficiently. You can increase the number of cluster
nodes as the dataset sizes increase.

For convenience in this scenario, one scoring task is submitted within a single
Batch AI job. However, it can be more efficient to score multiple data chunks
within the same Batch AI job. In those cases, write custom code to read in
multiple datasets and execute the scoring script for those during a single Batch
AI job execution.

### File servers

When using Batch AI, you can choose multiple storage options depending on the
throughput needed for your scenario. For workloads with low throughput
requirements, using blob storage should be enough. Alternatively, Batch AI also
supports a [Batch AI File
Server](https://docs.microsoft.com/en-us/azure/batch-ai/resource-concepts#file-server),
a managed, single-node NFS, which can be automatically mounted on cluster nodes
to provide a centrally accessible storage location for jobs. For most cases,
only one file server is needed in a workspace, and you can separate data for
your training jobs into different directories.

If a single-node NFS isn't appropriate for your workloads, Batch AI supports
other storage options, including [Azure
Files](https://docs.microsoft.com/en-us/azure/storage/files/storage-files-introduction)
and custom solutions such as a Gluster or Lustre file system.

Management considerations
-------------------------

### Monitoring Batch AI jobs

It's important to monitor the progress of running jobs, but it can be a
challenge to monitor across a cluster of active nodes.

To get a sense of the overall state of the cluster, go to the **Batch AI** blade
of the [Azure Portal](https://portal.azure.com) to inspect the state of the
nodes in the cluster. If a node is inactive or a job has failed, the error logs
are saved to blob storage, and are also accessible in the **Jobs** blade of the
portal.

For richer monitoring, connect logs to [Application
Insights](https://docs.microsoft.com/en-us/azure/application-insights/app-insights-overview),
or run separate processes to poll for the state of the Batch AI cluster and its
jobs.

### Logging in Batch AI

Batch AI logs all stdout/stderr to the associated Azure storage account. For
easy navigation of the log files, use a storage navigation tool such as [Azure
Storage Explorer](https://azure.microsoft.com/en-us/features/storage-explorer/).

When you deploy this reference architecture, you have the option to set up a
simpler logging system. With this option, all the logs across the different jobs
are saved to the same directory in your blob container as shown below. Use these
logs to monitor how long it takes for each job and each image to process, so you
have a better sense of how to optimize the process.

![https://docs.microsoft.com/en-us/azure/architecture/reference-architectures/ai/\_images/batch-ai-logging.png](media/be732f935dbbe58a37f414fa2b87252c.png)

Cost considerations
-------------------

The most expensive components used in this reference architecture are the
compute resources.

The Batch AI cluster size scales up and down depending on the jobs in the queue.
You can enable [automatic
scaling](https://docs.microsoft.com/en-us/azure/batch/batch-automatic-scaling)
with Batch AI in one of two ways. You can do so programmatically, which can be
configured in the .env file that is part of the [deployment
steps](https://github.com/Azure/batch-scoring-for-dl-models), or you can change
the scale formula directly in the portal after the cluster is created.

For work that doesn't require immediate processing, configure the automatic
scaling formula so the default state (minimum) is a cluster of zero nodes. With
this configuration, the cluster starts with zero nodes and only scales up when
it detects jobs in the queue. If the batch scoring process only happens a few
times a day or less, this setting enables significant cost savings.

Automatic scaling may not be appropriate for batch jobs that happen too close to
each other. The time that it takes for a cluster to spin up and spin down also
incur a cost, so if a batch workload begins only a few minutes after the
previous job ends, it might be more cost effective to keep the cluster running
between jobs. That depends on whether scoring processes are scheduled to run at
a high frequency (every hour, for example), or less frequently (once a month,
for example).

Deployment
----------

The reference implementation of this architecture is available on
[GitHub](https://github.com/Azure/BatchAIAnomalyDetection). Follow the setup
steps there to build a scalable solution for scoring many models in parallel
using Batch AI.
