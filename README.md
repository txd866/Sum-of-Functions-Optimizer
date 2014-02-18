Sum of Functions Optimizer (SFO)
================================

This code implements the optimization algorithm, and reproduces the figures, contained in the paper    
```citation
Sohl-Dickstein, Jascha, Ben Poole, and Surya Ganguli
An adaptive low dimensional quasi-Newton sum of functions optimizer
arXiv preprint arXiv:1311.2115 (2013)
http://arxiv.org/abs/1311.2115
```

## Use SFO
To use SFO, you should first import SFO,
```python
from sfo import SFO
```
then initialize it, passing in your objective function and gradient, initial parameters, and an array of objects which will be passed into the objective function, with each entry in the array corresponding to a single subfunction or minibatch,    
```python
optimizer = SFO(f_df, theta_init, subfunction_references)
```
finally you should call the optimizer, specifying the number of optimization passes to perform,    
```python
theta = optimizer.optimize(num_passes=1)
```

Simple example code training an autoencoder is included at the end of this readme.  More detailed information can be found in the documentation in **sfo.py**.

## Reproduce figures from paper
To reproduce the figures from the paper, run **figure\_cartoon.py**, **figure\_overhead.py**, or **figure\_convergence.py**.  **figure\_overhead.py** and **figure\_convergence.py** both require a subdirectory **figure_data/** which contains training data, and is too large to commit to this GitHub repository.  This will be available for download shortly -- URL to follow.


## Example code

The following code example trains an autoencoder using SFO.

```python
from numpy import *
from numpy.random import randn
from sfo import SFO

# define an objective function and gradient
def f_df(theta, v):
    """
    Calculate reconstruction error and gradient for an autoencoder with sigmoid
    nonlinearity.  This is the objective function and gradient for each subfunction.
    v contains the training data, and will be different for each subfunction.
    """
    h = 1./(1. + exp(-(dot(theta['W'], v) + theta['b_h'])))
    v_hat = dot(theta['W'].T, h) + theta['b_v']
    f = sum((v_hat - v)**2) / v.shape[1]
    dv_hat = 2.*(v_hat - v) / v.shape[1]
    db_v = sum(dv_hat, axis=1).reshape((-1,1))
    dW = dot(h, dv_hat.T)
    dh = dot(theta['W'], dv_hat)
    db_h = sum(dh*h*(1.-h), axis=1).reshape((-1,1))
    dW += dot(dh*h*(1.-h), v.T)
    dfdtheta = {'W':dW, 'b_h':db_h, 'b_v':db_v}
    return f, dfdtheta

# set model and training data parameters
M = 20 # number visible units
J = 10 # number hidden units
D = 100000 # full data batch size
N = int(sqrt(D)/10.) # number minibatches
# generate random training data
v = randn(M,D)

# create the array of subfunction specific arguments
sub_refs = []
for i in range(N):
    # extract a single minibatch of training data.
    sub_refs.append(v[:,i::N])

# initialize parameters
theta_init = {'W':randn(J,M), 'b_h':randn(J,1), 'b_v':randn(M,1)}
# initialize the optimizer
optimizer = SFO(f_df, theta_init, sub_refs)
# run the optimizer for 1 pass through the data
theta = optimizer.optimize(num_passes=1)
# continue running the optimizer for another 50 passes through the data
theta = optimizer.optimize(num_passes=50)
# test the gradient of f_df
optimizer.check_grad()
```
