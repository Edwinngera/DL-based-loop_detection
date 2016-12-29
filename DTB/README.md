Dynamic Training Bench
======================

Dynamic Training Bench (DTB) helps the developers to train and manage ML models built in Tensorflow.

Developers should focus on the model and inputs definitions only: they shouldn't waste their time rewriting training procedures, visualizations, metrics and so on.

DTB uses python 3 and the latest Tensorflow release.

# Standard workflow

1. Define the dataset: implement the `input/Input.py` interface
2. Define the model: implement a model interface

# Define a model

The model definition must be an implementation of one of the available interfaces:

1. [models/Classifier.py](models/Classifier.py)
2. [models/Autoencoder.py](models/Autoencoder.py)

The interface implementation must follow these rules:

1. Model must be placed into the `models/` folder
2. The class name must be equal to the file name.

E.g.: `class LeNet` into [`models/LeNet.py`](models/LeNet.py).

Once created the model file, the methods defined in the interface file must be implemented.

It's recommended, but not strictly required, to use the wrappers built around the Tensorflow methods to define the model: these wrappers creates log and visualizations for you.

## Wrappers

Wrappers are documented and intuitive: you can find it in the [models/utils.py](models/utils.py) file.

## Examples

DTB provides different models that can be used alone or can be used as examples of correct implementations.
Every model in the [models/](models/) folder is a valid example.

In general, the model definition is just the implementation of 2 methods:

1. `get` is which implementing the model itself
2. `loss` in which implementing the loss function

It's strictly required to return the parameters that the method documentation requires to, even if they're unused by your model.

E.g.: even if you never use a `is_training_` boolean placeholder in your model definition, define it and return it anyway.

# Define an input source

DTB provides a single interface to implement to define an input source:

1. [inputs/Input.py](inputs/Input.py)

The recommended steps to follow to implement this interface are:

1. Implement the `maybe_download_and_extract` method: this method must download the dataset and apply the desired transformations to its elements. There are some utility functions defined in the [`inputs/utils.py`](inputs/utils.py) file that can be used.
This method is executed as first operation when the dataset object is created, therefore is recommended to cache the results.
2. Implement the `num_classes` method: this method must return the number of classes of the dataset. If your dataset has no labels, just return 0.
3. Implement the `num_examples(input_type)` method: this method accepts an `InputType` enumeration, defined in `inputs/utils.py`.
This enumeration has 3 possible values: `InputType.train`, `InputType.validation`, `InputType.test`. As obvious, the method must return the number of examples for every possible value of this enumeration.
4. Implement the `inputs` and `distorted_inputs` methods.
The `distorted_inputs` method is the method invoked while training: thus here you can distort the input using data augmentation techniques if required.
The `inputs` method is a general method that should return the real values of the dataset, related to the `InputType` passed, without any augmentation.

*Hint*: both `inputs` and `distorted_inputs` must return a Tensorflow queue of `value, label` pairs.

The better way to understand how to build the input source is to look at the examples in the [inputs/](inputs/) folder.

A small and working example that can be worth looking is the: [inputs/ORLFaces.py](inputs/ORLFaces.py).

# Train

A single model can be trained using various hyper-parameters, such as the learning rate, the weight decay penalty applied, the exponential learning rate decay, the optimizer and its parameters, ...

DTB allows to train a model with different hyper-parameter and automatically it logs every training process allowing the developer to visually compare them.

Moreover, if a training process is interrupted, it automatically resumes it from the last saved training step.

## Example

