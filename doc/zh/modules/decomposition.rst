.. _decompositions:


=================================================================
Decomposing signals in components (matrix factorization problems)
=================================================================

.. currentmodule:: sklearn.decomposition


.. _PCA:


Principal component analysis (PCA)
==================================

Exact PCA and probabilistic interpretation
------------------------------------------

PCA is used to decompose a multivariate dataset in a set of successive
orthogonal components that explain a maximum amount of the variance. In
scikit-learn, :class:`PCA` is implemented as a *transformer* object
that learns :math:`n` components in its ``fit`` method, and can be used on new
data to project it on these components.

The optional parameter ``whiten=True`` makes it possible to
project the data onto the singular space while scaling each component
to unit variance. This is often useful if the models down-stream make
strong assumptions on the isotropy of the signal: this is for example
the case for Support Vector Machines with the RBF kernel and the K-Means
clustering algorithm.

Below is an example of the iris dataset, which is comprised of 4
features, projected on the 2 dimensions that explain most variance:

.. figure:: ../auto_examples/decomposition/images/sphx_glr_plot_pca_vs_lda_001.png
    :target: ../auto_examples/decomposition/plot_pca_vs_lda.html
    :align: center
    :scale: 75%


The :class:`PCA` object also provides a
probabilistic interpretation of the PCA that can give a likelihood of
data based on the amount of variance it explains. As such it implements a
`score` method that can be used in cross-validation:

.. figure:: ../auto_examples/decomposition/images/sphx_glr_plot_pca_vs_fa_model_selection_001.png
    :target: ../auto_examples/decomposition/plot_pca_vs_fa_model_selection.html
    :align: center
    :scale: 75%


.. topic:: Examples:

    * :ref:`sphx_glr_auto_examples_decomposition_plot_pca_vs_lda.py`
    * :ref:`sphx_glr_auto_examples_decomposition_plot_pca_vs_fa_model_selection.py`


.. _IncrementalPCA:

Incremental PCA
---------------

The :class:`PCA` object is very useful, but has certain limitations for
large datasets. The biggest limitation is that :class:`PCA` only supports
batch processing, which means all of the data to be processed must fit in main
memory. The :class:`IncrementalPCA` object uses a different form of
processing and allows for partial computations which almost
exactly match the results of :class:`PCA` while processing the data in a
minibatch fashion. :class:`IncrementalPCA` makes it possible to implement
out-of-core Principal Component Analysis either by:

 * Using its ``partial_fit`` method on chunks of data fetched sequentially
   from the local hard drive or a network database.

 * Calling its fit method on a memory mapped file using ``numpy.memmap``.

:class:`IncrementalPCA` only stores estimates of component and noise variances,
in order update ``explained_variance_ratio_`` incrementally. This is why
memory usage depends on the number of samples per batch, rather than the
number of samples to be processed in the dataset.

.. figure:: ../auto_examples/decomposition/images/sphx_glr_plot_incremental_pca_001.png
    :target: ../auto_examples/decomposition/plot_incremental_pca.html
    :align: center
    :scale: 75%

.. figure:: ../auto_examples/decomposition/images/sphx_glr_plot_incremental_pca_002.png
    :target: ../auto_examples/decomposition/plot_incremental_pca.html
    :align: center
    :scale: 75%


.. topic:: Examples:

    * :ref:`sphx_glr_auto_examples_decomposition_plot_incremental_pca.py`


.. _RandomizedPCA:

PCA using randomized SVD
------------------------

It is often interesting to project data to a lower-dimensional
space that preserves most of the variance, by dropping the singular vector
of components associated with lower singular values.

For instance, if we work with 64x64 pixel gray-level pictures
for face recognition,
the dimensionality of the data is 4096 and it is slow to train an
RBF support vector machine on such wide data. Furthermore we know that
the intrinsic dimensionality of the data is much lower than 4096 since all
pictures of human faces look somewhat alike.
The samples lie on a manifold of much lower
dimension (say around 200 for instance). The PCA algorithm can be used
to linearly transform the data while both reducing the dimensionality
and preserve most of the explained variance at the same time.

The class :class:`PCA` used with the optional parameter
``svd_solver='randomized'`` is very useful in that case: since we are going
to drop most of the singular vectors it is much more efficient to limit the
computation to an approximated estimate of the singular vectors we will keep
to actually perform the transform.

For instance, the following shows 16 sample portraits (centered around
0.0) from the Olivetti dataset. On the right hand side are the first 16
singular vectors reshaped as portraits. Since we only require the top
16 singular vectors of a dataset with size :math:`n_{samples} = 400`
and :math:`n_{features} = 64 \times 64 = 4096`, the computation time is
less than 1s:

.. |orig_img| image:: ../auto_examples/decomposition/images/sphx_glr_plot_faces_decomposition_001.png
   :target: ../auto_examples/decomposition/plot_faces_decomposition.html
   :scale: 60%

.. |pca_img| image:: ../auto_examples/decomposition/images/sphx_glr_plot_faces_decomposition_002.png
   :target: ../auto_examples/decomposition/plot_faces_decomposition.html
   :scale: 60%

