Download Link: https://assignmentchef.com/product/solved-lab-8-2d-exclusive-scan-cmput398
<br>
<h1>1.  Objective</h1>

Implement an <strong>exclusive</strong> scan program on a 2D arrays.

The scan operator will be the addition (plus) operator, so in other words you are performing a Summed-Area Table.

You should try to implement the work efficient kernel (Blelloch) shown in the lectures, but it is fine if you implement the naive scan (Hillis and Steele). You will get 80% for correct outputs and 20% for having good optimization.

One thing to be aware of is that most implementations only handle arrays the same size as a block; however, your kernel should be able to handle input arrays of arbitrary dimension.  More instruction about this below.

This lab will be submitted as one zipped file through <a href="https://eclass.srv.ualberta.ca/portal/">eclass</a><a href="https://eclass.srv.ualberta.ca/portal/">.</a> Details for submission are at the end of the lab.

<h1>2.  Instructions</h1>

First, you need to implement a 1D scan kernel and from this 1D case you will be able to extend to 2D case.

<strong><u>2.1. Handling 1D list with arbitrary length:</u></strong>

First, make sure you carefully read the instructions after reading through the following NVIDIA article:

<a href="http://http.developer.nvidia.com/GPUGems3/gpugems3_ch39.html">http://http.developer.nvidia.com/GPUGems3/gpugems3_ch39.html</a> Especially examples 39-1 and 39-2.

These examples show you instructions on how to implement a good 1D prefix sum kernel that can handle list with length exactly equal to the block size or block size * 2 depend on the implementation but it’s a good place to start. You can do a little bit of extension to the kernel mentioned in the article to make it work on arrays with length less than block size as well as greater than block size.

<strong> </strong>

<strong><u>To make the kernel work on arrays with length less than block size</u></strong>, you just need to add two things to your kernel:

<ul>

 <li>When loading the input into shared memory, check if the index is out of bound (more than the length of the array – 1). If it is out of bound, simply load a 0 into shared memory. This is like padding the 1D list with 0 at the end.</li>

 <li>When you write the output, also check if the index is out of bound. If it is, do not write anything</li>

</ul>

After this step, you have a kernel that can handle lists with length less than or equal to block size (or block size * 2)

<strong><u>To make the kernel work on arrays with length greater than block size:</u> </strong>Let’s assume we are running an <strong>inclusive</strong> scan with blocks of size 4 and we have the following input array of length 12. We are doing <strong>exclusive</strong> scan but the logic is the very same except we are shifting our output by 1 to the right.




<table width="622">

 <tbody>

  <tr>

   <td width="52"><strong>X</strong><strong>0 </strong></td>

   <td width="52"><strong>X</strong><strong>1 </strong></td>

   <td width="52"><strong>X</strong><strong>2 </strong></td>

   <td width="52"><strong>X</strong><strong>3 </strong></td>

   <td width="52"><strong>X</strong><strong>4 </strong></td>

   <td width="52"><strong>X</strong><strong>5 </strong></td>

   <td width="52"><strong>X</strong><strong>6 </strong></td>

   <td width="52"><strong>X</strong><strong>7 </strong></td>

   <td width="52"><strong>X</strong><strong>8 </strong></td>

   <td width="52"><strong>X</strong><strong>9 </strong></td>

   <td width="52"><strong>X</strong><strong>10 </strong></td>

   <td width="52"><strong>X</strong><strong>11 </strong></td>

  </tr>

 </tbody>

</table>




For an inclusive scan, we want the following:




<table width="580">

 <tbody>

  <tr>

   <td width="44"><strong>X</strong><strong>0 </strong></td>

   <td width="80"><strong>∑(X<sub>0</sub>..X<sub>1</sub>) </strong></td>

   <td width="80"><strong>∑(X<sub>0</sub>..X<sub>2</sub>) </strong></td>

   <td width="80"><strong>∑(X<sub>0</sub>..X<sub>3</sub>) </strong></td>

   <td width="44"><strong>… </strong></td>

   <td width="80"><strong>∑(X<sub>0</sub>..X<sub>9</sub>) </strong></td>

   <td width="86"><strong>∑(X<sub>0</sub>..X<sub>10</sub>) </strong></td>

   <td width="86"><strong>∑(X<sub>0</sub>..X<sub>11</sub>) </strong></td>

  </tr>

 </tbody>

