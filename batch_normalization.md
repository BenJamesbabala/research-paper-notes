## Batch Normalization

**Authors**: Sergey Ioffe and Christian Szegedy, "Batch Normalization: Accelerating Deep Network Training by Reducing Internal Covariate Shift", ICML 2015.

**One-liner**: The problem of *internal covariate shift* is addressed by adding mini-batch normalization layers to neural network model architectures.

### Problem Setting

Network models tend to train faster when their input data is whitened, i.e. features are uncorrelated and have zero-mean and unit variance. For example, feature standardization and ZCA whitening are some common preprocessing steps when working with images.

The problem is that due to the hierarchical nature of neural networks, even though the first layer sees data that follows a nice distribution, deeper layers receive much different ditributions because the weights of previous layers are being continuously updated and shifted during training. This phenomenon is what the authors define as *internal covariate shift* (**ICV**) or the change in the distribution of network activations due to the change in network parameters during training.

Here is a plot generated by Andrej Karpathy's code snippet in [CS231n](cs231n.github.io). It shows the evolution of the mean and standard deviations of the inputs across 10 `tanh` nonlinearities.

<p align="center">
 <img src="/img/batch_normalization/distribution.png" width="460px">
</p>

Notice that since `tanh` is symmetric, the mean stays approximately the same (~ 0), however the standard deviation plummets down to 0. This is rather bad for deep networks because backpropagation will compute very small gradients (they are proportional to the value of the weights) effectively leading to *vanishing gradients*.

In short, ICV slows down training because it requires lower learning rates, careful weight initialization, and makes it notoriously hard to train models with saturating nonlinearities such as `tanh`.

### Meat of the paper

To combat ICV, the authors propose to insert differentiable batch normalization layers into the network. This special layer benefits from the following 2 properties:

- Instead of using the entire training set to perform input normalization, an operation that is costly and not everywhere differentiable (bad for backprop), it instead normalizes each scalar feature **independently**.
- It uses the mini-batches to estimate the mean and variances for each activation rather than the whole training set.

This is the pseudocode implementation for training written in the paper.

<p align="center">
 <img src="/img/batch_normalization/alg1.png" width="280px">
</p>

Basically, the batch normalization layer uses a minibatch of data to estimate the mean and standard deviation across every single feature dimension. These parameters are then used to zero-center and normalize the features of the minibatch. Also, notice the last line of the algorithm. The batch normalization layer includes learnable parameters `gamma` and `beta` which shift and scale the normalized values. **Why is this particulary useful?** 

Well, we would like for our NN to choose the level of saturation of its activations. Normalizing the inputs would restrict them to a certain region of the nonlinearity, something that could cause the activations to die or explode during training. Backprop will finetune `gamma` and `beta` such that our network can choose to take full advantage of BN, or completely ignore it (i.e. identity mapping can be recovered by setting `gamma = std`, `beta = mean`).

### Training vs. Testing

During testing, the BN layer implementation is different. In fact, the authors assert that the output should only depend on the input, deterministically, rather than on the mini-batch statistics.
 
<p align="center">
 <img src="/img/batch_normalization/alg2.png" alt="Drawing" width="240px">
</p>

**What does this mean exactly?**

Well, when we're predicting an output, the layer is supposed to only see one data point at a time. This means that computing the mean and variance along a whole batch is technically *cheating*. Instead, a running average of these means and standard deviations is kept during training, and at test time these running averages are used to center and normalize features, effectively leading to *unbiased* population estimates. 

###Where to Insert Batch Norm Layers?

Usually inserted after FC/Conv layers and before the nonlinearity.

### Advantages of Batch Normalization

- improves the gradient flow
- more tolerant to saturating nonlinearities such as `tanh`
- reduces the dependence of gradients on the scale of the parameters or their initialization
- allows higher learning rates
- acts as a form of regularization
- all in all accelerates neural network training