.. centered:: |orig_img| |pca_img|

Note: with the optional parameter ``svd_solver='randomized'``, we also
need to give :class:`PCA` the size of the lower-dimensional space
``n_components`` as a mandatory input parameter.

If we note :math:`n_{\max} = \max(n_{\mathrm{samples}}, n_{\mathrm{features}})` and
:math:`n_{\min} = \min(n_{\mathrm{samples}}, n_{\mathrm{features}})`, the time complexity
of the randomized :class:`PCA` is :math:`O(n_{\max}^2 \cdot n_{\mathrm{components}})`
instead of :math:`O(n_{\max}^2 \cdot n_{\min})` for the exact method
implemented in :class:`PCA`.

The memory footprint of randomized :class:`PCA` is also proportional to
:math:`2 \cdot n_{\max} \cdot n_{\mathrm{components}}` instead of :math:`n_{\max}
\cdot n_{\min}` for the exact method.

Note: the implementation of ``inverse_transform`` in :class:`PCA` with
``svd_solver='randomized'`` is not the exact inverse transform of
``transform`` even when ``whiten=False`` (default).


.. topic:: Examples:

    * :ref:`sphx_glr_auto_examples_applications_plot_face_recognition.py`
    * :ref:`sphx_glr_auto_examples_decomposition_plot_faces_decomposition.py`

.. topic:: References:

    * `"Finding structure with randomness: Stochastic algorithms for
      constructing approximate matrix decompositions"
      <http://arxiv.org/abs/0909.4061>`_
      Halko, et al., 2009


.. _kernel_PCA:

Kernel PCA（内核 PCA）
---------------------

:class:`KernelPCA` 是 PCA 的扩展，通过使用内核实现非线性 dimensionality reduction（降维） (参阅 :ref:`metrics`)。它具有许多应用，包括 denoising（去噪）, compression（压缩） 和 structured prediction（结构预测） (kernel dependency estimation（内核依赖估计）)。 :class:`KernelPCA` 支持 ``transform`` 和 ``inverse_transform`` 。

.. figure:: ../auto_examples/decomposition/images/sphx_glr_plot_kernel_pca_001.png
    :target: ../auto_examples/decomposition/plot_kernel_pca.html
    :align: center
    :scale: 75%

.. topic:: 示例:

    * :ref:`sphx_glr_auto_examples_decomposition_plot_kernel_pca.py`


.. _SparsePCA:

稀疏主成分分析（Sparse principal components analysis） (SparsePCA 和 MiniBatchSparsePCA)
---------------------------------------------------------------------------------------

:class:`SparsePCA` 是 PCA 的一个变体，目的是提取能重建数据的 sparse components （稀疏组件）集合。

Mini-batch sparse PCA（小批量稀疏 PCA） (:class:`MiniBatchSparsePCA`) 是一个 :class:`SparsePCA` 的变种，速度更快，但不太准确。通过迭代一组特征的 small chunks （小块）来达到增加的速度，对于给定的迭代次数。


Principal component analysis（主成分分析） (:class:`PCA`) 的缺点在于，通过该方法提取的成分具有唯一的密集表达式，即当表示为原始变量的线性组合时，它们具有非零系数。这可以使解释变得困难。在许多情况下，真正的基础组件可以更自然地想象为稀疏向量; 例如在面部识别中，组件可能自然地映射到面部的部分。

Sparse principal components（稀疏的主成分）产生更简洁，可解释的表示，明确强调哪些 original features （原始特征）有助于样本之间的差异。

以下示例说明了使用 Olivetti faces dataset 中的稀疏 PCA 提取的 16 个 components （组件）。可以看出 regularization term （正则化术语）如何引发许多零。此外，数据的 natural structure （自然结构）导致非零系数 vertically adjacent （垂直相邻）。该模型不会在数学上强制执行: 每个 component （组件）都是一个向量  :math:`h \in \mathbf{R}^{4096}`, 没有 vertical adjacency （垂直相邻性）的概念，除了人类友好的可视化视图为 64x64 像素图像。下面显示的组件出现局部的事实是数据的固有结构的影响，这使得这种局部模式使重建误差最小化。存在考虑到邻接和不同类型结构的稀疏诱导规范; 参见 [Jen09]_ 对这种方法进行审查。
有关如何使用稀疏 PCA 的更多详细信息，请参阅下面的示例部分。


.. |spca_img| image:: ../auto_examples/decomposition/images/sphx_glr_plot_faces_decomposition_005.png
   :target: ../auto_examples/decomposition/plot_faces_decomposition.html
   :scale: 60%

.. centered:: |pca_img| |spca_img|

请注意，稀疏 PCA 问题有许多不同的配方。这里实行的是基于 [Mrl09]_ 。解决的优化问题是一个 PCA 问题（dictionary learning（字典学习）），具有 :math:`\ell_1` 对 components （组件）的 penalty （惩罚）:

