# Spooktroscopy: Frequency-Domain Ghost Imaging

## Target Problem
Solve <img src="https://render.githubusercontent.com/render/math?math=(A \otimes G)X = B"> under [regularizations](#Regularizations). A, G are matrices acting on the two indices of X, i.e. 
![(AG)X=B](https://latex.codecogs.com/svg.latex?\Large&space;\sum_{w,q}A_{iw}G_{jq}X_{wq}=B_{ij}) 

G is optional, by default (`G=None`) it is identity, in which case this is more like the conventional Spooktroscopy, i.e. to solve <img src="https://render.githubusercontent.com/render/math?math=AX=B"> under regularizations. 

The reason for G is to accommodate the combination with pBASEX, in which case this is solving the two linear inversions in one step.

### Regularizations
Common regularizations are the following three types, all of which optional, depending on what _a prior_ knowledge one wants to enforce on the problem solving.

1. Nonnegativity: <img src="https://render.githubusercontent.com/render/math?math=X\succeq 0">
2. Sparsity: To penalize on <img src="https://render.githubusercontent.com/render/math?math=\|X\|_1"> or <img src="https://render.githubusercontent.com/render/math?math=\|X\|_2^2">
3. Smoothness: To penalize on roughness of X , along the two indices, independently. For <img src="https://render.githubusercontent.com/render/math?math=\omega">-axis, which is the photon energy axis, the form is fixed <img src="https://render.githubusercontent.com/render/math?math=\|(L_{N_w}\otimes I)X\|_2^2"> where <img src="https://render.githubusercontent.com/render/math?math=L_{N_w}"> is the laplacian. Roughness along the second axis of X is customizable through parameter `Bsmoother`, which by default is laplacian squared too.

Sparsity and Smoothness are enforced through penalties in this package, and the penalties are weighted by hyperparameters `lsparse` and `lsmooth` in the total objective function.

## Solvers

Different combinations of regularizations can lead to different forms of objective function. Solvers in package always formalize the specific problem into either a [Quadratic Programming](https://en.wikipedia.org/wiki/Quadratic_programming) or a linear equation. 

| Nonnegativity | Sparsity            | Smoothness | Solver              | Notes                                                        |
| ------------- | ------------------- | ---------- | ------------------- | ------------------------------------------------------------ |
| True          | L1 or False         | Quadratic  | `SpookPosL1`        | This solver can serve tasks like in [Li _et al_](https://iopscience.iop.org/article/10.1088/1361-6455/abcdf1) |
| True          | L2 squared          | Quadratic  | Not implemented yet |                                                              |
| False         | L2 squared or False | Quadratic  | `SpookLinSolve`     | This solver is so far the work-horse for SpookVMI            |
| False         | L1                  | Quadratic  | Not implemented yet |                                                              |



### Quadratic Programming

For cases where it can be formalized into a [Quadratic Programming](https://en.wikipedia.org/wiki/Quadratic_programming) , [OSQP](https://osqp.org) does the job. Thus the root numerical method is alternating direction method of multipliers (ADMM). Looking into the [solver settings of OSQP](https://osqp.org/docs/interfaces/solver_settings.html) is always encouraged, but the default settings usually work fine for `spook` . If one needs to pass in settings, the OSQP solver is `SpookQPBase._prob` .

### Linear Equation

A rare case that it can be formalized into a linear equation is the third line in the table above: no nonnegativity constraint, and the sparsity is L2 norm squared. This is implemented in `SpookLinSolve` , which calls `numpy.linalg.solve` or `scipy.sparse.linalg.spsolve` .


## Normalization Convention

The entries in $A^TA, G^TG$ are preferred to be on the order of magnitude $10^0$, because regularization-related quadratic form matrices have their entries around unity. The scale factors are set as

![normalization](https://latex.codecogs.com/svg.latex?\Large&space;s_a=\sqrt{\frac{1}{N_w}\mathrm{tr}(A^TA)},s_g=\sqrt{\frac{1}{N_q}\mathrm{tr}(G^TG)})

where $N_w, N_q$ are the dimensions along w-axis and q-axis, respectively.

The entries in $B$ are not always accessible, because of the option to pass in precontracted results and run with `mode='contracted'`. Therefore nothing in either $B$ or $(A\otimes G)^TB$ is normalized.

## Unit Tests

`unittest/testPosL1.py` is a good example to play with `SpookPosL1`.



## Dependencies

numpy > 1.19

scipy > 1.7

osqp > 0.6.2

## Acknowledgement
This work was supported by the U.S. Department of Energy (DOE), Office of Science, Office of Basic Energy Sciences (BES), Chemical Sciences, Geosciences, and Biosciences Division (CSGB).

test
