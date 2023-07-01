# NVFlare Decorator Proposal

In this repository, we demonstrate how to rewrite the official [CIFAR10 tutorial](https://github.com/pytorch/tutorials/blob/main/beginner_source/blitz/cifar10_tutorial.py) provided by PyTorch and make it runnable in NVFlare using the "decorator" (For reference, we copy the entire original code [here](./cifar10_tutorial.py)).

Note that in this demo we choose CIFAR10 because it is one of the simplest and well-known example to the community.
The same decorator mechanism can be applied into more complicated ML scenarios.

First, we remove the visualization and testing codes and comments from the original code, which we call the "clean" version. The code is [here](./cifar10_tutorial_clean.py). Note that we move the network definition to [here](./net.py).

Up to this point, we have not made any modifications (other than removal) to the original code.

For the next step, we need to restructure the original flat code into "train" and "evaluate" methods. Finally, we need to use the NVFlare decorators "fl_train" and "fl_evaluate" to decorate the restructured code so that the new code can run in NVFlare smoothly.

We have different options to restructure and apply the decorators.

We have come up with the following proposals, and their code is put under the corresponding directory. The restructured code is "cifar10.py," and the FL decorated code is "cifar10_fl.py".

## Proposal A [[code]](./A)

Description: Use the "injected attributes" directly in the method body code.

Re-structure diff: https://www.diffchecker.com/iJwkCVPw/

ML-to-FL diff: https://www.diffchecker.com/r2yx6So3/ 

Pros:

- The train and evaluate methods can have arbitrary input arguments defined and decide what to return (can return None).
- Takes less effort to restructure code.
- If users have existing training and evaluation methods, they do not need to change the existing method signature.

Cons:

- Takes more effort to convert from restructured code to FL code.
- Changing the restructured code to FL version requires 7 lines of changes.
- Need to know where to load from the global model (`net.load_state_dict(evaluate.fl_model_params)`).
- Need to know the "injected attribute's name" to set it correctly (`train.fl_model_params = net.state_dict()`).

## Proposal B [[code]](./B)

Description: Add an additional argument to the method definition/signature. The object passed in that argument represents the model parameters.

Re-structure diff: https://www.diffchecker.com/LGMRn46u/

ML-to-FL diff: https://www.diffchecker.com/Tr4P7CW1/

Pros:

- Changing the restructured code to FL version needs 3 lines of changes.

Cons:

- Takes more effort to restructure code.
- The restructured code needs to comply with the following requirements:
  - The train method needs to have an argument referring to model parameters so that the FL decorator can substitute the argument to pass in global model parameters from the NVFlare server side.
  - The train method needs to return model parameters so that the FL decorator can return the trained model back using the return value.
  - The evaluate method needs to have an argument referring to model parameters so that the FL decorator can substitute the argument to pass in global model parameters from the NVFlare server side.
  - The evaluate method needs to return metric.
  - The model parameters need to have a form that the FL decorator supports, for example, a dictionary of PyTorch tensors.

## Proposal C [[code]](./C)

Description: Add an additional argument to the method definition/signature. The object passed in that argument is a PyTorch module that has "load_state_dict" and "state_dict" methods implemented. The pros and cons are the same as Proposal B.

Re-structure diff: https://www.diffchecker.com/BBl4cSEN/

ML-to-FL diff: https://www.diffchecker.com/gMhBr6U4/

Pros:

- Changing the restructured code to FL version needs 3 lines of changes.

Cons:

- Takes more effort to restructure code.
- The restructured code needs to comply with the following requirements:
  - The train method needs to have an argument referring to model parameters so that the FL decorator can substitute the argument to pass in global model parameters from the NVFlare server side.
  - The train method needs to return model parameters so that the FL decorator can return the trained model back using the return value.
  - The evaluate method needs to have an argument referring to model parameters so that the FL decorator can substitute the argument to pass in global model parameters from the NVFlare server side.
  - The evaluate method needs to return metric.
  - The model parameters need to have a form that the FL decorator supports, for example, a dictionary of PyTorch tensors.
