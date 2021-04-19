# Few-shot Image Generation via Cross-domain Correspondence

[Utkarsh Ojha](https://utkarshojha.github.io/), [Yijun Li](https://yijunmaverick.github.io/), [Jingwan Lu](https://research.adobe.com/person/jingwan-lu/), [Alexei A. Efros](https://people.eecs.berkeley.edu/~efros/), [Yong Jae Lee](https://web.cs.ucdavis.edu/~yjlee/), [Eli Shechtman](https://research.adobe.com/person/eli-shechtman/), [Richard Zhang](https://richzhang.github.io/)

Adobe Research, UC Davis, UC Berkeley

![teaser](https://github.com/utkarshojha/few-shot-gan-adaptation/blob/gh-pages/resources/concept.gif)

Repository for downloading the datasets and generated images used for performing the evaluations shown in Tables 1 and 2.

### [Project page](https://utkarshojha.github.io/few-shot-gan-adaptation/) | [Paper](https://arxiv.org/abs/2104.06820)

## Overview

<img src='imgs/method_diagram.png' width="840px"/>

Our method helps adapt the source GAN where one-to-one correspondence is preserved between the source G<sub>s</sub>(z) and target G<sub>t</sub>(z) images.

## Testing

We provide the pre-trained models for different source and adapted (target) GAN models. 
| Source GAN: G<sub>s</sub> | Target GAN: G<sub>s&#8594;t</sub> |
| ------------------------- | --------------------------------- |
| [FFHQ]() | [[Sketches]()] [[Caricatures]()] [[Amedeo Modigliani]()] [[Babies]()] [[Sunglasses]()] [[Rafael]()] [[Otto Dix]()] |
| [LSUN Church]() | [[Haunted houses]()] [[Van Gogh houses]() [[Landscapes]()] [[Caricatures]()] |
| [LSUN Cars]() | [[Wrecked cars]()] [[Landscapes]()] [[Haunted houses]()] [[Caricatures]()] | 
| [LSUN Horses]() | [[Landscapes]()] [[Caricatures]()] [[Haunted houses]()] |
| [Hand gestures]() | [[Google Maps]()] [[Landscapes]()] |

Note that the table above contains both the realted (e.g. FFHQ/sketches) as well as unrelated source/target pairs (e.g. LSUN Cars/Haunted houses) discussed in Section 4.2 of the main paper.

Download the pre-trained model(s), and store it into `./checkpoints` directory.

### Sample images from a model

To generate images from a pre-trained GAN, run the following command:

`CUDA_VISIBLE_DEVICES=0 python generate.py --ckpt_target model_name`

Here, `model_name` follows the notation of `source_target`, e.g. `ffhq_sketches`. Use the `--load_noise` option to use the noise vectors used for some figures in the paper (Figures 1-4). For example:

`CUDA_VISIBLE_DEVICES=0 python generate.py --ckpt_target ffhq_sketches --load_noise noise.pt`

### Visualizing correspondence results

To visualize the same noise in the source and adapted models, i.e. G<sub>s</sub>(z) and G<sub>s&#8594;t</sub>(z), run the following command(s):

```bash
# generate two image grids of 5x5 for source and target
CUDA_VISIBLE_DEVICES=0 python3 generate.py --ckpt_source source_ffhq --ckpt_target ffhq_caricatures --load_noise noise.pt

# visualize the interpolations of source and target
CUDA_VISIBLE_DEVICES=0 python3 generate.py --ckpt_source source_ffhq --ckpt_target ffhq_caricatures --load_noise noise.pt --mode interpolate
```

### Hand gesture experiments

We collected images of random hand gestures being performed on a plain surface (~ 18k images), and used that as the data to train a source model (from scratch). We then adapted it to two different target domains; Landscape images and Google maps. The goal was to see if, during inference, interpolating the hand genstures can result in meaningful variations in the target images. Run the following commands to see the results:

```bash
CUDA_VISIBLE_DEVICES=0 python3 generate.py --ckpt_source source_hand --ckpt_target hand_maps --load_noise noise.pt --mode interpolate
CUDA_VISIBLE_DEVICES=0 python3 generate.py --ckpt_source source_hand --ckpt_target hand_landscapes --load_noise noise.pt --mode interpolate
```


### Evaluating FID 

There are three sets of images which are used to get the results in Table 1:
- A set of real images from a target domain -- R<sub>test</sub> 
- 10 images from the above set (R<sub>test</sub>) used to train the algorithm -- R<sub>train</sub>
- 5000 generated images using the GAN-based method -- F

The following table provides a link to each of these images:

| | R<sub>train</sub> | R<sub>test</sub> | F |
|-- | ------ | ------- | ------------------------------|
| Babies | [link](http://vision9.idav.ucdavis.edu:8001/babies_real_train.zip) | [link](http://vision9.idav.ucdavis.edu:8001/babies_real_test.zip) | [link](http://vision9.idav.ucdavis.edu:8001/babies_fake.zip) |
| Sunglasses | [link](http://vision9.idav.ucdavis.edu:8001/sunglasses_real_train.zip) | [link](http://vision9.idav.ucdavis.edu:8001/sunglasses_real_test.zip) | [link](http://vision9.idav.ucdavis.edu:8001/sunglasses_fake.zip) |
| Sketches | [link](http://vision9.idav.ucdavis.edu:8001/sketches_real_train.zip) | [link](http://vision9.idav.ucdavis.edu:8001/sketches_real_test.zip) | [link](http://vision9.idav.ucdavis.edu:8001/sketches_fake.zip) |

R<sub>train</sub> is given just to illustate what the algorithm sees, and **won't** be used for computing the FID score.

Download, and unzip the set of images into your desired directory, and compute the FID score (taken from [pytorch-fid](https://github.com/mseitzer/pytorch-fid)) between the real (R<sub>test</sub>) and fake (F) images, by running the following command

`python -m pytorch_fid /path/to/real/images /path/to/fake/images`

### Evaluating intra-cluster distance

Download the entire set of images from this [link](https://drive.google.com/file/d/1GtFHCnS_J8FbrQ0tkF4AFMYubyLsu_Xu/view?usp=sharing) (1.1 GB), which are used for the results in Table 2. The organization of this collection is as follows:
 
```
cluster_centers
└── amedeo			# target domain -- will be from [amedeo, sketches]
    └── ours			# baseline -- will be from [tgan, tgan_ada, freezeD, ewc, ours]
        └── c0			# center id -- there will be 10 clusters [c0, c1 ... c9]
            ├── center.png	# cluster center -- this is one of the 10 training images used. Each cluster will have its own center
            │── img0.png   	# generated images which matched with this cluster's center, according to LPIPS metric.
            │── img1.png
            │      .
	    │      .
                   
```
Unzip the file, and then run the following command to compute the results for a baseline on a dataset:

`CUDA_VISIBLE_DEVICES=0 python3 feat_cluster.py --baseline <baseline> --dataset <target_domain> --mode intra_cluster_dist`

`CUDA_VISIBLE_DEVICES=0 python3 feat_cluster.py --baseline tgan --dataset sketches --mode intra_cluster_dist`


We also provide the utility to visualize the closest and farthest members of a cluster, as shown in Figure 14 (shown below), using the following command:

`CUDA_VISIBLE_DEVICES=0 python3 feat_cluster.py --baseline tgan --dataset sketches --mode visualize_members`

The command will save the generated image which is closest/farthest to/from a center as `closest.png`/`farthest.png` respectively.

<img src='imgs/cluster_members.png' width="840px"/>

**Note** We cannot share the images for the caricature domain due to license issues.

## Training (adapting) your own GAN

### Choose the source domain
- Only the pre-trained model is needed, i.e. no need for access to the source data.
- Refer to the first column of the pre-trained models table above.
- If you wish to use some other source model, make sure that it follows the generator architecture defined in this [pytorch implementation](https://github.com/rosinality/stylegan2-pytorch) of StyleGAN2

### Choose the target domain

Below are the links to all the target domains, each consisting of 10 images, used in the paper.

| Sketches | Amedeo Modigliani | Babies | Sunglasses | Rafael | Otto Dix | Haunted houses | Van Gogh houses | Landscapes | Wrecked cars | Maps |  
| -- | -- | -- | -- | -- | -- | -- | -- | -- | -- |





## Bibtex
If you find our code useful, please cite our paper:

```
@inproceedings{ojha2021few-shot-gan,
  title={Few-shot Image Generation via Cross-domain Correspondence},
  author={Ojha, Utkarsh and Li, Yijun and Lu, Cynthia and Efros, Alexei A. and Lee, Yong Jae and Shechtman, Eli and Zhang, Richard},
  booktitle={CVPR},
  year={2021}
}
```

## Acknowledgment

As mentioned before, the StyleGAN2 model is borrowed from this wonderful [pytorch implementation](https://github.com/rosinality/stylegan2-pytorch) by [@rosinality](https://github.com/rosinality). We are also thankful to [@mseitzer](https://github.com/mseitzer) and [@richzhang](https://github.com/richzhang) for their user friendly implementations of computing [FID score](https://github.com/mseitzer/pytorch-fid) and [LPIPS metric](https://github.com/richzhang/PerceptualSimilarity) respectively. 