</table>




Let say we <strong>fix our block size to a constant</strong> and divide the bigger list into smaller lists of equal size that are handled by each block. Each block will do a scan on each smaller list (of length block size * 2 or block size in this example).

After we do scan on smaller list, we will have: Block 0:

<table width="287">

 <tbody>

  <tr>

   <td width="47"><strong>X</strong><strong>0 </strong></td>

   <td width="80"><strong>∑(X<sub>0</sub>..X<sub>1</sub>) </strong></td>

   <td width="80"><strong>∑(X<sub>0</sub>..X<sub>2</sub>) </strong></td>

   <td width="80"><strong>∑(X<sub>0</sub>..X<sub>3</sub>) </strong></td>

  </tr>

  <tr>

   <td width="47"> </td>

   <td width="80"> </td>

   <td width="80"> </td>

   <td width="80"> </td>

  </tr>

  <tr>

   <td width="47"><strong>X</strong><strong>4 </strong></td>

   <td width="80"><strong>∑(X<sub>4</sub>..X<sub>5</sub>) </strong></td>

   <td width="80"><strong>∑(X<sub>4</sub>..X<sub>6</sub>) </strong></td>

   <td width="80"><strong>∑(X<sub>4</sub>..X<sub>7</sub>) </strong></td>

  </tr>

 </tbody>

</table>

Block 1:

Block 2:

<table width="298">

 <tbody>

  <tr>

   <td width="45"><strong>X</strong><strong>8 </strong></td>

   <td width="80"><strong>∑(X<sub>8</sub>..X<sub>9</sub>) </strong></td>

   <td width="86"><strong>∑(X<sub>8</sub>..X<sub>10</sub>) </strong></td>

   <td width="86"><strong>∑(X<sub>8</sub>..X<sub>11</sub>) </strong></td>

  </tr>

 </tbody>

</table>




Now, if we add <strong>0</strong> to all the output of block 0, <strong>∑(X<sub>0</sub>..X<sub>3</sub>) </strong>to all the output of block 1 and

<strong>∑(X<sub>0</sub>..X<sub>3</sub>) + ∑(X<sub>4</sub>..X<sub>7</sub>) </strong>to the output of block two we will have Block 1:

<table width="287">

 <tbody>

  <tr>

   <td width="47"><strong>X</strong><strong>4 </strong></td>

   <td width="80"><strong>∑(X<sub>0</sub>..X<sub>5</sub>) </strong></td>

   <td width="80"><strong>∑(X<sub>0</sub>..X<sub>6</sub>) </strong></td>

   <td width="80"><strong>∑(X<sub>0</sub>..X<sub>7</sub>) </strong></td>

  </tr>

 </tbody>

</table>

Block 2:

<table width="298">

 <tbody>

  <tr>

   <td width="45"><strong>X</strong><strong>8 </strong></td>

   <td width="80"><strong>∑(X<sub>0</sub>..X<sub>9</sub>) </strong></td>

   <td width="86"><strong>∑(X<sub>0</sub>..X<sub>10</sub>) </strong></td>

   <td width="86"><strong>∑(X<sub>0</sub>..X<sub>11</sub>) </strong></td>

  </tr>

 </tbody>

</table>

This is exactly what we wanted.

So the trick is to have an additional array that can somehow contains the sum of elements from the beginning of the list to the elements that are at index of multiples of the block size. How do we get this? From the above example, let say we have <strong>∑(X<sub>0</sub>..X<sub>3</sub>) </strong>and<strong> ∑(X<sub>4</sub>..X<sub>7</sub>)</strong> and we want <strong>0, ∑(X<sub>0</sub>..X<sub>3</sub>)</strong> and<strong> ∑(X<sub>0</sub>..X<sub>3</sub>) + ∑(X<sub>4</sub>..X<sub>7</sub>)</strong>. This is exactly equal to doing a scan on a list with two elements <strong>∑(X<sub>0</sub>..X<sub>3</sub>) </strong>and <strong>∑(X<sub>4</sub>..X<sub>7</sub>).  </strong>

