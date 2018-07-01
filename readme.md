# Table of Content

* Few Shot Learning Literature
  * General Setup and Datasets
  * Siamese Networks for One Shot Learning 
  * Matching Networks 
  * Meta-Agnostic Model Learning 
  * Triplet Networks 
  * Prototypical Networks 
  * Activations to Parameters 
  * Weight Imprinting 
* Human Robot Interaction Setting 
  * Differences to Few Shot Literature 
  * KUKA Innovation Challenge 

# Few Sot Learning using HRI

Few Shot Learning, the ability to learn from few labeled samples, is a vital step in robot manipulation. In order for robots to operate in dynamic and unstructured environments, they need to learn novel objects on the fly from few samples. The current object recognition methods using convolutional networks are based on supervised learning with large-scale datasets such as ImageNet, with hundreds or thousands labeled examples. However, even with large-scale datasets they remain limited in multiple aspects, not all objects in our lives are within the 1000 labels provided in ImageNet. 

As humans we can hold the object and check it from different viewpoints and try to interact with it to learn more about the object. Thus the robot should be able to teach itself from the few samples for the different object viewpoints. If we are aiming as well at human centered artificial intelligence, a natural step is to teach robots about their environment through human robot interaction. A human teacher can show the object with different poses and verbally instruct the robot on what it is and how it can be used. 

<div align="center"><img src="objects.png" class="img-responsive" alt=""> </div>

## Few Shot Learning Literature:
What motivated me to write on this topic was working on the KUKA innovation challenge, I was part of team Alberta that were in the 5 finalists. It turned out to be an exciting way of understanding the problem. While surveying and reading papers can give you the understanding of what the literature are working on. However, some new problems from working on the demo popped up that we realized are still lacking from the literature and my intention is to share these. 


### General Setup and Datasets:
The few shot learning is formulated as a **m shot n way** classification problem, where **m is the number of labeled samples per class**, and **n is the number of classes** to classify among. Two main datasets are used in the literature:
* Omniglot Dataset [1], the few-shot version of MNIST. It is a character recognition dataset which contains 50 alphabets, each alphabet has around 15 to 40 characters, and each character is produced by 20 drawers. 
* Mini ImageNet dataset [2] on the other hand is a more realistic setting. 100 random classes from ImageNet are chose, with 80 for training and 20 for testing.  
* In a newer work in CVPR'18 [3] instead of evaluating on the few-shot set solely, evaluating on both few-shot set and the large-scale set data on the whole ImageNet data with the 1000-way accuracy was reported.

<div align="center"><img src="omniglot.png" class="img-responsive" alt=""> </div>

### Siamese and Triplet Networks
Metric learning methods have the advantage that they rapidly learn novel concepts without retraining. 

#### Cross Entropy Loss
One of the earliest attempts that was designed mainly for few shot learning using siamese networks was by Koch [6]. It formulated the few shot learning problem as a **verification task**. A siamese network consists of two twin networks with shared weights, and a weighted L1 distance function is learned. This is done by applying L1 distance on the output embeddings then adding one fully connected layer to learn the weighted distance. The loss function used in the paper is a regularized cross entropy, where the main aim is to drive similar samples to predict 1, and 0 otherwise.

<div><img src="ce.png" width="40%" class="img-responsive" alt=""> </div>

#### Contrastive Loss
One approach is to learn a mapping from inputs to vectors in an embedding space where the inputs of the same class are closer than those of different classes. Once the mapping is learned, at test time a nearest neighbors method can be used for classification for new classes that are unseen. A siamese network is trained with the output features fed to a Contrastive Loss [4]:

<div><img src="cl.png" width="50%" class="img-responsive" alt=""> </div>

Y label is 0 for similar class samples, and 1 for dissimilar. D_w is the distance function that is euclidean distance. So the loss will decrease the distance D when the samples are from the same class, on the other hand when they are dissimilar it will try to increase D with a certain margin m. The margin purpose is to neglect samples that have larger distance than m, since we only want to focus on dissimilar samples that appear to be close.

#### Triplet Loss
A better extension on the contrastive loss idea is to use a triplet network with triplet loss [5]. The triplet network inspiring from the siamese networks will have three copies of the network with shared weights. The input contains an anchor sample, a positive sample and a negative sample. The three output embeddings are then fed to the triplet loss [5]:

 <div><img src="triplet.png" width="50%" class="img-responsive" alt=""> </div>

X is the anchor sample, X+ is the positive sample, X- is the negative sample, D_w is the distance function and m is the margin. It is basically decreasing the distance between the anchor and its positive sample while at the same time increasing its distance to the negative sample. Why this is better than Contrastive loss, cause ...

#### Summary
To sum it up there are three things to think of when desiging your method :

