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

## Models collapsing before your very eyes

Putting all this together into a longer run, we can see how model collapse causes the generations to tend towards a small self-reinforcing set. Rather than accurately sampling from the input distribution, excessive training on a model's own outputs causes very high confidence in outputs driven by model bias rather than the variance inherent in the data.

![animation](https://github.com/user-attachments/assets/bc1ed16d-76af-4f46-b158-2ee943df3739)

## Key Findings

- Model collapse does indeed happen, and in an application which can be considered somewhat commercially relevant. Many previous papers on model collapse focus on highly artificial settings. Some are only applicable to transformers (https://arxiv.org/pdf/2406.11263), only relevant to RAG or equivalent schemes which while useful do not threaten a foundation model's functionality, only affect tails of distributions (and as such may only threaten fact-based reasoning) (https://arxiv.org/abs/2403.07857), or are otherwise not suitably general. It has been shown in the case of LLMs that model collapse can lead to repetitive outputs (https://arxiv.org/abs/2311.09807). Much of the research has been performed in extremely synthetic environments (https://arxiv.org/abs/2402.07712).
- This shows that injection of model outputs into training data can, over a reasonable amount of time, lead to failure of the model. In particular, the generative capabilities of the model are threatened.
- There seems to be a smoothing and averaging of pixel values, from the extreme prevalence of 0s to something more like normally distributed values with mean pixel value 0.5. It makes sense that as gaussian noise in the VAE accumulates, the maximum entropy distribution is tended towards for values between 0 and 1, which is the uniform distribution. I have not yet run this for enough iterations for this to occur, but I expect it to happen.
- Old classifiers perform less well on newly generated datasets. This implies a gradually growing statistical distance and growing irrelevance of early features
- Whether the classifier or VAE is reinitialised or not seems to make little difference. This implies that enough samples are being generated (over 10,000) that previous data is nearly completely lost.
- Crucially, model collapse can be clearly seen for image data. As these experiments can take around 2 hours to run even with large amounts of data, this may serve as a good playground for testing this phenomenon. LLMs are likely too big for any small researcher to see model collapse occur in any important way.
- The rate of model collapse can be measured in a data-driven way by using the accuracy and loss of classifiers trained on earlier iterations of the model collapse, applied to later generations. This gives a metric of the rate of model collapse
- Average VAE loss decreases after each iteration. This implies that the generations of a previous VAE are easier to compress for the next, which makes sense from the perspective of a gradual loss of complexity and the reinforcement of pre-existing features.

## Further Improvements

- VAE architecture currently only has one hidden layer. Good results have been achieved with a 784-256-32-24-16 architecture. https://arxiv.org/pdf/1812.07238
- Convolutional classifier is likely considerably faster than the current FNN
- I could experiment with EMNIST? https://arxiv.org/abs/1702.05373. An interesting way to make it more practical. [An EMNIST version can now be found in another colab notebook]
- Introduce newly generated images into the dataset slowly, such that it doesn't just train on entirely new digits. This way, the gradual failure of the VAE could be observed in a more realistic way. This is, after all, supposed to reproduce the model collapse which one would expect to happen to LLMs in the next couple of years.
- The paper https://arxiv.org/pdf/1812.07238 on sparsity in VAEs gives some good tools for testing how smooth the latent space is. As the data after many iterations may converge on a kind of clustered noise, the latent space directions should have sparse properties initially and lose them over successive iterations of generation and compression.
- Try using the FID score to test the difference between generated and original datasets (as opposed to separate classifier loss).
- See how pixel statistics change over time. It seems from cursory analysis that the mean pixel value tends towards 0.5, as opposed to the MNIST value of around 0.13. If this were the case, it may imply something general about how model collapse could be observed in real-world foundation models. I would expect a regression towards the mean for continual introduction of gaussian errors, but I can't see a good reason why the mean of such errors should be 0.5?