.. math::
   (U^*, V^*) = \underset{U, V}{\operatorname{arg\,min\,}} & \frac{1}{2}
                ||X-UV||_2^2+\alpha||V||_1 \\
                \text{subject to\,} & ||U_k||_2 = 1 \text{ for all }
                0 \leq k < n_{components}


sparsity-inducing（稀疏性诱导） :math:`\ell_1` 规范也可以避免学习成分的噪音降低，而 training samples （训练样本）很少。可以通过超参数 ``alpha`` 来调整惩罚程度（从而减少稀疏度）。小值导致了温和的正则化因式分解，而较大的值将许多系数缩小到零。

.. note::

  虽然本着在线算法的精神， :class:`MiniBatchSparsePCA` 类不实现 ``partial_fit`` , 因为算法沿特征方向在线，而不是样本方向。

.. topic:: 示例:

   * :ref:`sphx_glr_auto_examples_decomposition_plot_faces_decomposition.py`

.. topic:: 参考文献:

  .. [Mrl09] `"Online Dictionary Learning for Sparse Coding"
     <http://www.di.ens.fr/sierra/pdfs/icml09.pdf>`_
     J. Mairal, F. Bach, J. Ponce, G. Sapiro, 2009
  .. [Jen09] `"Structured Sparse Principal Component Analysis"
     <www.di.ens.fr/~fbach/sspca_AISTATS2010.pdf>`_
     R. Jenatton, G. Obozinski, F. Bach, 2009


.. _LSA:

Truncated singular value decomposition and latent semantic analysis（截断奇异值分解和潜在语义分析）
===============================================================================================

:class:`TruncatedSVD` 实现了一个奇异值分解（SVD）的变体，它只能计算 :math:`k` 最大的奇异值，其中 :math:`k` 是用户指定的参数。

当 truncated SVD （截断的 SVD） 被应用于术语文档矩阵（由 ``CountVectorizer`` 或 ``TfidfVectorizer`` 返回）时，这种转换被称为 `latent semantic analysis <http://nlp.stanford.edu/IR-book/pdf/18lsi.pdf>`_ (LSA), 因为它将这样的矩阵转换为低纬度的 "semantic（语义）" 空间。
特别地， LSA 已知能够抵抗同义词和多义词的影响（两者大致意味着每个单词有多重含义），这导致术语文档矩阵过度稀疏，并且在诸如余弦相似性的度量下表现出差的相似性。

.. note::
    LSA 也被称为潜在语义索引 LSI，尽管严格地说它是指在 persistent indexes （持久索引）中用于 information retrieval （信息检索）的目的。

数学表示中， truncated SVD 应用于训练样本 :math:`X` 产生一个 low-rank approximation （低阶逼近于） :math:`X`:

.. math::
    X \approx X_k = U_k \Sigma_k V_k^\top

在这个操作之后，:math:`U_k \Sigma_k^\top` 是转换后的训练集，其中包括 :math:`k` 个特征（在 API 中被称为 ``n_components`` ）。

还需要转换一个测试集 :math:`X`, 我们乘以 :math:`V_k`: 

.. math::
    X' = X V_k

.. note::
    自然语言处理(NLP) 和信息检索(IR) 文献中的 LSA 的大多数处理方式交换 axes of the :math:`X` matrix （:math:`X` 矩阵的轴）,使其具有形状 ``n_features`` × ``n_samples`` 。
    我们以 scikit-learn API 相匹配的不同方式呈现 LSA, 但是找到的奇异值是相同的。

:class:`TruncatedSVD` 非常类似于 :class:`PCA`, 但不同之处在于它适用于示例矩阵 :math:`X` 而不是它们的协方差矩阵。
当从特征值中减去 :math:`X` 的列（每个特征）手段时，得到的矩阵上的 truncated SVD 相当于 PCA 。
实际上，这意味着 :class:`TruncatedSVD` transformer（转换器）接受 ``scipy.sparse`` 矩阵，而不需要对它们进行加密，因为即使对于中型文档集合，densifying （密集）也可能填满内存。

虽然 :class:`TruncatedSVD` transformer（转换器）与任何（sparse（稀疏））特征矩阵一起工作，建议在 LSA/document 处理设置中使用它在 tf–idf 矩阵上的原始频率计数。
特别地，应该打开 sublinear scaling （子线性缩放）和 inverse document frequency （逆文档频率） (``sublinear_tf=True, use_idf=True``) 以使特征值更接近于高斯分布，补偿 LSA 对文本数据的错误假设。

.. topic:: 示例:

   * :ref:`sphx_glr_auto_examples_text_document_clustering.py`

.. topic:: 参考文献:

  * Christopher D. Manning, Prabhakar Raghavan and Hinrich Schütze (2008),
    *Introduction to Information Retrieval*, Cambridge University Press,
    chapter 18: `Matrix decompositions & latent semantic indexing
    <http://nlp.stanford.edu/IR-book/pdf/18lsi.pdf>`_


.. _DictionaryLearning:

Dictionary learning（字典学习）
=============================

.. _SparseCoder:

