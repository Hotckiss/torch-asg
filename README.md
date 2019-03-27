# Auto Segmentation Criterion (ASG) for pytorch

This repo contains a pytorch implementation of the auto segmentation criterion (ASG), introduced in the paper 
[_Wav2Letter: an End-to-End ConvNet-based Speech Recognition System_](https://arxiv.org/abs/1609.03193) by Facebook.

As mentioned in [this blog post](http://danielgalvez.me/jekyll/update/2018/01/12/wav2letter.html) by Daniel Galvez,
ASG, being an alternative to the connectionist temporal classification (CTC) criterion widely used in deep learning, 
has the advantage of being a globally normalized model without the conditional independence assumption of CTC and the 
potential of playing better with
[WFST](https://en.wikipedia.org/wiki/Finite-state_transducer#Weighted_automata) frameworks. 

Unfortunately, Facebook's implementation in its official 
[wav2letter++](https://github.com/facebookresearch/wav2letter) project is based on the ArrayFire C++ framework, which 
makes experimentation rather difficult. Hence we have ported the ASG implementation in wav2letter++ to pytorch as
C++ extensions. For the CPU implementation we have stayed quite close to the original implementation, whereas for the
GPU we have taken a lot of leeway while ensuring that the results tally with the CPU's.

## Project status

* [x] CPU (openmp) implementation
* [ ] GPU (cuda) implementation
* [ ] extensive testing
* [ ] performance tuning and comparison
* [ ] Viterbi decoders 
* [ ] generalization to better integrate with general WFSTs decoders

## Using the project

Ensure pytorch > 1.01 is installed, clone the project and in terminal do

```bash
cd torch_asg
pip install .
```

Tested with python 3.7.1. You need to have suitable C++ toolchain installed. Status on Windows is unclear.

Then in your python code:

```python
import torch
from torch_asg import ASGLoss


def test_run():
    num_labels = 7
    input_batch_len = 6
    num_batches = 2
    target_batch_len = 5
    asg_loss = ASGLoss(num_labels=num_labels,
                       reduction='mean',  # mean (default), sum, none
                       scale_mode='none'  # none (default), input_size, input_size_sqrt, target_size, target_size_sqrt
                       )
    for i in range(1):
        # Note that inputs follows the CTC convention so that the batch dimension is 1 instead of 0,
        # in order to have a more efficient GPU implementation
        inputs = torch.randn(input_batch_len, num_batches, num_labels, requires_grad=True)
        targets = torch.randint(0, num_labels, (num_batches, target_batch_len))
        input_lengths = torch.randint(1, input_batch_len + 1, (num_batches,))
        target_lengths = torch.randint(1, target_batch_len + 1, (num_batches,))
        loss = asg_loss.forward(inputs, targets, input_lengths, target_lengths)
        print('loss', loss)
        # You can get the transition matrix if you need it.
        # transition[i, j] is transition score from label j to label i.
        print('transition matrix', asg_loss.transition)
        loss.backward()
        print('transition matrix grad', asg_loss.transition.grad)
        print('inputs grad', inputs.grad)

test_run()
```

Hope this repo can help with your research.