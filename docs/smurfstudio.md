---
layout: default
nav: smurf
permalink: /smurfstudio/
title: Smurfstudio
---


Smurfstudio is MLD's addition to the widely used Nerfstudio, a publicly available and collaborative repository for creating and training NeRF models. I recommend that anyone interested check out the Nerfstudio webpage, they have even more resources that would be valuable for those getting into NeRF modeling for research. 

Those using Smurfstudio should consult the ReadMe when getting set up, it will walk you through the initial steps of getting set up and into a docker container. Past this there is a bit of a heavy lift with respect to getting familiar with the container ---  again I highly recommend checking out the pipelines overview section in Nerfstudio (under Developer Guides).

## Training in Smurfstudio

To train using any set of images in Smurfstudio you first need to generate a transforms.json file. This contains all image locations, as well as their corresponsing extrinsic camera matrices and intrinsic camera values. I recommend converting all image information into a CSV, and modelling a similar script after my own dataframes_transforms.py found under `smurfstudio/nerfstudio/scripts/dataframes` in the repo. Take extra care to get the extrinsic transforms correct, with APL specifically some of their models are generated using different axis conventions and there's basically no way to tell unless you ask them. Currently I have my script set up with the two different conventions I've seen so far ("hubble") being the most frequently used, and what will probably continue to be used in the future if Allen Mathews is the one who is generating the images you use. You will quickly be able to tell if your matrices are wrong upon entering the viewer, either the images will look like they are in the complete wrong orientation, or the model will fail to train (the tell here is usually that the model will learn some blob in the center but never any detail). 

Once you have the transforms.json file you can move on to `smurfstudio/model-training`. This has some examples of the objects you will construct to your own models. The most important arguments will have comments next to them about what they are and what they do. I would say the most important to know about is the scene-box pixel sampler, setting the bas orientation to none, and the scene-box scale factor. When training a model, Nerfstudio automatically provides you with a viewer that you can access from the `ports` tab in the terminal. This allows you to interact with the model while and after it has trained.

## IO Script

Finally, at some point you may want to simply output images from a model instead of look at the model in the viewer, I recommend that you check out `smurfstudio/nerfstudio/scripts/experiments/utils`. This shows you how to use the ImageRenderer class I have made, which takes in a model and camera poses to generate images from a pre-trained model.

## Closing Remarks

I will probably continue to add more to this page where appropriate, if something is missing and you figure it ought to be included here, please let me know!


> ![Smurfstudio training timelapse]({{ "/assets/img/skeleton-doot.gif" | relative_url }})