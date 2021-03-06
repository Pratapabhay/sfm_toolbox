Here are some examples of 3d usage of the toolbox.

Homographies
############

So far, the toolbox only deals with homographies with 2D geometry. demoSfm(1) and demoSfm(2) are examples on how to use the toolbox to recover homographies. Let's go over demoSfm(2).

demoSfm(2) considers 100 random 2D points in 5 views, some of which being hidden.

Let's create an empty Animation and some random 2D points:

.. code-block:: matlab

   anim=Animation(); anim.W=zeros(2,100,5);
   X=rand(2,100);

Now, let's create a random mask for the point and apply a random homography for each of those views:

.. code-block:" matlab

  for i=1:5
    anim.mask(:,i)=logical(rand(100,1)>0.5);
    if i>=1; H=rand(3,3); else H=eye(3); end
    anim.W(:,anim.mask(:,i),i)=normalizePoint(H*normalizePoint( ...
      X(:,anim.mask(:,i)),-3),3);
  end

Let's add some noise to the measurements:

.. code-block:: matlab

  anim=anim.addNoise('noiseW', 0.01, 'doFillW', true);

Now, let's compute a homography between each view and the first one.

.. code-block:: matlab

  anim.P=zeros(3,3,anim.nFrame);
  for i=1:5
    mask=anim.mask(:,1) & anim.mask(:,i);
    anim.P(:,:,i)=computeHomography( anim.W(:,mask,1), anim.W(:,mask,i), 1 );
  end

And we can now apply bundle adjustment to refine all those transformations (and compare the error before and after):

.. code-block:: matlab

  err = anim.computeError()
  [ anim ] = bundleAdjustment( anim );
  err = anim.computeError()
