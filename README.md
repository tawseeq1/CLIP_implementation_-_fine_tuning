## CLIP Report 
CLIP, which stands for Contrastive Language-Image Pretraining, was introduced by Open AI in 2021 to bridge the gap between visual and textual data. Deep learning models have revolutionized the field of Computer vision, but we still face a problem. The problem is that the current models are inefficient in predicting the type of data they were not being trained on. For example: let's suppose the model was trained on a vast image dataset of fruits and vegetables to perform classification tasks, but if we have a car to be classified, it will perform poorly.

It has been found that incorporating NLP techniques in Computer Vision tasks improves the capabilities of zero-shot transfer. Using NLP has a lot of advantages as it is easy to collect the data from the internet (image text pairs), and it doesn't just learn a representation but also connects that representation to language, which is the main reason for zero-shot transfer. 

Already existing datasets were not suitable to be used for CLIP due to several issues, eg. no text label for the images, and some utilize automatically generated filenames such as "2342456 113957.JPG" as their "titles" or include "descriptions" of camera exposure settings. Because of this new data was needed to be collected. The model (CLIP) is trained on a dataset of 400 million training examples, where the data was collected from the internet, mostly Wikipedia as image-text pairs where the text is a part of the construction process whose text includes one of a set of 500,000 queries. The base query list is all words occurring at least 100 times in the English version of Wikipedia. This is augmented with bi-grams. 
Finally, 20,000 image text pairs were obtained, hence  million training dataset was obtained.

Contrastive learning: Instead of predicting the exact words of the text, the model (computationally more expensive and performs poorly) trains to recognize which text goes with which image in a batch of many images and texts. This approach focuses on matching images with their correct descriptions and not on the exact words.


The model works on the transformer architecture, where both image and text encoders are the transformers. Let's suppose we have  number of images and  number of sentences (captions), each image related to one caption. This will be used for our training purposes. Each Image is passed to an encoder, which is a transformer. This outputs a particular vector for that training image. On the other hand, the sentence (caption) is input into a text encoder, which is also a transformer. The output from this text transformer is also the same dimensional vector as the image encoder.  Then an  matrix is formed with  vectors generated by the image transformer in rows and  vectors generated by the text transformer in columns. Each element in the matrix is the dot product of its respective vector of row and column, which gives us the idea of cosine similarity. 
In the training process, our aim is to maximize the diagonal elements to as close to 1 and other elements close to 0, which is changing the parameters (with respect to the transformer) such that the cosine similarity between correct image and text pairs is very high and incorrect pairs is very low. 


In this image, the highlighted box values should be maximized and others minimized. 

Two different types of architectures for image transformers were used. 
ResNet-50 as the base architecture with some modifications using ResNet-D and replacing the global average pooling layer with an attention pooling mechanism, which means the model focuses more on important parts of the image.
Vision Transformer (ViT), that was introduced in 2020, as it can capture more detailed relationships in the data.

The text encoder is a transformer as described in this paper. It has 63 million parameters, 12 layers, and is a 512-meter model with 8 attention heads. Input text is encoded as byte-pair encoding that is input into the text transformer. 

Scaling the model: 
ResNet Scaling: Models are usually scaled by making them wider (more units per layer) or deeper (more layers), but in CLIP, efficient scaling is done, that is, increasing width, depth, and image resolution together works better than increasing just one of these. For simplicity, CLIP increases all three dimensions equally. 
Text Encoder Scaling: Only the width of the text model is increased to match the increase in width of the image model, while depth of the text model is not changed because the performance of CLIP is less sensitive to changes in the text model's depth.

Training:
5 ResNets and 3 Vision Transformers were trained. ResNet-50, a ResNet-101, and then 3 more models that follow the EfficientNet-style model scaling (labelled RN50x4, RN50x16, and RN50x64 because they use 4 times, 16 times, and 64 times the computing power of a ResNet-50) were the ResNets trained. For the Vision Transformers, ViT-B/32, a ViT-B/16, and a ViT-L/14 were trained. With some hyper parameter tuning and some other tasks for training, the model was trained with the following time:
The largest ResNet model, RN50x64, took 18 days to train on 592 V100 GPUs.
The largest Vision Transformer took 12 days on 256 V100 GPUs. For the ViT-L/14 we also pre-train at a higher 336 pixel resolution for one additional epoch to boost performance similar to FixRes. We denote this model as ViT-L/14@336px. 