Sparse coding with a precomputed dictionary（稀疏编码与预先计算的字典）
--------------------------------------------------------------------

:class:`SparseCoder` 对象是一个 estimator（估计器），可以用来将信号转换成固定的预先计算的字典的 sparse linear combination （稀疏线性组合），如 discrete wavelet basis 。因此，该对象不实现 ``fit`` 方法。该转换相当于 sparse coding problem （稀疏编码问题）: 将数据的表示尽可能少的 dictionary atoms （字典原子）的线性组合。字典学习的所有变体实现以下变换方法，可以通过 ``transform_method`` 初始化参数进行控制: 

* Orthogonal matching pursuit(正交匹配 pursuit ) (:ref:`omp`)

* Least-angle regression (最小角度回归)(:ref:`least_angle_regression`)

* Lasso computed by least-angle regression(Lasso 通过最小角度回归计算)

* Lasso using coordinate descent (Lasso 使用梯度下降)(:ref:`lasso`)

* Thresholding(阈值)

Thresholding （阈值）非常快，但是不能产生精确的 reconstructions（重构）。
它们在分类任务的文献中已被证明是有用的。对于 image reconstruction tasks （图像重构任务）， orthogonal matching pursuit 产生最精确，unbiased（无偏）的重构。 

dictionary learning（字典学习）对象通过 ``split_code`` 参数提供分类稀疏编码结果中的正值和负值的可能性。当使用 dictionary learning （字典学习）来提取将用于监督学习的特征时，这是有用的，因为它允许学习算法将不同的权重分配给 particular atom （特定原子）的 negative loadings （负的负荷），从相应的 positive loading （正加载）。

split code for a single sample（单个样本的分割代码）具有长度 ``2 * n_components`` ，并使用以下规则构造: 首先，计算长度为 ``n_components`` 的常规代码。然后， ``split_code`` 的第一个 ``n_components`` 条目将用正常代码向量的正部分填充。分割代码的下半部分填充有代码矢量的负部分，只有一个正号。因此， split_code 是非负的。 


.. topic:: 示例:

    * :ref:`sphx_glr_auto_examples_decomposition_plot_sparse_coding.py`


Generic dictionary learning（通用字典学习）
-----------------------------------------

Dictionary learning（字典学习） (:class:`DictionaryLearning`) 是一个矩阵因式分解问题，相当于找到一个（通常是不完整的）字典，它将在拟合数据的稀疏编码中表现良好。

将数据表示为来自 overcomplete dictionary（过度完整字典）的稀疏组合的原子被认为是  mammal primary visual cortex works（哺乳动物初级视觉皮层的工作方式）. 
因此，应用于 image patches （图像补丁）的 dictionary learning （字典学习）已被证明在诸如 image completion ，inpainting（修复） and denoising（去噪）以及监督识别的图像处理任务中给出良好的结果。

Dictionary learning（字典学习）是通过交替更新稀疏代码来解决的优化问题，作为解决多个 Lasso 问题的一个解决方案，考虑到 dictionary fixed （字典固定），然后更新字典以最适合 sparse code （稀疏代码）。

.. math::
   (U^*, V^*) = \underset{U, V}{\operatorname{arg\,min\,}} & \frac{1}{2}
                ||X-UV||_2^2+\alpha||U||_1 \\
                \text{subject to\,} & ||V_k||_2 = 1 \text{ for all }
                0 \leq k < n_{\mathrm{atoms}}


.. |pca_img2| image:: ../auto_examples/decomposition/images/sphx_glr_plot_faces_decomposition_002.png
   :target: ../auto_examples/decomposition/plot_faces_decomposition.html
   :scale: 60%

.. |dict_img2| image:: ../auto_examples/decomposition/images/sphx_glr_plot_faces_decomposition_006.png
   :target: ../auto_examples/decomposition/plot_faces_decomposition.html
   :scale: 60%

.. centered:: |pca_img2| |dict_img2|


在使用这样一个过程来 fit the dictionary （拟合字典）之后，变换只是一个稀疏的编码步骤，与所有的字典学习对象共享相同的实现。(参见 :ref:`SparseCoder`)。

以下图像显示从 raccoon face （浣熊脸部）图像中提取的 4x4 像素图像补丁中学习的字典如何。


.. figure:: ../auto_examples/decomposition/images/sphx_glr_plot_image_denoising_001.png
    :target: ../auto_examples/decomposition/plot_image_denoising.html
    :align: center
    :scale: 50%


.. topic:: 示例:

  * :ref:`sphx_glr_auto_examples_decomposition_plot_image_denoising.py`


.. topic:: 参考文献:

  * `"Online dictionary learning for sparse coding"
    <http://www.di.ens.fr/sierra/pdfs/icml09.pdf>`_
    J. Mairal, F. Bach, J. Ponce, G. Sapiro, 2009

.. _MiniBatchDictionaryLearning:

Mini-batch dictionary learning
------------------------------

:class:`MiniBatchDictionaryLearning` implements a faster, but less accurate
version of the dictionary learning algorithm that is better suited for large
datasets.