<strong> </strong>

This bring us to the idea in section 39.2.4 Arrays of Arbitrary Size of the link <a href="http://http.developer.nvidia.com/GPUGems3/gpugems3_ch39.html">http://http.developer.nvidia.com/GPUGems3/gpugems3_ch39.html</a> :

<em>“Before zeroing the last element of block i (the block of code labeled B in Listing 39-2), we store the value (the total sum of block i) to an auxiliary array SUMS. We then scan SUMS in the same manner, writing the result to an array INCR. We then add INCR[i] to all elements of block i using a simple uniform add kernel invoked on N/B thread blocks of B/2 threads each.” </em>(Note that the above sentence is just a quote and does not apply to this example)




Come back to the example, we allocate an array which is the same length as the number of blocks (in this case 3). Then each block stores the max value in the aux array. If the entire array fits into a single block, then we don’t need the aux array and can pass NULL as the argument. To use the aux array, we add the following line at the end of the kernel code.




<ol>

 <li><strong>if</strong> (aux &amp;&amp; threadIdx.x == 0)</li>

 <li>aux[blockIdx.x] = temp[BLOCK_SIZE – 1]; // where temp is the shared memory array</li>

</ol>

Note that BLOCK_SIZE -1 is supposed to point to the last element in the block or the max sum, but this might be different for your implementation. For example in the second block aux[blockIdx.x] will be <strong>∑(X<sub>4</sub>..X<sub>7</sub>)</strong>. With the example above the aux array should be:




<table width="246">

 <tbody>

  <tr>

   <td width="80"><strong>∑(X<sub>0</sub>..X<sub>3</sub>) </strong></td>

   <td width="80"><strong>∑(X<sub>4</sub>..X<sub>7</sub>) </strong></td>

   <td width="86"><strong>∑(X<sub>8</sub>..X<sub>11</sub>) </strong></td>

  </tr>

 </tbody>

</table>

Now we can perform scan on the aux array. In this case the array fits into one block and we do not need to pass anything to the aux argument. By performing scan on the aux array, we get:




<table width="246">

 <tbody>

  <tr>

   <td width="80"><strong>∑(X<sub>0</sub>..X<sub>3</sub>) </strong></td>

   <td width="80"><strong>∑(X<sub>0</sub>..X<sub>7</sub>) </strong></td>

   <td width="86"><strong>∑(X<sub>0</sub>..X<sub>11</sub>) </strong></td>

  </tr>

 </tbody>

</table>




Now we can perform uniform sum on the output array from the first scan:




Block 0:

<table width="287">

 <tbody>

  <tr>

   <td width="47"><strong>X</strong><strong>0 </strong></td>

   <td width="80"><strong>∑(X<sub>0</sub>..X<sub>1</sub>) </strong></td>

   <td width="80"><strong>∑(X<sub>0</sub>..X<sub>2</sub>) </strong></td>

   <td width="80"><strong>∑(X<sub>0</sub>..X<sub>3</sub>) </strong></td>

  </tr>

 </tbody>

</table>

Block 1 + aux[0]:

<table width="320">

 <tbody>

  <tr>

   <td width="80"><strong>∑(X<sub>0</sub>..X<sub>4</sub>) </strong></td>

   <td width="80"><strong>∑(X<sub>0</sub>..X<sub>5</sub>) </strong></td>

   <td width="80"><strong>∑(X<sub>0</sub>..X<sub>6</sub>) </strong></td>

   <td width="80"><strong>∑(X<sub>0</sub>..X<sub>7</sub>) </strong></td>

  </tr>

 </tbody>

</table>

Block 2 + aux[1]:

<table width="332">

 <tbody>

  <tr>

   <td width="80"><strong>∑(X<sub>0</sub>..X<sub>8</sub>) </strong></td>

   <td width="80"><strong>∑(X<sub>0</sub>..X<sub>9</sub>) </strong></td>

   <td width="86"><strong>∑(X<sub>0</sub>..X<sub>10</sub>) </strong></td>

   <td width="86"><strong>∑(X<sub>0</sub>..X<sub>11</sub>) </strong></td>

  </tr>

 </tbody>

