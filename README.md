# Measuring the similarity scores of text-image pairs.  
<div style="text-align: right"> By: Trung Nguyen </div>

## About  

### My code, output/input  

The code is in "LeoAI.ipynb". Results with similarity scores are saved to "results.csv". Statistics of code running including time and memory usages are saved to "stats.log". Images and the data "challenge_set.csv" should be stored in "data/".  

### Model choice  

My approach is to use OpenAI's CLIP model [[1]](#1). CLIP includes a text encoder and an image encoder which respectively convert a text sequence and an image to a text embendding vector and an image embedding vector. The reason I chose CLIP is that this model has already aligned the text encoder and the image encoder. Otherwise, I would need to collect data and train at least an alighnment module (or both the encoders).  

#### The choice of the model checkpoint  

There are different variants of CLIP with varying model sizes and model accuracies. The rule of thumb is that the bigger size, CLIP provides better indications on the similarities for text-image pairs.  

The code uses Huggingface and Pytorch to simplify the experiment. Also, the parallelism was quite good as the waiting time is much shorter than the total CPU and GPU time (e.g. 7.7s vs more than 3000s). Huggingface provides the following CLIP model cards (refer to the paper for more details on the checkpoints):  
* "openai/clip-vit-base-patch32"  
* "openai/clip-vit-base-patch16"  
* "openai/clip-vit-large-patch14"   
* "openai/clip-vit-large-patch14-336"  

### About the scores  
The score between a pair is calculated based on how these two vectors (corresponding the text sequence and the image in the pair) align. This is basically the product between the two vectors given that they are already normalized.   

The score range yielded by this model is from 0 to 100. The higher score the more similar.  
It may be easier to see how similar a text and an image are by giving several candiates (images or text sequences) and compare (with softmax).  

## How my method can be used to effectively curate data for text-to-image model training?  

Disclaimer: this section quite resembles the methodology of OpenAI's ''Dall-e 2" [[2]](#2) (this paper is not exactly about Dall-e 2 but it is said that developing Dall-e 2 followed this methodology). I think a figure for illustration will help but I cannot do due to my time limit.  

For the text-to-image task, diffusion has been prevailing (over GANs) although there are still new studies on GANs. Diffusion enables adding details to abstract information, which can be the text prompt itself or the prompt represented in the embedding space (converted by an encoder). Although directly generating images from text prompts using diffusion (or GANs) is possible, it is a good idea to divide this process into several steps. Most state-of-the-art generative models for images use multiple generation steps.  

An idea to divide this whole text-to-image process is to take advantage of CLIP encoders. We can first use a diffusion model to generate the image embedding vector conditioned on the text prompt and/or the text embedding vector. Te next step is, with another diffusion model, to generate an image at a specified low resolution conditioned on the generated image embedding and/or the text prompt. Then, we can add a few steps to gradually increase the image resolution by generating finer images with higher resolutions at each diffusion step.  

This idea can take advantage of the pretrained encoders in CLIP, which were aligned using a approxiately 400m text-image pairs.  

### References  
<a id="1">[1]</a> Radford, A., Kim, J. W., Hallacy, C., Ramesh, A., Goh, G., Agarwal, S., ... & Sutskever, I. (2021, July). Learning transferable visual models from natural language supervision. In International conference on machine learning (pp. 8748-8763). PMLR.  
<a id="2">[2]</a> Ramesh, A., Dhariwal, P., Nichol, A., Chu, C., & Chen, M. (2022). Hierarchical text-conditional image generation with clip latents. arXiv preprint arXiv:2204.06125, 1(2), 3.  
