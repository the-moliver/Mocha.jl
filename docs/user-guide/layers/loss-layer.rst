Loss Layers
~~~~~~~~~~~

.. class:: MultinomialLogisticLossLayer

   The multinomial logistic loss is defined as :math:`\ell = -w_g\log(x_g)`, where
   :math:`x_1,\ldots,x_C` are probabilities for each of the :math:`C` classes
   conditioned on the input data, :math:`g` is the corresponding
   ground-truth category, and :math:`w_g` is the *weight* for the :math:`g`-th
   class (default 1, see bellow).

   If the conditional probability blob is of the shape ``(dim1, dim2, ...,
   dim_channel, ..., dimN)``, then the ground-truth blob should be of the shape
   ``(dim1, dim2, ..., 1, ..., dimN)``. Here ``dim_channel``, historically called
   the "channel" dimension, is the user specified tensor dimension to compute
   loss on. This general case allows to produce multiple labels for each
   sample. For the typical case where only one (multi-class) label is produced
   for one sample, the conditional probability blob is the shape ``(dim_channel,
   dim_num)`` and the ground-truth blob should be of the shape ``(1, dim_num)``.

   The ground-truth should be a **zero-based** index in the range of
   :math:`0,\ldots,C-1`.

   .. attribute:: bottoms

      Should be a vector containing two symbols. The first one specifies the
      name for the conditional probability input blob, and the second one
      specifies the name for the ground-truth input blob.

   .. attribute:: weights

      This can be used to specify weights for different classes. The following
      values are allowed

      * Empty array (default). This means each category should be equally
        weighted.
      * A 3D tensor of the shape (width, height, channels). Here the (w,h,c)
        entry indicates the weights for category c at location (w,h).
      * A 1D vector of length ``channels``. When both width and height are 1,
        this is equivalent to the case above. Otherwise, the weight vector
        across channels is repeated at every location (w,h).

   .. attribute:: dim

      Default ``-2`` (penultimate). Specify the dimension to operate on.

   .. attribute:: normalize

      Indicating how weights should be normalized if given. The following values
      are allowed

      * ``:local`` (default): Normalize the weights locally at each location
        (w,h), across the channels.
      * ``:global``: Normalize the weights globally.
      * ``:no``: Do not normalize the weights.

      The weights normalization are done in a way that you get the same
      objective function when specifying *equal weights* for each class as when
      you do not specify any weights. In other words, the total sum of the
      weights are scaled to be equal to weights x height x channels. If you
      specify ``:no``, it is your responsibility to properly normalize the
      weights.


.. class:: SoftmaxLossLayer

   This is essentially a combination of :class:`MultinomialLogisticLossLayer`
   and :class:`SoftmaxLayer`. The given predictions :math:`x_1,\ldots,x_C` for
   the :math:`C` classes are transformed with a softmax function

   .. math::

      \sigma(x_1,\ldots,x_C) = (\sigma_1,\ldots,\sigma_C) = \left(\frac{e^{x_1}}{\sum_j
      e^{x_j}},\ldots,\frac{e^{x_C}}{\sum_je^{x_j}}\right)

   which essentially turn the predictions into non-negative values with
   exponential function and then re-normalize to make them look like
   probabilties. Then the transformed values are used to compute the multinomial
   logsitic loss as

   .. math::

      \ell = -w_g \log(\sigma_g)

   Here :math:`g` is the ground-truth label, and :math:`w_g` is the weight for
   the :math:`g`-th category. See the document of :class:`MultinomialLogisticLossLayer` for more
   details on what the weights mean and how to specify them.

   The shapes of the inputs are the same as for the :class:`MultinomialLogisticLossLayer`:
   the multi-class predictions are assumed to be along the channel dimension.

   The reason we provide a combined softmax loss layer instead of using one softmax
   layer and one multinomial logistic layer is that the combined layer produces
   the back-propagation error in a more numerically robust way.

   .. math::

      \frac{\partial \ell}{\partial x_i} = w_g\left(\frac{e^{x_i}}{\sum_j e^{x_j}}
      - \delta_{ig}\right) = w_g\left(\sigma_i - \delta_{ig}\right)

   Here :math:`\delta_{ig}` is 1 if :math:`i=g`, and 0 otherwise.

   .. attribute:: bottoms

      Should be a vector containing two symbols. The first one specifies the
      name for the conditional probability input blob, and the second one
      specifies the name for the ground-truth input blob.

   .. attribute:: dim

      Default ``-2`` (penultimate). Specify the dimension to operate on. For
      a 4D vision tensor blob, the default value (penultimate) translates to the
      3rd tensor dimension, usually called the "channel" dimension.

   .. attribute::
      weights
      normalize

      Properties for the underlying :class:`MultinomialLogisticLossLayer`. See
      its documentation for details.

.. class:: SquareLossLayer

   Compute the square loss for real-valued regression problems:

   .. math::

      \frac{1}{2N}\sum_{i=1}^N \|\mathbf{y}_i - \hat{\mathbf{y}}_i\|^2

   Here :math:`N` is the batch-size, :math:`\mathbf{y}_i` is the real-valued
   (vector or scalar) ground-truth label of the :math:`i`-th sample, and
   :math:`\hat{\mathbf{y}}_i` is the corresponding prediction.

   .. attribute:: bottoms

      Should be a vector containing two symbols. The first one specifies the
      name for the prediction :math:`\hat{\mathbf{y}}`, and the second one
      specifies the name for the ground-truth :math:`\mathbf{y}`.