</table>




We need to change the signature of our scan function from (this is in the cuda scan page above):

__global__ <strong>void</strong> scan(<strong>float</strong> *g_odata, <strong>float</strong> *g_idata, <strong>int</strong> n)




The arguments are as follows:




<ul>

 <li>g_odata – output array which is the same size as the input array</li>

 <li>g_idata – input array</li>

 <li>n – block size</li>

</ul>




To:

__global__ <strong>void</strong> scan(<strong>float</strong> *g_odata, <strong>float</strong> *g_idata,<strong> float</strong> *g_aux, <strong>int</strong> len)

Where:

<ul>

 <li>g_odata – output array which is the same size as the input array</li>

 <li>g_idata – input array</li>

 <li>g_aux – is an array the size of the number of blocks we will see why later</li>

 <li>len – is the size of the array</li>

</ul>




We removed n and additionally predefine the block size:




#define BLOCK_SIZE 512




The one issue we must still solve is when the length of aux array is larger than block size (i.e. when the input length is greater than block size * block size or (block size * 2) * (block size * 2)). In that case, we need another aux array to add the sums from the previous blocks. A simple solution is to write a recursive function on the <strong>CPU</strong> as a wrapper of the actual scan kernel.




<strong>void</strong> recursive_scan(<strong>float</strong> *g_odata, <strong>float</strong> *g_idata, <strong>int</strong> len)




The pseudo code will look something like this:




<ol>

 <li>FUNCTION recursive_scan(output, input, len)</li>

 <li>Calculate the number of blocks needed</li>

 <li>IF only one block is needed perform scan by calling the kernel and exit function</li>

 <li>ELSE</li>

 <li>Allocate memory for aux array (contains the sum of elements of each block)</li>

 <li>Allocate memory for scanned aux array (contain the prefix sum of aux)</li>

 <li>Perform scan passing in the arguments from the function and the aux array</li>

 <li>Call recursive_scan (scanned_aux, aux, num_of_blocks)</li>

 <li>Perform uniform addition on output with the scanned aux array</li>

 <li>Free aux array and scanned aux array</li>

 <li>ENDIF</li>

</ol>

<strong>For the pseudo code above, red lines are CUDA kernel calls. Line 7 being the actual scan kernel for 1D list. </strong>

<strong><u>The exclusive scan is like the inclusive scan; however, everything is shifted to the</u> <u>right by one.</u></strong> For example, using the example array above, we will get the following:

Block 0:

<table width="287">

 <tbody>

  <tr>

   <td width="47"><strong>0 </strong></td>

   <td width="80"><strong>X</strong><strong>0 </strong></td>

   <td width="80"><strong>∑(X<sub>0</sub>..X<sub>1</sub>) </strong></td>

   <td width="80"><strong>∑(X<sub>0</sub>..X<sub>2</sub>) </strong></td>

  </tr>

  <tr>

   <td width="47"> </td>

   <td width="80"> </td>

   <td width="80"> </td>

   <td width="80"> </td>

  </tr>

  <tr>

   <td width="47"><strong>0 </strong></td>

   <td width="80"><strong>X</strong><strong>4 </strong></td>

   <td width="80"><strong>∑(X<sub>4</sub>..X<sub>5</sub>) </strong></td>

   <td width="80"><strong>∑(X<sub>4</sub>..X<sub>6</sub>) </strong></td>

  </tr>

 </tbody>

</table>

Block 1:

Block 2:

<table width="291">

 <tbody>

  <tr>

   <td width="46"><strong>0 </strong></td>

   <td width="78"><strong>X</strong><strong>8 </strong></td>

   <td width="80"><strong>∑(X<sub>8</sub>..X<sub>9</sub>) </strong></td>

   <td width="86"><strong>∑(X<sub>8</sub>..X<sub>10</sub>) </strong></td>

  </tr>

 </tbody>

