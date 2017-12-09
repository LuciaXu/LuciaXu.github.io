---
layout: post
title: Convolution Arithmetic
tags: [Network]
comments: True
---
<div class="message">
  This blog provides some formulas between the parameters of convolution layers. It will cover the standard convolution, transpose convolution, as well as dilated convolution.
</div>

# Convolution Arithmetic
Convolution makes deep learning so powerful in visual tasks, because it can preserve spatial information of images. Here, we focus on the mathematic part of convolution by giving the relationship among the input size, output size and filter size.

The analysis of relationship between convolution layers does not interact across axes. Here are some commonly used settings of convolution layers.

- the inputs are squares, with size *i*
- the filters are squares, with size *k*
- the strides along both axes are equal, with stride *s*
- the zero padding along both axes are equal, with padding *p*

## Standard Convolution

The formula of standard convolution:

$$
o = \lfloor \frac{ i + 2p - k}{s}\rfloor + 1
$$

*o* is the output feature map size.

The number of output feature map is the number of filters *F_o*. If there are *F_i* input feature map, the parameters in this layer is:
 $$(k \cdot k \cdot F_i) \cdot F_o + F_o$$


## Transposed Convolution

It is also called deconvolution. It is achieved by adding *s-1* zeros between each unit of input feature maps.

When *i* satisfies *i + 2p - k* is multiple of *s*, the formula of transposed convolution is:

$$
o = s (i' - 1) + a + k - 2p
$$

where *i'* is the stretched input obtained by adding zeros between each unit of input. *a* represents the number of zeros added to the top and right edges of the input.

## Dilated Convolution

It is also called atrous convolution. It's a convolution applied to input with defined gaps.

A dilated convolution with rate *r*, means skipping *r-1* pixels per step. An alternative way is to use standard convolution with enlarged filter size, by adding *r-1* zeros each filter unit:

$$
k' = k + (k - 1) \cdot (r - 1)
$$
