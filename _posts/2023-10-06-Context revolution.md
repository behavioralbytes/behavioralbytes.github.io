
---
title: The situational context revolution in emotion recognition
date: 2023-10-06 14:41:25 -0300
categories: [Emotion]
tags: [discussions]     # TAG names should always be lowercase
---

It is not new to anyone how facial expression recognition is a powerful tool to classify and describe perceived emotions in people. Reading faces is natural to many of us, at least to some level; while experts can decode muscle activations in a "lie to me" level (cheers, Dr Ekman!), most of us can interpret simple expressions of anger, disgust or fear in people that we interact with routinely. However, a significant component is almost always forgotten: faces are always accompanied by background information that adds context to what is happening.

  

The year 2017 was an important year for the emotion recognition "community": with the availability of EMOTIC [kosti2017emotic], a dataset for emotion recognition that has a significant appeal to context-aware emotion perception, we saw emotion recognition taking shape as a task, and distancing themselves from their old "parent", facial expression recognition. After that, many other scientists started researching how context could help emotion recognition from a computer vision perspective, and most current techniques focus on processing these cues to improve accuracy. In this post, we will take a byte on the topic of situational context, both from a cognitive science point of view as from computer science.

  

Dr. Lisa Barrett's work, as corroborated by many other scientists, states that there is a consistency in how context affects emotion perception, specifically how a judge encodes facial expressions. This assumption was validated in the past decade by using fMRI (an imaging technique that is used to map brain activity by detecting changes in blood flow and oxygenation) in an experiment by Mobbs et al. (2006) [mobbs2006kuleshov], in which given identical faces across different contextual backgrounds, perceivers would judge these faces with different emotional features based on the contextual framing. Righart and de Gelder (2008) [righart2008rapid] extend these experiments to understand when the context is encoded when processing emotional information by using event-related potentials (ERPs). ERPs are a tool in the neuroscience field that studies the brain's electrical activity for specific stimuli. Two ERPs are especially relevant in this scenario: the N170, which is a component related to face encoding and happens ~170 ms after the stimuli, and the P1, which is also related to facial processing and happens at around 100 ms after the stimuli. Therefore, how do these two components are affected by emotional contexts? The study shows that different amplitude levels for these ERPs were recorded based on the added emotional context but also that different emotions have faster reaction times.

  

So, what we know so far is:

1.  Faces do not come to us isolated but, rather, have contextual information encoded automatically.
2.  Context is capable of changing drastically the emotional perception of someone.
3.  The encoding of context happens early and is triggered by ERPs that are related to facial processing.

Suppose context is essential for emotion perception in humans. In that case, we may argue that this will also be at least helpful for the task of emotion recognition. Therefore, we can treat situational context as a nonverbal cue and design a model that would:

1.  Encode contextual information automatically without the need to design a feature extraction pipeline that will manually highlight essential data.
2.  Take this encoded information into account to allow different perceptions based on context -- in other words, context also needs to have a significant contribution to the process.
3.  It should happen early so that it is non-dependent on other processing data through the inference pipeline.

  

We propose two experiments that are based on these three findings: in the first experiment, we are interested in the visual data of the background, and we use context as a nonverbal cue together with other cues, encoding it in an early way so that it can be taken into consideration with the same level of importance at the start of inference. In the second experiment, we are interested in checking how we can tackle cases with unrepresentative contexts, as it is difficult to judge this a priori. So, we extract contextual descriptions from images and follow the same 3-step "requirements" to classify emotion.

  

#### Experiment 1 - Image-based context processing

> The results of this experiment are currently under consideration for publication at ESWA. The preprint is available here: https://papers.ssrn.com/sol3/papers.cfm?abstract_id=4255748

We design our context encoding stream so that we can automatically select important regions of the background, inspired by how humans encode context. Therefore, we occlude the facial region in the input images. In this experiment, we decided to go with simple convolutional blocks: convolutional layers with 3x3 kernels reducing spatial size and increasing the channel dimensions from 3 → 32 → 64 → 128 and → 256. Then, at the last layer, we have several learned representations of data, and we need to guide the model to what is actually important. For this, we use two strategies:

We employ a self-calibrated convolution layer [liu2020improving], a convolutional layer that allows self-calibrating properties by building long-range spatial and inter-channel dependencies. In other words, this layer allows triggers that will improve the representations generated during the forward step.