By default, :class:`MiniBatchDictionaryLearning` divides the data into
mini-batches and optimizes in an online manner by cycling over the mini-batches
for the specified number of iterations. However, at the moment it does not
implement a stopping condition.

The estimator also implements ``partial_fit``, which updates the dictionary by
iterating only once over a mini-batch. This can be used for online learning
when the data is not readily available from the start, or for when the data
does not fit into the memory.

.. currentmodule:: sklearn.cluster

.. image:: ../auto_examples/cluster/images/sphx_glr_plot_dict_face_patches_001.png
    :target: ../auto_examples/cluster/plot_dict_face_patches.html
    :scale: 50%
    :align: right

.. topic:: **Clustering for dictionary learning**

   Note that when using dictionary learning to extract a representation
   (e.g. for sparse coding) clustering can be a good proxy to learn the
   dictionary. For instance the :class:`MiniBatchKMeans` estimator is
   computationally efficient and implements on-line learning with a
   ``partial_fit`` method.

    Example: :ref:`sphx_glr_auto_examples_cluster_plot_dict_face_patches.py`

.. currentmodule:: sklearn.decomposition

.. _FA:

Factor Analysis
===============

In unsupervised learning we only have a dataset :math:`X = \{x_1, x_2, \dots, x_n
\}`. How can this dataset be described mathematically? A very simple
`continuous latent variable` model for :math:`X` is

.. math:: x_i = W h_i + \mu + \epsilon

The vector :math:`h_i` is called "latent" because it is unobserved. :math:`\epsilon` is
considered a noise term distributed according to a Gaussian with mean 0 and
covariance :math:`\Psi` (i.e. :math:`\epsilon \sim \mathcal{N}(0, \Psi)`), :math:`\mu` is some
arbitrary offset vector. Such a model is called "generative" as it describes
how :math:`x_i` is generated from :math:`h_i`. If we use all the :math:`x_i`'s as columns to form
a matrix :math:`\mathbf{X}` and all the :math:`h_i`'s as columns of a matrix :math:`\mathbf{H}`
then we can write (with suitably defined :math:`\mathbf{M}` and :math:`\mathbf{E}`):

.. math::
    \mathbf{X} = W \mathbf{H} + \mathbf{M} + \mathbf{E}

In other words, we *decomposed* matrix :math:`\mathbf{X}`.

If :math:`h_i` is given, the above equation automatically implies the following
probabilistic interpretation:

.. math:: p(x_i|h_i) = \mathcal{N}(Wh_i + \mu, \Psi)

For a complete probabilistic model we also need a prior distribution for the
latent variable :math:`h`. The most straightforward assumption (based on the nice
properties of the Gaussian distribution) is :math:`h \sim \mathcal{N}(0,
\mathbf{I})`.  This yields a Gaussian as the marginal distribution of :math:`x`:

.. math:: p(x) = \mathcal{N}(\mu, WW^T + \Psi)

Now, without any further assumptions the idea of having a latent variable :math:`h`
would be superfluous -- :math:`x` can be completely modelled with a mean
and a covariance. We need to impose some more specific structure on one
of these two parameters. A simple additional assumption regards the
structure of the error covariance :math:`\Psi`:

* :math:`\Psi = \sigma^2 \mathbf{I}`: This assumption leads to
  the probabilistic model of :class:`PCA`.

* :math:`\Psi = \mathrm{diag}(\psi_1, \psi_2, \dots, \psi_n)`: This model is called
  :class:`FactorAnalysis`, a classical statistical model. The matrix W is
  sometimes called the "factor loading matrix".

Both models essentially estimate a Gaussian with a low-rank covariance matrix.
Because both models are probabilistic they can be integrated in more complex
models, e.g. Mixture of Factor Analysers. One gets very different models (e.g.
:class:`FastICA`) if non-Gaussian priors on the latent variables are assumed.

Factor analysis *can* produce similar components (the columns of its loading
matrix) to :class:`PCA`. However, one can not make any general statements
about these components (e.g. whether they are orthogonal):

.. |pca_img3| image:: ../auto_examples/decomposition/images/sphx_glr_plot_faces_decomposition_002.png
    :target: ../auto_examples/decomposition/plot_faces_decomposition.html
    :scale: 60%

.. |fa_img3| image:: ../auto_examples/decomposition/images/sphx_glr_plot_faces_decomposition_009.png
    :target: ../auto_examples/decomposition/plot_faces_decomposition.html
    :scale: 60%

.. centered:: |pca_img3| |fa_img3|

