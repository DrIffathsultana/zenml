---
jupytext:
  cell_metadata_filter: -all
  formats: md:myst
  text_representation:
    extension: .md
    format_name: myst
    format_version: 0.12
    jupytext_version: 1.9.1
kernelspec:
  display_name: Python 3
  language: python
  name: python3
---

# Quickstart
Get up and running in (almost) 3 steps.
Let’s get you started with a simple pipeline. Please make sure to also check out the [advanced concepts.](core-concepts.md) This quickstart uses some built-ins and a very simple model.

```{hint}
If you are here just to see the code, you can find it on [GitHub](https://github.com/maiot-io/zenml#quickstart).
```

## **For visual learners**

If you don't feel like reading right now, please watch this video for a visual explanation of the quickstart:

<iframe width="560" height="315" src="https://www.youtube.com/embed/Stg5rA_0oa8" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

### **Step 0: Installation**

ZenML is available for easy installation into your environment via PyPI:

```{code-cell}
%%bash
pip install zenml
```

Alternatively, if you’re feeling brave, feel free to install the bleeding edge: **NOTE:** Do so on your own risk, no guarantees given!

```{code-cell}
%%bash
pip install git+https://github.com/maiot-io/zenml.git@main --upgrade
```

### Step 1: Initialize a ZenML repo from within a git repo

```{code-cell}
zenml init
```

### **Step 2: Assemble, run and evaluate your pipeline locally**

```{code-cell}
from zenml.core.datasources.csv_datasource import CSVDatasource
from zenml.core.pipelines.training_pipeline import TrainingPipeline
from zenml.core.steps.evaluator.tfma_evaluator import TFMAEvaluator
from zenml.core.steps.split.random_split import RandomSplit
from zenml.core.steps.preprocesser.standard_preprocesser.standard_preprocesser import StandardPreprocesser
from zenml.core.steps.trainer.tensorflow_trainers.tf_ff_trainer import FeedForwardTrainer

training_pipeline = TrainingPipeline(name='Quickstart')

# Add a datasource. This will automatically track and version it.
ds = CSVDatasource(name='Pima Indians Diabetes Dataset', 
                   path='gs://zenml_quickstart/diabetes.csv')
training_pipeline.add_datasource(ds)

# Add a random 70/30 train-eval split
training_pipeline.add_split(RandomSplit(split_map={'train': 0.7, 'eval': 0.3}))

# StandardPreprocesser() has sane defaults for normal preprocessing methods
training_pipeline.add_preprocesser(
    StandardPreprocesser(
        features=['times_pregnant', 'pgc', 'dbp', 'tst', 'insulin', 'bmi',
                  'pedigree', 'age'],
        labels=['has_diabetes'],
        overwrite={'has_diabetes': {
            'transform': [{'method': 'no_transform', 'parameters': {}}]}}
    ))

# Add a trainer
training_pipeline.add_trainer(FeedForwardTrainer(
    loss='binary_crossentropy',
    last_activation='sigmoid',
    output_units=1,
    metrics=['accuracy'],
    epochs=20))


# Add an evaluator
training_pipeline.add_evaluator(
    TFMAEvaluator(slices=[['has_diabetes']],
                  metrics={'has_diabetes': ['binary_crossentropy',
                                            'binary_accuracy']}))

# Run the pipeline locally
training_pipeline.run()
```

### **Step 3: Leverage powerful integrations**

```{code-cell}
# See schema of data
training_pipeline.view_schema()

# See statistics of train and eval
training_pipeline.view_statistics()

# Creates a notebook for evaluation
training_pipeline.evaluate()
```

Of course, each of these steps can be [extended quite easily](../steps/what-is-a-step.md) to accommodate more complex 
scenarios and use-cases. There is a steadily-growing number of integrations available, for example, 
[Google Dataflow for distributed preprocessing](../backends/what-is-a-backend.md) 
or Google Cloud AI Platform as a [training(../backends/training-backends.md)  backend].

## What to do next?
* Read about [core concepts](core-concepts.md) of ZenML.
* [Convert your legacy code-base](organizing-zenml.md) to ZenML pipelines.
* Understand deeper what makes a [ZenML Repository](../repository/what-is-a-repository.md).
* See what ZenML has to offer with standard powerful abstractions like [Pipelines](../pipelines/what-is-a-pipeline.md), 
[Steps](../steps/what-is-a-step.md), [Datasources](../datasources/what-is-a-datasource.md) and [Backends](../backends/what-is-a-backend.md). 
  If the standard ones don't fit your needs, you can also [create custom logic](creating-custom-logic.md) with ZenML.
