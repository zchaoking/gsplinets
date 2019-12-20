# l = gsplinets.layers.ConvGG( inputs, N_out, kernel_size, h_grid, opt_args )

---

````
l = gsplinets.layers.ConvGG( 
    inputs, 
    N_out, 
    kernel_size, 
    h_grid, 
    opt_args )
````

---

Creates a group convolution layer object that contains the result of convolving the input tensor (inputs) via a set of transformed G convolution kernels that are expanded in B-splines. 

---

## Input arguments

Main attributes of the generated layer object are `l.outputs` and `l.kernels`, see details below. The class takes the following mandatory inputs:

* `inputs` is a ND tensorflow tensor corresponding to G-feature map. It's dimension N should therefore correspond to N = 1 + n + m + 1, where the first 1 is the batch dimension, the n is the spatial dimension, the m is grid dimension of the H part (usually 1, which corresponds to the h_grid), and the last 1 is the feature channel dimension. If inputs is a 5D input tensor, then it is recognized as batch of G feature maps with dimensions [B,X1,X2,H,C], with B for batch size, X1 and X2 the spatial dimensions, H the nr of transformations sampled, and C the number of input channels.
* `N_out` is an integer. It specifies the number of output channels.
* `kernel_size` is an integer. It specifies the spatial kernel size. E.g., with `kernel_size=5` convolutions (on R^2) are done with 5x5 kernels.
* `h_grid` is a grid object generated by the group class whose main attribute is h_grid.grid, which is a tf tensor of size [N,m] with N the number of grid points and m the dimension of the sub-group parameterization. See gsplines/group for more details. This is the grid on which the output is sampled on the H part of the feature map's domain G = R^n \rtimes H.

and has the following optinal inputs regarding the spatial part of the B-spline basis functions

* `xx_basis_size=None` is an integer. By default `xx_basis_size=kernel_size`, i.e., one basis function for each pixel. It specifies the number of basis functions along each axis. The basis is always symmetric in each axis. If an odd nr of basis functions is specified then one is centered at the origin and an equal amount of basis functions on either side. E.g. with `xx_basis_size=3` (and `xx_scale=1`) the centers will be at [-1,0,1]. When the number of basis functions is even then, e.g. when `xx_basis_size=4` (and `xx_scale=1`), then the centers will be at [-1.5,-0.5,0.5,1.5].
* `xx_basis_scale=1` is a float. It specifies the distances between neighbouring splines.
* `xx_basis_type='B_2'` is a string. Currently only strings of type 'B_o', with 'o' the order of the B-splines are supported.
* `xx_basis_mask=False` is a boolean for whether or not to force the xx_centers to be within a disk mask of radius floor(kernel_size/2). This is recommended for SE(2) networks to reduce axis bias.

and has the following optinal inputs regarding the sub-group H part of the B-spline basis functions

* `h_basis_size=None` is an integer. It specifies the number of basis functions to use to cover H. See also gsplinets.group.G.grid.
* `h_basis_scale=1` is a float. It specifies the distance (w.r.t. to the `Log`-map) between neihbouring spline centers in H.
* `h_basis_type='B_2'` is a string. Currently only strings of type 'B_o', with 'o' the order of the B-splines are supported.
* `h_basis_local=False` specifies whether the basis should uniformly cover H (False), or if localized convolution kernels should be used (True). See gsplinets.group.G.grid.
* `input_h_grid=None`. If the grid on the H part of the input tensor is different then that of the specified output grid (see option `h_grid`), then it should be specified with this option. The option value should be a grid object. See gsplinets.group.G.grid.

and the following generic optional inputs

* `stride=1` is an integer. It specifies the spatial stride.
* `padding='VALID'` is a string that specifies whether or not padding is used. With `padding='VALID'` no padding is used (the input tensor size will be reduce), with `padding='SAME' zero padding will be used s.t. the spatial output size stays the same.
* `name=None` is a string. It gives the generated variables a name. With `name=None` a unique random name will be generated.

---

## Layer attributes

The generated object has the following main attributes:

* `l.outputs` returns the output tensor as a tensorflow tensor. The dimensionality of the tensor will be the same as the input [B,X1',X2',...,Cout], with B the batch size, the Xi' the new spatial dimensions (see also padding options), and Cout the number of feature maps specified by the input argument `N_out`.
* `l.kernels()` returns the sampled convolution kernels as a tensorflow tensor. The shape of the kernels will be [K1,K2,...,Kn,C,Cout], with Ki the kernel size (see `kernel_size` input), C the number of input feature channels (derived from `inputs`) and Cout the number of output channels (specified in `N_out`).
* `l.kernels(h)` returns the sampled convolution kernels which are transformed by group element h (a 1D tensorflow array that specifies the parameterization in H).
* `l.xx_centers` the locations on which the B-splines are centered in R^n.
* `l.weights` the corresponding weights of the B-splines. **These are the actual weights over which is optimized**.
* `l.N_k` the number of basis functions used.