The main advantage for Factor Analysis (over :class:`PCA` is that
it can model the variance in every direction of the input space independently
(heteroscedastic noise):

.. figure:: ../auto_examples/decomposition/images/sphx_glr_plot_faces_decomposition_008.png
    :target: ../auto_examples/decomposition/plot_faces_decomposition.html
    :align: center
    :scale: 75%

This allows better model selection than probabilistic PCA in the presence
of heteroscedastic noise:

.. figure:: ../auto_examples/decomposition/images/sphx_glr_plot_pca_vs_fa_model_selection_002.png
    :target: ../auto_examples/decomposition/plot_pca_vs_fa_model_selection.html
    :align: center
    :scale: 75%


.. topic:: Examples:

    * :ref:`sphx_glr_auto_examples_decomposition_plot_pca_vs_fa_model_selection.py`

.. _ICA:

Independent component analysis (ICA)
====================================

Independent component analysis separates a multivariate signal into
additive subcomponents that are maximally independent. It is
implemented in scikit-learn using the :class:`Fast ICA <FastICA>`
algorithm. Typically, ICA is not used for reducing dimensionality but
for separating superimposed signals. Since the ICA model does not include
a noise term, for the model to be correct, whitening must be applied.
This can be done internally using the whiten argument or manually using one
of the PCA variants.

It is classically used to separate mixed signals (a problem known as
*blind source separation*), as in the example below:

.. figure:: ../auto_examples/decomposition/images/sphx_glr_plot_ica_blind_source_separation_001.png
    :target: ../auto_examples/decomposition/plot_ica_blind_source_separation.html
    :align: center
    :scale: 60%


ICA can also be used as yet another non linear decomposition that finds
components with some sparsity:

.. |pca_img4| image:: ../auto_examples/decomposition/images/sphx_glr_plot_faces_decomposition_002.png
    :target: ../auto_examples/decomposition/plot_faces_decomposition.html
    :scale: 60%

.. |ica_img4| image:: ../auto_examples/decomposition/images/sphx_glr_plot_faces_decomposition_004.png
    :target: ../auto_examples/decomposition/plot_faces_decomposition.html
    :scale: 60%

.. centered:: |pca_img4| |ica_img4|

.. topic:: Examples:

    * :ref:`sphx_glr_auto_examples_decomposition_plot_ica_blind_source_separation.py`
    * :ref:`sphx_glr_auto_examples_decomposition_plot_ica_vs_pca.py`
    * :ref:`sphx_glr_auto_examples_decomposition_plot_faces_decomposition.py`


.. _NMF:

Non-negative matrix factorization (NMF or NNMF)
===============================================

NMF with the Frobenius norm
---------------------------

:class:`NMF` [1]_ is an alternative approach to decomposition that assumes that the
data and the components are non-negative. :class:`NMF` can be plugged in
instead of :class:`PCA` or its variants, in the cases where the data matrix
does not contain negative values. It finds a decomposition of samples
:math:`X` into two matrices :math:`W` and :math:`H` of non-negative elements,
by optimizing the distance :math:`d` between :math:`X` and the matrix product
:math:`WH`. The most widely used distance function is the squared Frobenius
norm, which is an obvious extension of the Euclidean norm to matrices:

.. math::
    d_{\mathrm{Fro}}(X, Y) = \frac{1}{2} ||X - Y||_{\mathrm{Fro}}^2 = \frac{1}{2} \sum_{i,j} (X_{ij} - {Y}_{ij})^2

Unlike :class:`PCA`, the representation of a vector is obtained in an additive
fashion, by superimposing the components, without subtracting. Such additive
models are efficient for representing images and text.

It has been observed in [Hoyer, 2004] [2]_ that, when carefully constrained,
:class:`NMF` can produce a parts-based representation of the dataset,
resulting in interpretable models. The following example displays 16
sparse components found by :class:`NMF` from the images in the Olivetti
faces dataset, in comparison with the PCA eigenfaces.

.. |pca_img5| image:: ../auto_examples/decomposition/images/sphx_glr_plot_faces_decomposition_002.png
    :target: ../auto_examples/decomposition/plot_faces_decomposition.html
    :scale: 60%

.. |nmf_img5| image:: ../auto_examples/decomposition/images/sphx_glr_plot_faces_decomposition_003.png
    :target: ../auto_examples/decomposition/plot_faces_decomposition.html
    :scale: 60%

.. centered:: |pca_img5| |nmf_img5|


The :attr:`init` attribute determines the initialization method applied, which
has a great impact on the performance of the method. :class:`NMF` implements the
method Nonnegative Double Singular Value Decomposition. NNDSVD [4]_ is based on
two SVD processes, one approximating the data matrix, the other approximating
positive sections of the resulting partial SVD factors utilizing an algebraic
property of unit rank matrices. The basic NNDSVD algorithm is better fit for
sparse factorization. Its variants NNDSVDa (in which all zeros are set equal to
the mean of all elements of the data), and NNDSVDar (in which the zeros are set
to random perturbations less than the mean of the data divided by 100) are
recommended in the dense case.

Note that the Multiplicative Update ('mu') solver cannot update zeros present in
the initialization, so it leads to poorer results when used jointly with the
basic NNDSVD algorithm which introduces a lot of zeros; in this case, NNDSVDa or
NNDSVDar should be preferred.

:class:`NMF` can also be initialized with correctly scaled random non-negative
matrices by setting :attr:`init="random"`. An integer seed or a
``RandomState`` can also be passed to :attr:`random_state` to control
reproducibility.

In :class:`NMF`, L1 and L2 priors can be added to the loss function in order
to regularize the model. The L2 prior uses the Frobenius norm, while the L1
prior uses an elementwise L1 norm. As in :class:`ElasticNet`, we control the
combination of L1 and L2 with the :attr:`l1_ratio` (:math:`\rho`) parameter,
and the intensity of the regularization with the :attr:`alpha`
(:math:`\alpha`) parameter. Then the priors terms are:

.. math::
    \alpha \rho ||W||_1 + \alpha \rho ||H||_1
    + \frac{\alpha(1-\rho)}{2} ||W||_{\mathrm{Fro}} ^ 2
    + \frac{\alpha(1-\rho)}{2} ||H||_{\mathrm{Fro}} ^ 2

and the regularized objective function is:

.. math::
    d_{\mathrm{Fro}}(X, WH)
    + \alpha \rho ||W||_1 + \alpha \rho ||H||_1
    + \frac{\alpha(1-\rho)}{2} ||W||_{\mathrm{Fro}} ^ 2
    + \frac{\alpha(1-\rho)}{2} ||H||_{\mathrm{Fro}} ^ 2

:class:`NMF` regularizes both W and H. The public function
:func:`non_negative_factorization` allows a finer control through the
:attr:`regularization` attribute, and may regularize only W, only H, or both.

NMF with a beta-divergence
--------------------------

As described previously, the most widely used distance function is the squared
Frobenius norm, which is an obvious extension of the Euclidean norm to
matrices:

.. math::
    d_{\mathrm{Fro}}(X, Y) = \frac{1}{2} ||X - Y||_{Fro}^2 = \frac{1}{2} \sum_{i,j} (X_{ij} - {Y}_{ij})^2

Other distance functions can be used in NMF as, for example, the (generalized)
Kullback-Leibler (KL) divergence, also referred as I-divergence:

.. math::
    d_{KL}(X, Y) = \sum_{i,j} (X_{ij} \log(\frac{X_{ij}}{Y_{ij}}) - X_{ij} + Y_{ij})

Or, the Itakura-Saito (IS) divergence:

.. math::
    d_{IS}(X, Y) = \sum_{i,j} (\frac{X_{ij}}{Y_{ij}} - \log(\frac{X_{ij}}{Y_{ij}}) - 1)

These three distances are special cases of the beta-divergence family, with
:math:`\beta = 2, 1, 0` respectively [6]_. The beta-divergence are
defined by :

.. math::
    d_{\beta}(X, Y) = \sum_{i,j} \frac{1}{\beta(\beta - 1)}(X_{ij}^\beta + (\beta-1)Y_{ij}^\beta - \beta X_{ij} Y_{ij}^{\beta - 1})

.. figure:: ../auto_examples/decomposition/images/sphx_glr_plot_beta_divergence_001.png
    :target: ../auto_examples/decomposition/plot_beta_divergence.html
    :align: center
    :scale: 75%

Note that this definition is not valid if :math:`\beta \in (0; 1)`, yet it can
be continously extended to the definitions of :math:`d_{KL}` and :math:`d_{IS}`
respectively.

:class:`NMF` implements two solvers, using Coordinate Descent ('cd') [5]_, and
Multiplicative Update ('mu') [6]_. The 'mu' solver can optimize every
beta-divergence, including of course the Frobenius norm (:math:`\beta=2`), the
(generalized) Kullback-Leibler divergence (:math:`\beta=1`) and the
Itakura-Saito divergence (:math:`\beta=0`). Note that for
:math:`\beta \in (1; 2)`, the 'mu' solver is significantly faster than for other
values of :math:`\beta`. Note also that with a negative (or 0, i.e.
'itakura-saito') :math:`\beta`, the input matrix cannot contain zero values.

The 'cd' solver can only optimize the Frobenius norm. Due to the
underlying non-convexity of NMF, the different solvers may converge to
different minima, even when optimizing the same distance function.

NMF is best used with the ``fit_transform`` method, which returns the matrix W.
The matrix H is stored into the fitted model in the ``components_`` attribute;
the method ``transform`` will decompose a new matrix X_new based on these
stored components::

    >>> import numpy as np
    >>> X = np.array([[1, 1], [2, 1], [3, 1.2], [4, 1], [5, 0.8], [6, 1]])
    >>> from sklearn.decomposition import NMF
    >>> model = NMF(n_components=2, init='random', random_state=0)
    >>> W = model.fit_transform(X)
    >>> H = model.components_
    >>> X_new = np.array([[1, 0], [1, 6.1], [1, 0], [1, 4], [3.2, 1], [0, 4]])
    >>> W_new = model.transform(X_new)

.. topic:: Examples:

    * :ref:`sphx_glr_auto_examples_decomposition_plot_faces_decomposition.py`
    * :ref:`sphx_glr_auto_examples_applications_plot_topics_extraction_with_nmf_lda.py`
    * :ref:`sphx_glr_auto_examples_decomposition_plot_beta_divergence.py`

.. topic:: References:

    .. [1] `"Learning the parts of objects by non-negative matrix factorization"
      <http://www.columbia.edu/~jwp2128/Teaching/W4721/papers/nmf_nature.pdf>`_
      D. Lee, S. Seung, 1999

    .. [2] `"Non-negative Matrix Factorization with Sparseness Constraints"
      <http://www.jmlr.org/papers/volume5/hoyer04a/hoyer04a.pdf>`_
      P. Hoyer, 2004

    .. [4] `"SVD based initialization: A head start for nonnegative
      matrix factorization"
      <http://scgroup.hpclab.ceid.upatras.gr/faculty/stratis/Papers/HPCLAB020107.pdf>`_
      C. Boutsidis, E. Gallopoulos, 2008

    .. [5] `"Fast local algorithms for large scale nonnegative matrix and tensor
      factorizations."
      <http://www.bsp.brain.riken.jp/publications/2009/Cichocki-Phan-IEICE_col.pdf>`_
      A. Cichocki, P. Anh-Huy, 2009

    .. [6] `"Algorithms for nonnegative matrix factorization with the beta-divergence"
      <http://http://arxiv.org/pdf/1010.1763v3.pdf>`_
      C. Fevotte, J. Idier, 2011


.. _LatentDirichletAllocation:

Latent Dirichlet Allocation (LDA)
=================================

Latent Dirichlet Allocation is a generative probabilistic model for collections of
discrete dataset such as text corpora. It is also a topic model that is used for
discovering abstract topics from a collection of documents.

The graphical model of LDA is a three-level Bayesian model:

.. image:: ../images/lda_model_graph.png
   :align: center

When modeling text corpora, the model assumes the following generative process for
a corpus with :math:`D` documents and :math:`K` topics:

  1. For each topic :math:`k`, draw :math:`\beta_k \sim \mathrm{Dirichlet}(\eta),\: k =1...K`

  2. For each document :math:`d`, draw :math:`\theta_d \sim \mathrm{Dirichlet}(\alpha), \: d=1...D`

  3. For each word :math:`i` in document :math:`d`:

    a. Draw a topic index :math:`z_{di} \sim \mathrm{Multinomial}(\theta_d)`
    b. Draw the observed word :math:`w_{ij} \sim \mathrm{Multinomial}(beta_{z_{di}}.)`

For parameter estimation, the posterior distribution is:

.. math::
  p(z, \theta, \beta |w, \alpha, \eta) =
    \frac{p(z, \theta, \beta|\alpha, \eta)}{p(w|\alpha, \eta)}

Since the posterior is intractable, variational Bayesian method
uses a simpler distribution :math:`q(z,\theta,\beta | \lambda, \phi, \gamma)`
to approximate it, and those variational parameters :math:`\lambda`, :math:`\phi`,
:math:`\gamma` are optimized to maximize the Evidence Lower Bound (ELBO):

.. math::
  \log\: P(w | \alpha, \eta) \geq L(w,\phi,\gamma,\lambda) \overset{\triangle}{=}
    E_{q}[\log\:p(w,z,\theta,\beta|\alpha,\eta)] - E_{q}[\log\:q(z, \theta, \beta)]

Maximizing ELBO is equivalent to minimizing the Kullback-Leibler(KL) divergence
between :math:`q(z,\theta,\beta)` and the true posterior
:math:`p(z, \theta, \beta |w, \alpha, \eta)`.

:class:`LatentDirichletAllocation` implements online variational Bayes algorithm and supports
both online and batch update method.
While batch method updates variational variables after each full pass through the data,
online method updates variational variables from mini-batch data points.

.. note::

  Although online method is guaranteed to converge to a local optimum point, the quality of
  the optimum point and the speed of convergence may depend on mini-batch size and
  attributes related to learning rate setting.

When :class:`LatentDirichletAllocation` is applied on a "document-term" matrix, the matrix
will be decomposed into a "topic-term" matrix and a "document-topic" matrix. While
"topic-term" matrix is stored as :attr:`components_` in the model, "document-topic" matrix
can be calculated from ``transform`` method.

:class:`LatentDirichletAllocation` also implements ``partial_fit`` method. This is used
when data can be fetched sequentially.

.. topic:: Examples:

    * :ref:`sphx_glr_auto_examples_applications_plot_topics_extraction_with_nmf_lda.py`

.. topic:: References:

    * `"Latent Dirichlet Allocation"
      <https://www.cs.princeton.edu/~blei/papers/BleiNgJordan2003.pdf>`_
      D. Blei, A. Ng, M. Jordan, 2003

    * `"Online Learning for Latent Dirichlet Allocation”
      <https://www.cs.princeton.edu/~blei/papers/HoffmanBleiBach2010b.pdf>`_
      M. Hoffman, D. Blei, F. Bach, 2010

    * `"Stochastic Variational Inference"
      <http://www.columbia.edu/~jwp2128/Papers/HoffmanBleiWangPaisley2013.pdf>`_
      M. Hoffman, D. Blei, C. Wang, J. Paisley, 2013
