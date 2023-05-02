Download Link: https://assignmentchef.com/product/solved-computer-vision-lab-exercise-4
<br>
<h1>Structure from Motion</h1>

This week, the lab introduces structure from motion – inferring 3D shape from 2D images. Points matches across frames are saved into a matrix and the Tomasi-Kanade factorization is used for performing SfM (Structure from Motion).

<h2>1       Finding point correspondences</h2>

For finding point correspondences between multiple images (Not necessarily from the same camera) there are two possible options that we will consider in this course.

<ol>

 <li>Extracting SIFT descriptors and matching these descriptors at keypoint locations followed by a <em>chaining </em> This will be discussed in Lab Assignment 6.</li>

 <li>Using Optical Flow to estimate the apparent motion vectors: <em>v<sub>x</sub>,v<sub>y </sub></em>of image pixels or regions from one frame to the next. The theoretical details of Optical Flow computations will be treated in a coming lecture and are not essential for successfully completing this lab assignment.</li>

</ol>

For this lab session, we provide a script that implements the Lucas-Kanade Optical Flow-based tracker: <em>&lt;</em>LKtracker.m<em>&gt;</em>.

<strong>Note: </strong>You are not expected to do any work with this script for optical flow based tracking. If you are interested, you can have a look at the implementation, as extra information for the Optical Flow lecture.

For this lab we will provide:

<ul>

 <li>The point correspondences obtained from the Lucas-Kanade Optical Flow-based tracker as mat files ( ‘Xpoints’ &amp; ‘Ypoints’). You can use these to perform the Structure-from-motion assignment.</li>

 <li>We also provide <strong>measurement matrix.txt </strong>point correspondences obtained using SIFT matches and chaining as we will do in Lab 6.</li>

</ul>

<h2>2        Structure from Motion</h2>

Using the feature points from the provided measurement matrix or the provided Xpoints and Ypoints mat file containing Optical Flow tracked points, as input, perform the affine structure from motion procedure.

The following steps will guide you through factorization for structure from motion :

<ol>

 <li>After you load the point correspondences, normalize them by subtracting the centroid (mean).</li>

</ol>

<ul>

 <li><strong>Hint </strong>: Mean should be taken across different views of each point. So, you need the mean of each column.</li>

</ul>

<ol start="2">

 <li>The matrix of measurements, <em>D </em>needs to be decomposed into two matrices: <em>M </em>and <em>S</em>.</li>

</ol>

Apply SVD to the 2<em>m </em>× <em>n </em>loaded points matrix to express it as <em>D </em>= <em>U </em>∗ <em>W </em>∗ <em>V </em><sup>0</sup>.

<table width="488">

 <tbody>

  <tr>

   <td width="59">1</td>

   <td width="429">[… ,… ,… ] = svd(X);</td>

  </tr>

 </tbody>

</table>

The matrix <em>D </em>must have rank 3. Keep only the top 3 columns/rows such that:

<table width="488">

 <tbody>

  <tr>

   <td width="59">1</td>

   <td width="429">% The matrix of measurements must have rank 3.</td>

  </tr>

  <tr>

   <td width="59">2</td>

   <td width="429">% Keep only the corresponding top 3 rows/columns …from the SVD decompositions.</td>

  </tr>

  <tr>

   <td width="59">3</td>

   <td width="429">U = …</td>

  </tr>

  <tr>

   <td width="59">4</td>

   <td width="429">W = …</td>

  </tr>

  <tr>

   <td width="59">5</td>

   <td width="429">V = …</td>

  </tr>

 </tbody>

</table>

<ul>

 <li>U is a 2<em>m </em>× 3 matrix,</li>

 <li>W is a 3 × 3 matrix,</li>

 <li>V is a <em>n </em>× 3 matrix.</li>

</ul>

<ol start="3">

 <li>From <em>U,W,V </em>compute the matrices M and S. A possible decomposition is and</li>

</ol>

<table width="488">

 <tbody>

  <tr>

   <td width="41">1</td>

   <td width="446">% Compute M and S: One possible decomposition is …M = U Wˆ{1/2} and S = Wˆ{1/2} V’</td>

  </tr>

  <tr>

   <td width="41">2</td>

   <td width="446">M = …</td>

  </tr>

  <tr>

   <td width="41">3</td>

   <td width="446">S = …</td>

  </tr>

  <tr>

   <td width="41">4</td>

   <td width="446">save(‘M’,’M’)</td>

  </tr>

 </tbody>

</table>

