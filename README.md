# MNIST_Model_Collapse
Demonstrating model collapse on image data using the MNIST and EMNIST datasets

Model collapse is the term given to failure of a model to correctly generate or discriminate across a real world distribution when it is repeatedly fed its own generations as training data. Given the importance of LLMs in frontier AI development and the now ubiquitous presence of LLM outputs in internet text archives, this phenomenon could slow or otherwise severely disrupt the creation of new foundation models over the next couple of years. (For examples, see https://arxiv.org/pdf/2402.07712v2. For potential resolutions, see https://arxiv.org/pdf/2404.01413)

A lot of work has been done on model collapse applied to language models. By feeding such models their output as further training data, their generations degrade rapidly. They often wind up stuck on a phrase or word which they repeat endlessly, completely displacing any useful output. The bias inherent in any model design becomes the principle driver of generations, leading to a failure of the model to accurately sample from the distribution on which they were trained (https://arxiv.org/abs/2307.01850). The model becomes increasingly certain of its own predictions which are continually reinforced in its new training data, leading to forgetting of the original distribution and higher confidence in its current outputs https://arxiv.org/abs/2305.17493. 

For a good introduction to model collapse, see https://www.nature.com/articles/s41586-024-07566-y. Theoretical work in https://arxiv.org/abs/2402.07712 and https://arxiv.org/abs/2402.07043 point to model collapse being an inevitable result of data poisoning rather than a failure of current methods. 

## Image data

This repo aims to show model collapse on image data in a computationally efficient and easily understandable way.

A variational autoencoder (VAE) is trained on MNIST alongside a classifier. This VAE is then used to generate a new dataset of MNIST-like digits, which are then classified using the classifier trained on the true MNIST dataset. This process is repeated, with the next VAE trained on the previous VAE's generations, and the next VAE's generations classified by the previous classifier. This allows us to see how noise creeps into generations over time due to the saturation of prior model outputs.

## Training process

To understand how this impact the outputs, lets look at random samples taken from MNIST.

![image](https://github.com/user-attachments/assets/9506a7dd-cc8d-49d8-b73e-47c88d6652b7)

A VAE is trained on the full dataset of 60,000 images until convergence. Using this trained model, 60,000 new images are sampled to create a mock dataset for further training. After a single iteration, these generations are quite close to the original MNIST data.

![image](https://github.com/user-attachments/assets/33ad0457-64df-4a23-bc73-4282a6b37737)

This process can then be continued indefinitely. As you might have imagined, the quality of these generations does not stay high quality for very long. After just 10 iterations, the digits look significantly less clear and comprehensible that after 1 iteration. But, there is perhaps a more notable change which occurs. The images lose a very large amount of diversity. In almost every experiment done which does not quickly descend into white noise, the images generated wind up entirely 8s. 

![image](https://github.com/user-attachments/assets/4b5830bc-2e26-40d5-9e2b-8acec1ba7cf7)

And after over 80 generations, a stable point seems to emerge. A kindd of smudged dataset of almost entirely 8s seems to be an attractor state of this system.

![image](https://github.com/user-attachments/assets/c0d21a58-dcad-430e-b003-0e38d0d1ba3f)