</table>




Now we need to add <strong>∑(X<sub>0</sub>..X<sub>2</sub>) + X<sub>3</sub></strong> to Block 1 and <strong>∑(X<sub>0</sub>..X<sub>2</sub>) + X<sub>3 </sub>+ ∑(X<sub>4</sub>..X<sub>6</sub>) + X<sub>7</sub> </strong>to Block 2. There is a simple trick for the exclusive algorithm if you implement the work efficient kernel (Blelloch). After the Up-Sweep or Reduce phase we have the value we need.




Block 0:

<table width="287">

 <tbody>

  <tr>

   <td width="47"><strong>X</strong><strong>0 </strong></td>

   <td width="80"><strong>∑(X<sub>0</sub>..X<sub>1</sub>) </strong></td>

   <td width="80"><strong>X</strong><strong>2 </strong></td>

   <td width="80"><strong>∑(X<sub>0</sub>..X<sub>3</sub>) </strong></td>

  </tr>

  <tr>

   <td width="47"> </td>

   <td width="80"> </td>

   <td width="80"> </td>

   <td width="80"> </td>

  </tr>

  <tr>

   <td width="47"><strong>X</strong><strong>4 </strong></td>

   <td width="80"><strong>∑(X<sub>4</sub>..X<sub>5</sub>) </strong></td>

   <td width="80"><strong>X</strong><strong>6 </strong></td>

   <td width="80"><strong>∑(X<sub>4</sub>..X<sub>7</sub>) </strong></td>

  </tr>

 </tbody>

</table>

Block 1:

Block 2:

<table width="291">

 <tbody>

  <tr>

   <td width="46"><strong>X</strong><strong>8 </strong></td>

   <td width="80"><strong>∑(X<sub>8</sub>..X<sub>9</sub>) </strong></td>

   <td width="78"><strong>X</strong><strong>10 </strong></td>

   <td width="86"><strong>∑(X<sub>8</sub>..X<sub>11</sub>) </strong></td>

  </tr>

 </tbody>

</table>