<ol start="4">

 <li>This decomposition is not unique, to resolve the affine ambiguity, we use Least Squares and the Cholesky decomposition. For this we:

  <ul>

   <li>We define a matrix <em>L </em>such that <em>A<sub>i</sub>LA<sub>i </sub></em>= <em>Id</em>, where <em>A<sub>i </sub></em>are the <em>x </em>and <em>y </em>coordinates of the motion matrix <em>M</em>, and <em>Id </em>is the identity matrix.</li>

  </ul></li>

</ol>

<table width="453">

 <tbody>

  <tr>

   <td width="59">1</td>

   <td width="394">% We define the starting value for L, L0 as: …A1 L0 A1′ = Id</td>

  </tr>

  <tr>

   <td width="59">2</td>

   <td width="394">A1 = M(1:2, :);</td>

  </tr>

  <tr>

   <td width="59">3</td>

   <td width="394">L0 = pinv(A1′ * A1);</td>

  </tr>

  <tr>

   <td width="59">45</td>

   <td width="394">% We solve L by iterating through all images …and finding L one which minimizes …Ai*L*Ai’ = Id, for all i.</td>

  </tr>

  <tr>

   <td width="59">6</td>

   <td width="394">% LSQNONLIN solves non-linear least squares …problems. Please check the Matlab … documentation.</td>

  </tr>

  <tr>

   <td width="59">7</td>

   <td width="394">L = lsqnonlin(@residuals, L0);</td>

  </tr>

 </tbody>

</table>

We use the Matlab <em>lsqnonlin </em>function which minimizes a given function (In our case the function handler: <em>@residuals</em>) starting from an initial solution guess, in our case <em>L</em><sub>0</sub>.

We then need to define the function <em>&lt;</em>residuals.m<em>&gt; </em>to be optimized by the <em>lsqnonlin</em>: <em>A<sub>i</sub>LA<sub>i </sub></em>= <em>Id</em>.

<table width="453">

 <tbody>

  <tr>

   <td width="59">1</td>

   <td width="394">% Compute the residuals</td>

  </tr>

  <tr>

   <td width="59">23</td>

   <td width="394">for i = 1:size(M, 1)/2 % Loop over the … cameras: rows are ordered as x1,y1, … x2,y2, ..</td>

  </tr>

  <tr>

   <td width="59">4</td>

   <td width="394">% Define the x and y projections for the …current camera:</td>

  </tr>

  <tr>

   <td width="59">5</td>

   <td width="394">Ai = M(i*2-1 : i*2, : );</td>

  </tr>

  <tr>

   <td width="59">67</td>

   <td width="394">% Definite the function to be minimized: …Ai L Ai’ – Id = 0</td>

  </tr>

  <tr>

   <td width="59">8</td>

   <td width="394">diff i = …</td>

  </tr>

  <tr>

   <td width="59">910</td>

   <td width="394">diff(i,:) = diff i(:);</td>

  </tr>

  <tr>

   <td width="59">11</td>

   <td width="394">end</td>

  </tr>

 </tbody>

</table>

<ul>

 <li>We decompose <em>L </em>using the Cholesky decomposition into <em>L </em>= <em>CC<sup>T</sup></em>.</li>

 <li>We redefine <em>M </em>and <em>S </em>as: <em>M </em>= <em>MC </em>and <em>S </em>= <em>C</em><sup>−1</sup><em>S</em>.</li>

</ul>

<table width="453">

 <tbody>

  <tr>

   <td width="59">1</td>

   <td width="394">% Recover C from L by Cholesky decomposition.</td>

  </tr>

  <tr>

   <td width="59">2</td>

   <td width="394">C = chol(L,’lower’);</td>

  </tr>

  <tr>

   <td width="59">34</td>

   <td width="394">% Update M and S with the corresponding C …form: M = MC and S = Cˆ{-1}S.</td>

  </tr>

  <tr>

   <td width="59">5</td>

   <td width="394">M = …</td>

  </tr>

  <tr>

   <td width="59">6</td>

   <td width="394">S = …</td>

  </tr>

 </tbody>

</table>

<ol start="5">

 <li>With the now estimated ‘S’ matrix, you can use <em>plot3 </em>to visualize the points. The rows of S are the 3D coordinates (<em>x</em>, <em>y </em>and <em>z</em>) vectors needed as the argument to <em>plot3</em>.</li>

 <li>Perform these steps using the given points in the <strong>measurement matrix.txt </strong>and the points obtained from the Optical Flow tracker (Xpoints and Ypoints matrices). Compare the obtained results.</li>

</ol>