After the self-calibrated refinement, we proceed to employ an attention block that will boost the learnt features from this stream. It is important to notice that although self-calibration shares properties with attention mechanisms, they differ in how they were implemented in this work: self-calibration works within the layer, while attention blocks are applied between layers and modify the relationships of its input.

  

Finally, we fuse the contextual information with other nonverbal cues to predict emotion. This architecture is simple by design to allow low inference time.

  ![Simple overview of the model proposed in this experiment. For more details, please check the preprint mentioned above.](https://github.com/behavioralbytes/behavioralbytes.github.io/blob/main/_posts/arch.png?raw=true)

On the CAER-S dataset [lee2019context], which is a dataset for emotion recognition on images with contextual information available, we achieve 85.58% accuracy using only this contextual stream (without any other nonverbal cue), increasing to 89.76% in accuracy when combining face and body language information, ranking us in the second place in this benchmark (by 0.12%), with a much simpler architecture than the SOTA method, which combines facial information with context through execution in a global attention format. We can, however, perform inferences 9 times faster than this method.

  

#### Experiment 2 - Description-based context processing
> The results of this experiment was presented in the LatinX in AI Workshop @ CVPR 2023. The paper is available here: https://openaccess.thecvf.com/content/CVPR2023W/LatinX/html/de_Lima_Costa_High-Level_Context_Representation_for_Emotion_Recognition_in_Images_CVPRW_2023_paper.html

  

After our first results from Experiment 1, we started wondering what to do in application cases where the context would not be _too_ representative. You see, most research from the psychological point of view used samples with clear background information for emotion perception, such as a car crash or a supermarket (implicating neutrality, for example). But how can these models process cases with a simple, white background from a room?

  

Another problem relies on bias. Datasets such as CAER-S or EMOTIC contain a lot of common contexts (EMOTIC even has COCO samples!), but how could they encompass _all_ possible context scenarios? We propose, then, to not extract this automatically from images but rather generate natural language descriptions with sentic and semantic meaning.

  

Given an input image, we use ExpansionNet-V2 to extract image captioning and add sentic meaning using SenticNet, a large knowledge base for sentic descriptions. With these descriptions, we create a knowledge graph that will be used for predicting emotion.

  

We achieve interesting results on EMOTIC! Using only one cue, we surpass various models that use up to five cues, reaching 0.30 mAP. Although we are not SOTA with this experiment, we could validate a lot of interesting ideas.

  

#### Conclusion

  

Context is really meaningful for emotion recognition, and its variability allows for a lot of different experiments. EMOTICon, for example, even employs a pipeline to predict the depth of each person in the scene to describe social interactions! However, we are still seeking the most descriptive form to process context.

  

In this experiment, we investigate how we can extract contextual information from a more global perspective. This is especially helpful in some challenging cases in which the context could be more representative.

  
#### References

 - [kosti2017emotic] Kosti, R., Alvarez, J. M., Recasens, A., & Lapedriza, A. (2017). Emotion recognition in context. In _Proceedings of the IEEE conference on computer vision and pattern recognition_ (pp. 1667-1675).
 - [mobbs2006kuleshov] Mobbs, D., Weiskopf, N., Lau, H. C., Featherstone, E., Dolan, R. J., & Frith, C. D. (2006). The Kuleshov Effect: the influence of contextual framing on emotional attributions. _Social cognitive and affective neuroscience_, _1_(2), 95-106.
 - [righart2008rapid] Righart, R., & De Gelder, B. (2008). Rapid influence of emotional scenes on encoding of facial expressions: an ERP study. _Social cognitive and affective neuroscience_, _3_(3), 270-278.
 - [liu2020improving] Liu, J. J., Hou, Q., Cheng, M. M., Wang, C., & Feng, J. (2020). Improving convolutional networks with self-calibrated convolutions. In Proceedings of the IEEE/CVF conference on computer vision and pattern recognition (pp. 10096-10105).
 - [lee2019context] Lee, J., Kim, S., Kim, S., Park, J., & Sohn, K. (2019). Context-aware emotion recognition networks. In Proceedings of the IEEE/CVF international conference on computer vision (pp. 10143-10152).