Some experiments were performed using this model like Zero Shot Transfer, Representation Learning, Robustness to Natural Distribution Shift

The working principle of the model has been already explained above, noting that this prediction layer is a multinomial logistic regression classifier with L2-normalized inputs, L2-normalized weights, no bias, and temperature scaling.

# Zero Shot Transfer:
Visual N-Grams is an earlier method used for zero-shot learning in computer vision. It tries to recognize images without having seen specific examples of those images during training. It uses small visual elements (n-grams) to build up a recognition system.
CLIP significantly improves the accuracy of recognizing images on the ImageNet dataset from 11.5% with Visual N-Grams to 76.2%. CLIP achieves a top-5 accuracy of 95%, meaning it correctly identifies the image within the top 5 guesses 95% of the time. Below is the table for accuracies on different datasets by these two models.



Yahoo
ImageNet
SUN
Visual N-Grams
72.4
11.5
23.0
CLIP
98.4
76.2
58.5


In zero-shot learning, Prompt Engineering and Ensembling have some effect. There are a few issues we face in most of the standard computer vision tasks. The vast majority of datasets annotate images with just a numeric ID of the label and contain a file mapping these IDs back to their names in English and another issue is polysemy, which is multiple labels having the same meaning eg, the given word boxer can be both an athlete or a breed of dog, Thats why prompt engineering is required to specify the context so that model can clearly differentiate between different classes. Ensembling is the technique in which different models are trained, and their results are averaged out, reducing variance. Below is the plot describing the improvements due to Prompt Engineering and Ensembling. 

Zero-shot CLIP exhibits great performance compared to fully supervised models across various datasets. CLIP’s zero-shot performance is compared to a fully supervised logistic regression model based on ResNet-50 features. The above figure shows CLIP outperforms the baseline on 16 out of 27 datasets. CLIP excels in general object classification tasks (like ImageNet, CIFAR10/100) and some specific datasets like Stanford Cars and Food101, but it struggles with more specialized tasks like satellite image classification (EuroSAT), traffic sign recognition (GTSRB), and medical image classification (PatchCamelyon). Tasks like counting, satellite image classification, and traffic sign recognition are performed reliably by non-expert humans, indicating considerable potential for enhancement. However, caution is advised regarding the relevance of evaluating zero-shot transfer, rather than few-shot transfer, for tasks where learners lack prior experience, such as lymph node tumor classification, which remains challenging for most humans and possibly for CLIP as well.

The above plot describes the effectiveness of Zero-Shot CLIP. Few-shot learning means the model is trained with a small number of examples per class. Figure 6 above shows that zero-shot CLIP performs as well as a few-shot logistic regression model trained on four examples per class. Zero-shot CLIP matches the average performance of a 4-shot linear classifier trained on the same feature space and nearly matches the best results of a 16-shot linear classifier across publicly available models. The reason behind this might be due to the fact that CLIP’s zero-shot classifier is generated via natural language which allows for visual concepts to be directly specified (“communicated”). By contrast, “normal” supervised learning must infer concepts indirectly from training examples. Context-less example-based learning has the drawback that many different hypotheses can be consistent with the data, especially in the one-shot case. 
The efficiency of zero-shot CLIP is exceptionally well in some cases but, it is not the case always. 


Figure 7 estimates how many labeled examples per class a few-shot model needs to match zero-shot CLIP’s performance. Efficiency varies widely, with some datasets requiring just 5 examples per class (five shot) , while others need up to 184 (184 shot).