<table width="605">

 <tbody>

  <tr>

   <td width="27">1.</td>

   <td colspan="2" width="73">// Up-Sweep</td>

   <td colspan="4" width="505"> </td>

  </tr>

  <tr>

   <td colspan="7" width="605">2. …3.4.        <strong>if</strong> (threadIdx.x == 0) {5.        <strong>if</strong> (aux)</td>

  </tr>

  <tr>

   <td colspan="5" rowspan="2" width="339">6.       aux[blockIdx.x] = temp[BLOCK_SIZE – 1];7.       temp[BLOCK_SIZE – 1] = 0; // clear the last 8. }9.</td>

   <td width="172">// save the last element</td>

   <td rowspan="2" width="94"> </td>

  </tr>

  <tr>

   <td width="172"> element</td>

  </tr>

  <tr>

   <td colspan="2" rowspan="2" width="29">10. 11.12.</td>

   <td colspan="2" width="83">// Down-Sweep</td>

   <td colspan="3" rowspan="2" width="492"> </td>

  </tr>

  <tr>

   <td colspan="2" width="83"> …</td>

  </tr>

  <tr>

   <td width="27"></td>

   <td width="2"></td>

   <td width="70"></td>

   <td width="13"></td>

   <td width="226"></td>

   <td width="172"></td>

   <td width="94"></td>

  </tr>

 </tbody>

</table>

<strong><u>2.2. Extend 1D to 2D scan:</u></strong>

Extending 1D scan to 2D scan is relatively easy. First, you do a 1D scan on each row of the matrix. Then do another scan on the <strong>transpose</strong> of what you get from the previous step and then transpose back to get the final result. (See section 39.3.2 Summed-Area Tables of the CUDA scan page)




For this lab, you need to use a transpose kernel. A good place to start is in this link: <a href="https://devblogs.nvidia.com/parallelforall/efficient-matrix-transpose-cuda-cc/">https://devblogs.nvidia.com/parallelforall/efficient</a><a href="https://devblogs.nvidia.com/parallelforall/efficient-matrix-transpose-cuda-cc/">–</a><a href="https://devblogs.nvidia.com/parallelforall/efficient-matrix-transpose-cuda-cc/">matrix</a><a href="https://devblogs.nvidia.com/parallelforall/efficient-matrix-transpose-cuda-cc/">–</a><a href="https://devblogs.nvidia.com/parallelforall/efficient-matrix-transpose-cuda-cc/">transpose</a><a href="https://devblogs.nvidia.com/parallelforall/efficient-matrix-transpose-cuda-cc/">–</a><a href="https://devblogs.nvidia.com/parallelforall/efficient-matrix-transpose-cuda-cc/">cuda</a><a href="https://devblogs.nvidia.com/parallelforall/efficient-matrix-transpose-cuda-cc/">–</a><a href="https://devblogs.nvidia.com/parallelforall/efficient-matrix-transpose-cuda-cc/">cc/</a>  But any other code may also be fine.




The pseudo code for all the steps we mentioned:




<ol>

 <li><strong>for each row j in deviceInput array: </strong></li>

 <li><strong>recursive_scan(deviceTmpOutput[j,:] , deviceInput[j,:], numInputColumns) 3. deviceOutput = transpose(deviceTmpOutput) </strong></li>

 <li><strong>for each row j in deviceOutput array: </strong></li>

 <li><strong>recursive_scan(deviceTmpOutput[j,:] , deviceOutput[j,:], numInputColumns) </strong></li>

 <li><strong>deviceOutput = transpose(deviceTmpOutput) </strong></li>

</ol>




Note that in the above pseudo code we are using another array called deviceTmpOutput to store the temporary output (there a variable with same name in the code) and the assignment deviceOutput = transpose(deviceTmpOutput) may not be what the code actually really looks like (the transpose should be a kernel with input, output array in its argument list and should not return anything). The notation X[j,:] denotes the row j of the 2D array X (actually notation in your code should be different).




<strong><u>Important:</u> </strong>after launching a kernel in the pseudo code make sure you call:




wbCheck(cudaDeviceSynchronize());

Or

cudaDeviceSynchronize();




<h1>3.  Local Setup Instructions</h1>

Steps:

<ol>

 <li>Download “Lab8.zip”.</li>

 <li>Unzip the file.</li>

 <li>Open the Visual Studios Solution in Visual Studios 2013.</li>

 <li>Build the project. Note the project has two configurations.

  <ol>

   <li>Debug</li>

   <li>Submission</li>

  </ol></li>

</ol>

But make sure you have the “Submission” configuration selected when you finally submit.

<ol start="5">

 <li>Run the program by pressing the following button:</li>

</ol>




Make sure the “Debug” configuration is selected.

<h1>4.  Testing</h1>

To run all tests located in “Dataset/ Test”, first build the project with the “Submission” configuration selected. Make sure you see the “Submission” folder and the folder contains the executables.

To run the tests, click on “Testing_Script.bat”. This will take a couple of seconds to run and the terminal should close when finished. The output is saved in “Marks.js”, but to view the calculated grade open “Grade.html” in a browser. If you make changes and rerun the tests, then make sure you reload “Grade.html”. You can double check with the timestamp at the top of the page.

You can test your performance using NSight. First build with the “Test” configuration selected. Then select NSIGHT -&gt; Start Performance Analysis…




In “Application Settings” make sure the “Application:” is set to the executable you wish to run and the “Arguments:” should be:

-e output.raw -i input.raw -o myOutput.raw -t vector

The “Working Directory:” should be the path to the test. For example:

PathToLab8DatasetTest11

Remember to replace PathToLab8 with the actual path to lab 8.

To make it easier for you to test the 1D scan kernel (the first and important step of this lab), you will be given another Visual Studio solution that contains the skeleton code that you can write your <strong>exclusive</strong> scan kernel in and test it. There will be two project in the solution, 1 for inclusive and 1 for exclusive scan, but you just need to write the exclusive scan. There will be test script provided for this solution too. Moving the scan kernel from this 1D solution to the 2D solution should be a matter of copy paste.

Also, there will a reference file provided for you to compare the speed of your kernel.


