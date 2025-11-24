***Molecular Generation using Graph Neural Networks***

**Background**

Recently, I've become fascinated with graph networks and understanding their inner workings. Following this interest, I'm attempting to create something similar to Graph Network Dojo, which will contain implementations of various graph layers and models. The purpose of this project is for me to deepen my understanding of GNNs.

Graph Neural Networks (GNNs) represent a novel type of neural network architecture that operates on a graph-in, graph-out principle. The input to the model is a graph, which is processed and transformed into an output graph. This output maintains the same structure as the input, but with modified node and edge embeddings. These transformed graphs can then be utilized for various downstream tasks such as link prediction, node classification, or edge classification.

While there are many variants, modern GNNs generally fall into three categories:
1. Message Passing: Nodes exchange information (messages) with neighbors iteratively.
2. Convolution: Generalizing the convolution operator from image processing to graph structures (e.g., GCNs).
3. Attention: Using mechanisms like Graph Attention Networks (GAT) to weigh the importance of different neighbors dynamically.

For better understanding, articles on Distill are highly recommended.


**Introduction**

In my quest to learn more about GNNs, I've been exploring various applications to implement. Forst implementation was to solve (or atleast try!) the HAMD, the Hamiltonian cycle decision problem. The results were pretty satisfacory atleast for graphs with N <= 30, for N > 30, creating a dataset was the difficult part. After HAMD i began to search for papers to implment, I encountered the concept of molecule generation. After looking at different approaches, I was decided to test out  Xavier Bresson's two-step decoder method and chose to implement it.

Throughout this process, I found Chaitanya Joshi's blogs and projects to be incredibly helpful resources. However, despite this guidance, I still encountered numerous challenges while attempting to implement the model. This project has been be an excellent learning experience, allowing me deepen my understanding of both GNNs.

**GVAEs: Model Architecture**


The outline of the approach  is to map the input graph into a latent space using an encoder, and then sample from the space to reconstruct the molecules using a decoder, this is the basic approach in VAEs.

1. The Encoder:
    A generic encoder's task is to compress  high dimensional input data into low dimensional representation called latent space (often represented by z). The variational part of VAE, comes from the fact that instead of generating a low dimensional feature vector z, we generate 2 quantitites mu and sigma, which represnet the mean and variance of the low dimensional distribution we want to mimic.

    The goal is to make the low dimensional distribution, a standard gaussian distribution, so the generated mu and sigma for an ideal encoder should be 0, 1. This is achieved through reparametrization. For this GVAE, the encoder is similar to a generic encoder, a GNN is used as encoder in GVAE. 

2. 2 Step Decoder:

    In a generic decoder the low dimensional input is upsampled and the output follows the same structure as the input to the encoder, so in this case it should be a graph. But for graphs fixing the number of nodes is important, so the forst step of the decoder is node predictin, and the sencond step is edge prediction.

**The Sparsity Problem and Edge Loss**

A major challenge in molecular generation is the "Edge Loss" function. In a molecular graph, the number of actual bonds is very less compared to the number of possible bonds (a fully connected graph). If you train a naive model, it achieves high accuracy simply by predicting "No Bond" for every edge.

After testing reading about different approaches, weighted cross-entropy loss seemed appropriate. In this, weights are applied to each class based on the inverse of the frequency of that class in the dataset, so class that appeared less would have high weight, so a lower weight to no bond class and the model is forced to learn the structure of the graph rather than predict "no bond" all the time. Additionally, I would it challenging to handle proper indexing of nodes within the mini-batch for libraries like pytorch geometric, as in PyG, a minibatch is treated as a single graph o disconnected components.

**Balancing the KL Divergence**
During model training, the KL divergence term dropped to low values very early, this made the model unstable and it didn't improve that much afterwards.

To prevent this, I tried KL Annealing. In this a weight is given to the KL term that starts at 0 and gradually increases to 1 during training. This allows the model to first learn how to reconstruct the molecule (reducing reconstruction loss) before being trying to fit the latent space into a gaussian distribution.

**Depth and Residual Connections**

During experiments, I observed that deeper GNNs often struggle to propagate gradients effectively. Residual connections were very imporatant to reduce this effect. This allows gradients to flow backwards more efficiently, which stabilizes the training and results in better feature extraction.

**Future Directions in Generative Graphs**
Implementing a GVAE for molecular generation highlights both the power and the limitations of current methods.
* Adjacency Matrix Complexity: Reconstructing a dense adjacency matrix is computationally expensive, For larger molecules, this becomes a problem.
* Output molecule strucure: The model might predict a carbon atom with 6 bonds, which is not possible. Although the paper suggests a method for this, I didn't implement it.

* Looking forward, the field is moving toward autoregressive models (building the graph node-by-node) and reinforcement learning approaches. These methods can explicitly enforce chemical strcutre at each step of the generation process, potentially solving solving the strcture issues.