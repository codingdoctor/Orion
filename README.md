<p align="center">
<img width=30% src="https://dai.lids.mit.edu/wp-content/uploads/2018/08/orion.png" alt=“Orion” />
</p>

<p align="center">
<i>Orion is a machine learning library built for telemetry data generated by satellites.</I>
</p>

<p align="center">
<i>A project from Data to AI Lab at MIT.</I>
</p>

# Orion

Orion is a machine learning library built for telemetry data generated by Satellites.

With this data, our interest is to develop techniques to:

* identify rare patterns and flag them for expert review.
* predict outcomes ahead of time.

The library makes use of a number of *automated machine learning* tools developed under
["The human data interaction project"](https://github.com/HDI-Project) within the
[Data to AI Lab at MIT](https://dai.lids.mit.edu/).

With the ready availability of *automated machine learning* tools, the focus is on:

* domain expert interaction with the machine learning system;
* learning from minimal labels;
* explainability of model outputs;
* model audit;
* scalability;

## License

- MIT license


## Table of Contents

* [Data Format](#data-format)
   * [Input](#input)
   * [Output](#output)
* [Dataset we use in this library](#dataset-we-use-in-this-library)
* [Orion Pipelines](#orion-pipelines)
   * [Current Available Pipelines](#current-available-pipelines)
   * [Leaderboard](#leaderboard)
* [Getting Started](#getting-started)
   * [Requirements](#requirements)
   * [Install](#install)
   * [Quickstart: Run a Pipeline on the Demo Dataset](#quickstart-run-a-pipeline-on-the-demo-dataset)
* [Database](#database)
* [Docker Usage](#docker-usage)
   * [Requirements](#requirements-1)
   * [Building the Orion Docker Image](#building-the-orion-docker-image)
   * [Distributing the Orion Docker Image](#distributing-the-orion-docker-image)
   * [Running the orion-jupyter image](#running-the-orion-jupyter-image)

## Data Format

### Input

**Orion Pipelines** work on time Series that are provided as a single table of telemetry
observations with two columns:

* `timestamp`: an INTEGER or FLOAT column with the time of the observation in
  [Unix Time Format](https://en.wikipedia.org/wiki/Unix_time)
* `value`: an INTEGER or FLOAT column with the observed value at the indicated timestamp

This is an example of such table:

|  timestamp |     value |
|------------|-----------|
| 1222819200 | -0.366358 |
| 1222840800 | -0.394107 |
| 1222862400 |  0.403624 |
| 1222884000 | -0.362759 |
| 1222905600 | -0.370746 |

### Output

The output of the **Orion Pipelines** is another table that contains the detected anomalous
intervals and that has at least two columns:

* `start`: timestamp where the anomalous interval starts
* `end`: timestamp where the anomalous interval ends

Optionally, a third column called `score` can be included with a value that represents the
severity of the detected anomaly.

An example of such a table is:

|      start |        end |    score |
|------------|------------|----------|
| 1222970400 | 1222992000 | 0.572643 |
| 1223013600 | 1223035200 | 0.572643 |

## Dataset we use in this library

For development, evaluation of pipelines, we include a dataset which includes several satellite
telemetry signals already formatted as expected by the Orion Pipelines.

This formatted dataset can be browsed and downloaded directly from the
[d3-ai-orion AWS S3 Bucket](https://d3-ai-orion.s3.amazonaws.com/index.html).

This dataset is adapted from the one used for the experiments in the
[Detecting Spacecraft Anomalies Using LSTMs and Nonparametric Dynamic Thresholding paper](https://arxiv.org/abs/1802.04431).
[Original source data is available for download here](https://s3-us-west-2.amazonaws.com/telemanom/data.zip).
We thank NASA for making this data available for public use.

## Orion Pipelines

The main component in the Orion project are the **Orion Pipelines**, which consist of
[MLBlocks Pipelines](https://hdi-project.github.io/MLBlocks/advanced_usage/pipelines.html)
specialized in detecting anomalies in time series.

As ``MLPipeline`` instances, **Orion Pipelines**:

* consist of a list of one or more [MLPrimitives](https://hdi-project.github.io/MLPrimitives/)
* can be *fitted* on some data and later on used to *predict* anomalies on more data
* can be *scored* by comparing their predictions with some known anomalies
* have *hyperparameters* that can be *tuned* to improve their anomaly detection performance
* can be stored as a JSON file that includes all the primitives that compose them, as well as
  other required configuration options.

### Current Available Pipelines

In the **Orion Project**, the pipelines are included as **JSON** files, which can be found
inside the [orion/pipelines](orion/pipelines) folder.

This is the list of pipelines available so far, which will grow over time:

| name | location | description |
|------|----------|-------------|
| Dummy | [orion/pipelines/dummy.json](orion/pipelines/dummy.json) | Dummy Pipeline to showcase the input and output format and the usage of sample primitives |
| LSTM Dynamic Thresholding | [orion/pipelines/lstm_dynamic_threshold.json](orion/pipelines/lstm_dynamic_threshold.json) | LSTM Based pipeline inspired by the [Detecting Spacecraft Anomalies Using LSTMs and Nonparametric Dynamic Thresholding paper](https://arxiv.org/abs/1802.04431) |
| Mean 24h LSTM | [orion/pipelines/mean_24h_lstm.json](orion/pipelines/mean_24h_lstm.json) | LSTM Based pipeline with 24h mean aggregation preprocessing |
| Median 24h LSTM | [orion/pipelines/median_24h_lstm.json](orion/pipelines/median_24h_lstm.json) | LSTM Based pipeline with 24h median aggregation preprocessing |
| Sum 24h LSTM | [orion/pipelines/sum_24h_lstm.json](orion/pipelines/sum_24h_lstm.json) | LSTM Based pipeline with 24h sum aggregation preprocessing |
| Skew 24h LSTM | [orion/pipelines/skew_24h_lstm.json](orion/pipelines/skew_24h_lstm.json) | LSTM Based pipeline with 24h skew aggregation preprocessing |

### Leaderboard

In this repository we maintain this up-to-date leaderboard with the current scoring of the
pipelines according to the scoring procedure explained in the [SCORING.md](SCORING.md) document.

| pipeline                  |   rank |   accuracy |       f1 |   precision |   recall |
|---------------------------|--------|------------|----------|-------------|----------|
| LSTM Dynamic Thresholding |      1 |   0.963781 | 0.216231 |   0.666667  | 0.130074 |
| Median 24h LSTM           |      2 |   0.960239 | - Na -   |   - Na -    | - Na -   |
| Mean 24h LSTM             |      3 |   0.959948 | - Na -   |   - Na -    | - Na -   |
| Sum 24h LSTM              |      4 |   0.959275 | - Na -   |   - Na -    | - Na -   |
| Skew 24h LSTM             |      5 |   0.956565 | - Na -   |   - Na -    | - Na -   |
| Dummy                     |      6 |   0.911606 | 0.094981 |   0.0708985 | 0.156801 |


## Getting Started

### Requirements

#### Python

**Orion** has been developed and runs on [Python 3.6](https://www.python.org/downloads/release/python-360/).

Also, although it is not strictly required, the usage of a [virtualenv](https://virtualenv.pypa.io/en/latest/)
is highly recommended in order to avoid interfering with other software installed in the system
where you are trying to run **Orion**.

#### MongoDB

In order to be fully operational, **Orion** requires having access to a
[MongoDB](https://www.mongodb.com/) database running version **3.6** or higher.

### Install

Since **Orion** is a private project, the only way to install it is by cloning or downloading
its sources from its [GitHub repository](https://github.com/D3-AI/Orion):

```
git clone git@github.com:D3-AI/Orion.git
```

After cloning the repository and creating and activating a virtualenv, you can install
the project with this command:

```
make install
```

For development, use the following command instead, which will install some additional
dependencies for code linting and testing

```
make install-develop
```

#### Docker

Even thought it's not mandatory to use it, **Orion** comes with the possibility to be
distributed and run as a docker image, making its usage in offline systems easier.

For more details please head to the [Docker Usage](#docker-usage) section below.

### Quickstart: Run a Pipeline on the Demo Dataset

In the following steps we will show a short guide about how to run one of the **Orion Pipelines**
on one of the signals from the **Demo Dataset**.

**NOTE**: All the examples of this tutorial are run in an [IPython Shell](https://ipython.org/),
which you can install by running the following commands inside your *virtualenv*:

```
pip install ipython
ipython
```

#### 1. Load the data

In the first step we will load the **S-1** signal from the **Demo Dataset**,

To do so, we need to import the `orion.data.load_signal` function and call it passing
the `'S-1'` name.

```
In [1]: from orion.data import load_signal

In [2]: data = load_signal('S-1')

In [3]: data.head()
Out[3]:
    timestamp     value
0  1222819200 -0.366359
1  1222840800 -0.394108
2  1222862400  0.403625
3  1222884000 -0.362759
4  1222905600 -0.370746
```

#### 2. Detect anomalies using a pipeline

Once we have the data, let us try to use the LSTM pipeline to analyze it and search for anomalies.

In order to do so, we will have import the `orion.analysis.analyze` function and pass it
the loaded data and the path to the pipeline JSON that we want to use:

```
In [4]: from orion.analysis import analyze

In [5]: pipeline_path = 'orion/pipelines/lstm_dynamic_threshold.json'

In [6]: anomalies = analyze(pipeline_path, data)
Using TensorFlow backend.
Epoch 1/1
9899/9899 [==============================] - 55s 6ms/step - loss: 0.0559 - mean_squared_error: 0.0559
```

**NOTE:** Depending on your system and the exact versions that you might have installed
some *WARNINGS* may be printed. These can be safely ignored as they do not interfere
with the proper behavior of the pipeline.

The output of the previous command will be a ``pandas.DataFrame`` containing a table in the
Output format described above:

```
In [7]: anomalies
Out[7]:
        start         end     score
0  1398060000  1399442400  0.168381
```

#### 3. Evaluate performance

In this next step we will load some already known anomalous intervals and evaluate how
good our anomaly detection was by comparing those with our detected intervals.

For this, we will first load the known anomalies for the signal that we are using:

```
In [8]: from orion.data import load_anomalies

In [9]: truth = load_anomalies('S-1')

In [10]: truth
Out[10]:
        start         end
0  1392768000  1402423200
```

Afterwards, we pass the ground truth, the detected anomalies and the original data
to the `orion.metrics.accuracy_score` and `orion.metrics.f1_score` functions in order
to compute a score that indicates how good our anomaly detection was:

```
In [11]: from orion.metrics import accuracy_score, f1_score

In [11]: accuracy_score(truth, anomalies, data)
Out[11]: 0.956346078044935

In [12]: f1_score(truth, anomalies, data)
Out[12]: 0.173492893794023
```


## Database

**Orion** comes ready to use a MongoDB Database to easily register and explore:

* Multiple Datasets based on signals from one or more satellites.
* Multiple Pipelines, including historical Pipeline versions.
* Pipeline executions on the registered Datasets, including any environment details required to
  later on reproduce the results.
* Pipeline execution results and detected events.
* Comments about the detected events.

This, among other things, allows:

* Providing visibility about the system usage.
* Keeping track of the evolution of the registered pipelines and their performance over multiple datasets.
* Visualizing and browsing the detected events by the pipelines using a web application.
* Collecting comments from multiple domain experts about the detected events to be able to later
  on curate the pipelines based on their knowledge.
* Reproducing previous executions in identical environments to replicate the obtained results.
* Detecting and keeping a history of system failures for later investigation.

The complete **Database schema and usage instructions** can be found in the
[DATABASE.md](DATABASE.md) file


## Docker Usage

**Orion** comes configured and ready to be distributed and run as a docker image which starts
a jupyter notebook already configured to use orion, with all the required dependencies already
installed.

### Requirements

The only requirement in order to run the Orion Docker image is to have Docker installed and
that the user has enough permissions to run it.

Installation instructions for any possible system compatible can be found [here](https://docs.docker.com/install/)

Additionally, the system that builds the Orion Docker image will also need to have a working
internet connection that allows downloading the base image and the additional python depenedencies.

### Building the Orion Docker Image

After having cloned the **Orion** repository, all you have to do in order to build the Orion Docker
Image is running this command:

```
make docker-jupyter-build
```

After a few minutes, the new image, called `orion-jupyter`, will have been built into the system
and will be ready to be used or distributed.

### Distributing the Orion Docker Image

Once the `orion-jupyter` image is built, it can be distributed in several ways.

#### Distributing using a Docker registry

The simplest way to distribute the recently created image is [using a registry](https://docs.docker.com/registry/).

In order to do so, we will need to have write access to a public or private registry (remember to
[login](https://docs.docker.com/engine/reference/commandline/login/)!) and execute these commands:

```
docker tag orion-jupyter:latest your-registry-name:some-tag
docker push your-registry-name:some-tag
```

Afterwards, in the receiving machine:

```
docker pull your-registry-name:some-tag
docker tag your-registry-name:some-tag orion-jupyter:latest
```

#### Distributing as a file

If the distribution of the image has to be done offline for any reason, it can be achieved
using the following command.

In the system that already has the image:

```
docker save --output orion-jupyter.tar orion-jupyter
```

Then copy over the file `orion-jupyter.tar` to the new system and there, run:

```
docker load --input orion-jupyter.tar
```

After these commands, the `orion-jupyter` image should be available and ready to be used in the
new system.


### Running the orion-jupyter image

Once the `orion-jupyter` image has been built, pulled or loaded, it is ready to be run.

This can be done in two ways:

#### Running orion-jupyter with the code

If the Orion source code is available in the system, running the image is as simple as running
this command from within the root of the project:

```
make docker-jupyter-run
```

This will start a jupyter notebook using the docker image, which you can access by pointing your
browser at http://127.0.0.1:8888

In this case, the local version of the project will also mounted within the Docker container,
which means that any changes that you do in your local code will immediately be available
within your notebooks, and that any notebook that you create within jupyter will also show
up in your `notebooks` folder!

#### Running orion-jupyter without the orion code

If the Orion source code is not available in the system and only the Docker Image is, you can
still run the image by using this command:

```
docker run -ti -p8888:8888 orion-jupyter
```

In this case, the code changes and the notebooks that you create within jupyter will stay
inside the container and you will only be able to access and download them through the
jupyter interface.