Figure 8 shows that while zero-shot CLIP performs well, it still generally falls 10 to 25% behind fully supervised models (in our case, linear classification model applied to CLIP's feature representations). Only on a few datasets does zero-shot CLIP come close to fully supervised performance.
The linear regression model's slope, predicting zero-shot performance based on fully supervised performance, suggests that for every 1% increase in fully supervised performance, zero-shot performance tends to improve by 1.28%. 



Representation Learning:
Representation learning is a machine learning process of extracting some patterns from raw data to create representations that are easier to understand and process. Understanding the learned representations is crucial in CLIP, which is trained using contrastive learning. There are two common approaches of evaluating representation learning models: Linear classifiers and end-to-end fine-tuning. Linear classifiers are preferred because they provide clear feedback on the quality of representations and highlight failures in learning generalizable features, and training linear classifiers align with the approach used for zero-shot classifiers in CLIP, enabling extensive comparisons and analysis. Also linear classifiers require minimal hyper-parameter tuning and have standardized implementations and evaluation procedures, making them suitable for comparing a diverse set of techniques across multiple tasks.

In the results, it can be noticed that CLIP models perform extremely well in most of the cases but underperform in some compared to other models, suggesting potential areas for improvement such as data augmentation strategies. 



Robustness to Natural Distribution Shift
Deep learning models, despite achieving impressive results on certain tasks like ImageNet, still struggle with various challenges. Back in 2015, a deep learning model achieved better performance than humans on the ImageNet dataset. However, since then, researchers have discovered that these models still make many simple mistakes. New tests have revealed that their performance is often lower than expected when faced with different types of data or tasks. This discrepancy between their performance on ImageNet and real-world challenges raises questions about their robustness and generalization abilities. Researchers have proposed several explanations for this performance gap. One common idea is that deep learning models are very good at spotting patterns in the data they were trained on. However, some of these patterns may not hold true for other types of data, leading to a drop in performance when the model encounters new situations.
The findings of Taori reveal that the accuracy of ImageNet models drops when evaluated on these natural distribution shifts, indicating a lack of robustness. However, the study also observes a predictable relationship between in-distribution accuracy (accuracy on ImageNet) and out-of-distribution accuracy (accuracy on distribution shifts), suggesting that improvements in in-distribution accuracy could lead to better performance under distribution shifts.
Zero-shot models, like CLIP, are trained in a different way than traditional deep learning models. They are trained on a large dataset with natural language supervision, rather than being fine-tuned on specific tasks like ImageNet. These models offer an opportunity to explore whether their training approach leads to greater robustness and generalization.
Through various experiments, researchers have found that zero-shot models like CLIP indeed demonstrate higher robustness when faced with distribution shifts compared to traditional ImageNet-trained models. However, adapting these models to specific datasets like ImageNet may improve their performance on that dataset, but it might not necessarily translate to better performance on other types of data. Adapting CLIP to ImageNet leads to an increase in ImageNet accuracy, it results in a slight decrease in average robustness, indicating a trade-off between in-distribution performance and robustness to distribution shifts.
These findings suggest that training models in a task-agnostic manner and evaluating them on diverse datasets could lead to the development of more robust systems. This approach provides a more accurate assessment of a model's performance across different tasks and data types. Researchers are interested in exploring whether similar trends hold true for zero-shot models in other domains, such as natural language processing.

 
Comparison to Human Performance
This portion compares how well people do at identifying dog and cat breeds in pictures compared to our CLIP model. The researchers tested people by showing them pictures with no examples (zero-shot), one example (one-shot), and two examples (two-shot) of each breed. Humans performed reasonably well even in the zero-shot scenario (54% right), where they had no examples to reference. Their accuracy improved significantly when given one (76% right) or two examples per breed. Interestingly, the biggest gain in accuracy from zero-shot to one-shot occurred on images that humans were initially uncertain about. This suggests that humans can learn from just a few examples and adjust their predictions accordingly. 
CLIP does well even without any examples, but unlike people, it doesn't improve much by seeing more examples. The researchers think CLIP could be improved by making it learn from past experiences better, just like people do. Even though CLIP is really good, there's still a gap between how well machines and people learn from a few examples. Overall, people are better at learning from a few examples than CLIP, but CLIP is great at using what it already knows. There is a future prospectus for this improvement.

Data Overlap Analysis
This portion takes into account is their is an overlap between training dataset and evaluation datasets. It could potentially invalidate the evaluation results as a meaningful test of generalization. 
The authors used a program,  duplicate detector,  to find similar examples between the training data and each test set. Based on a manually set threshold, they divided the dataset into two subsets: Overlap (containing similar examples) and Clean (containing dissimilar examples). They calculated the ratio of Overlap to the full dataset size to measure the degree of data contamination. They assessed the impact of overlap on model performance by computing the zero-shot accuracy of CLIP on both the full dataset and the Clean subset. The difference in accuracy showed how much the overall score might be inflated by the model remembering (overlap) examples. Additionally, they ran tests to see if the accuracy difference between "Overlap" and "Clean" was statistically significant.

Summary of Analysis: Out of 35 datasets studied, 9 had no detected overlap. The median overlap was 2.2%, and the average was 3.2%. Overall, overlap rarely affected accuracy by more than 0.1%. Only 7 datasets significantly impacted accuracy, with a maximum increase of 0.6%. Even datasets with large overlaps didn't see significant accuracy changes, suggesting that the impact of overlap was minimal.

Their findings agree with other studies, suggesting that some overlap has minimal impact on performance. This analysis helps ensure the validity of the test results by showing how much overlap there is and how it affects the model's performance.

Limitations

Even though CLIP is a big step forward, there are still areas where it can be improved. Here's a breakdown of its weaknesses: 
Performance on Specific Tasks: While CLIP performs competitively on datasets without training splits, its performance is average compared to simple supervised models on tasks with training splits. CLIP struggles with fine-grained classification tasks (like differentiating between car models or flower species) and abstract tasks (like counting objects in an image). 
Generalization to Out-of-Distribution Data: CLIP struggles to generalize to data that is truly out-of-distribution. For example, while it performs well on digitally rendered text, its accuracy drops significantly on handwritten digits like those in the MNIST dataset. 
Flexibility in Output Generation: CLIP can generate classifiers for various tasks, but it's limited to choosing from predefined concepts. It can't generate novel outputs like a truly flexible approach such as image captioning. 
Poor Data Efficiency: CLIP compensates for its poor data efficiency by using a vast amount of training data. It would take a lot of time to iterate through all the image-text pairs seen during CLIP's training. Combining CLIP with self-supervised and self-training methods could improve data efficiency. 
Methodological Limitations: The methodology used to evaluate CLIP has limitations. Performance is often queried on full validation sets, which is unrealistic for true zero-shot scenarios. The selection of evaluation datasets might be biased, and creating new benchmarks designed explicitly for evaluating zero-shot transfer capabilities would help address this issue. 
Biases in Training Data: CLIP is trained on unfiltered and uncurated image-text pairs from the internet, leading to the learning of social biases. This can impact its performance and behavior. 
Limitations of Natural Language Interface: While specifying image classifiers through natural language is flexible, it has its limitations. Complex tasks and some visual concepts can be challenging to specify with just text. 
Few-Shot Learning Performance: CLIP does not optimize directly for few-shot performance, and its performance drops when transitioning from zero-shot to few-shot settings. Future work is needed to develop methods that combine CLIP's strong zero-shot performance with efficient few-shot learning.

Overall, CLIP is a great leap forward, but there's still work to be done before it can be used for everything. By addressing these limitations, CLIP can become even more powerful and versatile.





Broader Impact:
Wide Range of Capabilities: CLIP can perform arbitrary image classification tasks, from identifying cats and dogs to medical diagnosis. However, it's essential to evaluate its performance and suitability for different purposes and analyze its broader impacts. 
Creating Custom Classes: CLIP enables users to create their own classes for categorization without re-training. This capability, similar to large generative models like GPT-3, introduces challenges and possibilities for novel applications that may not be apparent initially. 
Promise for Widely-Applicable Tasks: CLIP shows promise for tasks like image retrieval or search, where it can find relevant images given text or relevant text given an image. Its ease of customization could unlock various applications that are currently difficult to envision. 
Capabilities for Surveillance: CLIP's capabilities, such as action recognition, object classification, and facial emotion recognition, can be used in surveillance. Understanding its implications in this domain is essential due to its potential social impact. 
Characterizing Social Biases: Efforts are made to characterize the social biases inherent in CLIP. However, further exploration and development of testing schemes are needed to understand better and mitigate biases in general-purpose computer vision models.

Bias and Surveillance:

This bias in the model can arise from the training data, the algorithmic decisions made during model development, and how classes are defined and used. The data used to train models like CLIP often contains inherent biases that reflect societal inequalities and prejudices. The model will likely learn these biases if the data contains biased associations. Also, how classes are defined and used in the model significantly impacts the biases in the output. Poor class design can lead to biased results. Since we can define our classes, this flexibility can introduce biases. Choices about model architecture, feature selection, and thresholds for class probabilities influence the bias. These decisions can cause biases if not carefully considered.

FairFace dataset is designed to balance representations of age, gender, and race to mitigate biases. CLIP was evaluated using this dataset through two methods. First one being zero-shot CLIP (ZS CLIP) and second one logistic regression classifier on CLIP’s features (LR CLIP). LR CLIP outperformed both the ResNext-101 Instagram model and FairFace’s own model in many classifications, though performance varied across different categories.

An experiment showed that certain demographic groups were more likely to be misclassified into harmful categories. For example, 14% of Black images were misclassified into non-human categories like ‘animal’ or ‘chimpanzee,’ whereas other racial groups had lower misclassification rates. Similarly, images of younger individuals (aged 0-20) were more frequently classified into crime-related and non-human categories.
Men were more likely to be misclassified into crime-related categories than women, and young individuals were more likely to be misclassified into both crime-related and non-human categories. However, adding a ‘child’ category significantly reduced the misclassification of young individuals into harmful categories, highlighting how class design can influence model behavior and mitigate biases. Different threshold values for label probabilities affect the quality and bias of labels. Lower thresholds resulted in gendered associations with certain occupations (e.g., ‘nanny’ for women and ‘prisoner’ for men).

Even if a model shows higher accuracy and lower disparity in performance on benchmarks, this does not guarantee fairness in real-world applications. Higher performance might be used to justify the product (model) that disproportionately impact certain groups. The ability to design and define classes flexibly in CLIP poses both opportunities and risks. Poorly designed classes can exacerbate biases, whereas thoughtful class design can mitigate some of these issues. Decisions about threshold values for class probabilities can significantly impact the nature of biases in the model’s outputs. Lower thresholds can introduce gendered and racial biases that might not be as apparent at higher thresholds.
————
Surveillance is like watching people closely, especially using computers and cameras (AI and computer vision). The goal might be to keep people safe or to keep track of what they're doing to control them. This kind of watching can be a sensitive issue, so it's important to understand how new AI tools like CLIP work in these situations.
CCTV Image Classification: This is divided into two categories: Coarse Classification and Fine-Grained Detection. Coarse Classification mainly focuses on identifying the main subjects of image while as fine-grained classification focuses on the deeper features of the image. In case of coarse classification, CLIP performed well with a top-1 accuracy of 91.8%, identifying scenes like empty parking lots or school campuses. However, when the task was made more challenging with similar but distinct options (e.g., “parking lot with white car” vs. “parking lot with red car”), accuracy dropped to 51.1%, with 40.7% of errors being closely related but incorrect choices. In case of fine-grained classification, CLIP’s performance was near random, indicating poor capability in detecting fine-grained details in low-resolution surveillance images.
Zero-Shot Celebrity Identification: CLIP's ability to recognize celebrities (it was done for the purpose of face detection) using the CelebA dataset, which contains a large number of images of celebrities. CLIP achieved 59.2% top-1 accuracy for 100 celebrity classes but dropped to 43.3% for 1,000 classes, indicating a decline in performance with an increase in class size. Though these results are not competitive with specialized models like Google’s Celebrity Recognition, these results were achieved using zero-shot capabilities without additional task-specific data, showcasing the potential of CLIP’s generalization ability.

Future Work

Identifying potentially beneficial downstream uses of models early in the research process, enabling other researchers to think about applications. 
Better characterizing biases in models, alerting other researchers to areas of concern and areas for interventions. 
Creating suites of tests to evaluate systems like CLIP on, so we can better characterize model capabilities earlier in the development cycle. 
Identifying potential failure modes and areas for further work.