<div align="center"><img src="metric_learning.png" width="50%" class="img-responsive" alt=""> </div>

* The base network architecture used in the siamese or triplet network.
* The distance function applied on the output embeddings:
  * L2 distance (Euclidean)
  * L1 distance
  * Weighted L1 distance
  * Cosine distance
* The loss function:
  * Contrastive Loss
  * Triplet Loss
  * Cross Entropy

[Other useful resources](https://hackernoon.com/one-shot-learning-with-siamese-networks-in-pytorch-8ddaab10340e).

### View-Manifold Learning

The previous approaches does not address the different viewpoints that can be available for the novel objects being learned. However in HRI setting you have the different viewpoints for the learned objects available. A very similar approach to the above but is specificaly designed to handle this [7]. They design a triplet network, with a cosine distance function between X1 and X2 vectors as:

<div><img src="cos.png" width="50%" class="img-responsive" alt=""> </div>

A triplet loss similar to the above but using cosine distance is used. Their experiments are done on 3D Models from ShapeNet dataset to incorporate different viewpoints for the learned objects. 

<div align="center"><img src="view_manifold.png" width="70%" class="img-responsive" alt=""> </div>

### Matching Networks

On the same line of metric learning based methods, matching networks tries to learn an end-to-end differentiable nearest neighbour [8]. It is based on this attention kernel:

<div><img src="attkernel.png" width="40%" class="img-responsive" alt=""> </div>

Where the possible class labels y are weighted with the a, which determines how much two samples x, x^hat are close. This a is computed as the softmax of the cosine distance between the two samples.

<div><img src="attention.png" width="40%" class="img-responsive" alt=""> </div>

f and g are the embeddings of both the test and training samples respectively. The training samples embedding is based on a bidirectional LSTM that learns the embedding in the support set context. The support set is the set of few labeled samples. While f is an LSTM with attention. 

<div align="center"><img src="matchnets.png" width="50%" class="img-responsive" alt=""> </div>

[Other useful resouces](https://github.com/karpathy/paper-notes/blob/master/matching_networks.md).

### MAML

Another direction in few shot learning that is away from metric based learning methods is meta learning. The MAML method [9] creates this model agnostic method, that has a meta objective being optimized over all tasks. The algorithm from the paper:

<div><img src="maml.png" width="50%" class="img-responsive" alt=""> </div>

For each sampled data points D we optimize using stochastic gradient descent and update the parameters based on this. But then a meta update is computed that sums the gradients over all tasks using the updated parameters \theta_i'. This is used to make a meta update to the parameters.

### Activations to Parameters

This year CVPR had an interesting paper on few shsot learning that is showing really good results with a very intuitive idea. The method is based on learning a mapping between activations and parameters/weights. This mapping can then be used when we have new classes that have few labeled samples to get their corresponding weights to be used in the classification.


### Imprinted Weights

## HRI Setting:
The fundamental differences between human robot interaction and the current few shot learning setting are: 
1. the abundance of temporal information for the different poses of the object. 
2. the hierarchy of category, different instances/classes within the same category, and different poses. 
3. The open set nature of the problem, which requires the identification of unknown objects. 
4. Different challenges introduced by the cluttered background, the different rigid and non-rigid transformations, occlusions and illumination changes. 
5. the continual learning of objects.

[1] Lake, Brenden, et al. "One shot learning of simple visual concepts." Proceedings of the Annual Meeting of the Cognitive Science Society. Vol. 33. No. 33. 2011.

[2] Vinyals, Oriol, et al. "Matching networks for one shot learning." Advances in Neural Information Processing Systems. 2016.

[3] Qiao, Siyuan, et al. "Few-shot image recognition by predicting parameters from activations." CoRR, abs/1706.03466 1 (2017).

[4] Hadsell, Raia, Sumit Chopra, and Yann LeCun. "Dimensionality reduction by learning an invariant mapping." null. IEEE, 2006.

[5] Hoffer, Elad, and Nir Ailon. "Deep metric learning using triplet network." International Workshop on Similarity-Based Pattern Recognition. Springer, Cham, 2015.

[6] Koch, Gregory, Richard Zemel, and Ruslan Salakhutdinov. "Siamese neural networks for one-shot image recognition." ICML Deep Learning Workshop. Vol. 2. 2015.

[7] Lin, Xingyu, et al. "Transfer of view-manifold learning to similarity perception of novel objects." arXiv preprint arXiv:1704.00033 (2017).

[8] Vinyals, Oriol, et al. "Matching networks for one shot learning." Advances in Neural Information Processing Systems. 2016.

[9] Finn, Chelsea, Pieter Abbeel, and Sergey Levine. "Model-agnostic meta-learning for fast adaptation of deep networks." arXiv preprint arXiv:1703.03400 (2017).