```
# LeNet: no regularization
python train_classifier.py --model LeNet --dataset MNIST

# LeNet: L2 regularization with value 1e-5
python train_classifier.py --model LeNet --dataset MNIST --l2_penalty 1e-5

# LeNet: L2 regularization with value 1e-2
python train_classifier.py --model LeNet --dataset MNIST --l2_penalty 1e-2

# LeNet: L2 regularization with value 1e-2, initial learning rate of 1e-4
# The default optimization algorithm is MomentumOptimizer, so we can change the momentum value
# The optimizer parameters are passed as a json string
python train_classifier.py --model LeNet --dataset MNIST --l2_penalty 1e-2 \
    --optimizer_args '{"learning_rate": 1e-4, "momentum": 0.5}'

# If, for some reason, we interrupt this training process, rerunning the same command
# will restart the training process from the last saved training step.
# If we want to delete every saved model and log, we can pass the --restart flag
python train_classifier.py --model LeNet --dataset MNIST --l2_penalty \
    --optimizer_args '{"learning_rate": 1e-4, "momentum": 0.5}' --restart
```

The commands above will create 4 different models. Every model has it's own log folder that shares the same root folder.

In particular, in the `log` folder there'll be a `LeNet` folder and within this folder, there'll be other 4 folders, each one with a name that contains the hyper-parameters previously defined.
This allows visualizing in the same graphs, using Tensorboard, the 4 models and easily understand which one performs better.

Depending on the implemented model interface, the training script must change. If an Autoencoder has been implemented, instead of the `train_classifier.py` script, the `train_autoencoder.py` script must be used.

A complete list of the available tunable parameters can be obtained running `python train_classifier.py --help` (`python train_autoencoder.py --help`).

For reference, a part of the output of `python train_classifier.py --help`:

```
usage: train_classifier.py [-h] --model --dataset
  -h, --help            show this help message and exit
  --model {<list of models in the models/ folder, without the .py suffix>}
  --dataset {<list of inputs in the inputs/folder, without the .py suffix}
  --restart
  --lr_decay
  --lr_decay_epochs LR_DECAY_EPOCHS
  --lr_decay_factor LR_DECAY_FACTOR
  --l2_penalty L2_PENALTY
  --optimizer {<list of tensorflow available optimizers>}
  --optimizer_args OPTIMIZER_ARGS
  --batch_size BATCH_SIZE
  --epochs EPOCHS
  --train_device TRAIN_DEVICE
  --eval_device EVAL_DEVICE
  --comment COMMENT
```

# Best models & results

DTB saves for you, in the log folder of every model,  the "best" model with respect to the metric defined in the train file.

For example, for the `LeNet` model created with the first command in the previous script, the following directory structure is created:

```
log/LeNet/
|---MNIST_Momentum_lr=0.01
|-----best
|-----train
|-----validation
```

`train` and `validation` folders contain the logs, used by Tensorboard to display in the same graphs train and validation metrics.

The `best` folder contains one single checkpoint file that is the model with the highest quality obtained during the training phase.


This model is used at the end of the training process to add a line to the `test_results.txt` and `validation_results.txt` files.

The `test_results.txt` file contains the results of the evaluation of the best model on the test set, whilst the `validation_results.txt` contains the same result but for the validation set.

Moreover, is possible to run the evaluation of any checkpoint file (in the `log/<MODEL>` folder or in the `log/<MODEL>/best` folder) using the `evaluate_classifier.py` (`evaluate_autoencoder.py`) script.

For example:

```
# Evaluate the validation accuracy
python evaluate_classifier.py  \
           --model LeNet \
           --dataset MNIST \
           --checkpoint_dir log/LeNet/MNIST_Momentum_lr\=0.01/
# outputs something like: validation accuracy = 0.993

# Evaluate the test accuracy
python evaluate_classifier.py  \
           --model LeNet \
           --dataset MNIST \
           --checkpoint_dir log/LeNet/MNIST_Momentum_lr\=0.01/ \
           --test
# outputs something like: test accuracy = 0.993
```

# Data visualization

Running tensorboard

```
tensorboard --logdir log/<MODEL>
```

It's possible to visualize the trend of the loss, the validation measures, the input values and so on.
To see some of the produced output, have a look at the implementation of the Convolutional Autoencoder, described here: https://pgaleone.eu/neural-networks/deep-learning/2016/12/13/convolutional-autoencoders-in-tensorflow/#visualization
