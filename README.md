Download Link: https://assignmentchef.com/product/solved-eecs678-project-3-buddy-memory-allocator
<br>



Kernel needs to allocate memory for user-level applications and the kernel itself. For user-level applications, it allocates a fixed size page frame at the time of each page fault. For kernel’s own allocations—such as allocating DMA buffers—it may need to allocate multiple physically contiguous page frames. To serve these requirements, Linux kernel implements an allocator based on the buddy algorithm.

In this project, you will implement and evaluate a memory allocator based on the buddy algorithm.

<h1>Background</h1>

The buddy algorithm manages memory blocks of exponential sizes (e.g., 4KB, 8KB, 16KB, …). For example, if 21KB is requested, then the buddy allocator will return 32KB of memory block (waste of 11KB).  The allocator keeps a set of free-lists for each block size.




Below table illustrates the workings of Buddy Allocator. Consider we have 1024K(1MB) of memory and the smallest possible block in the system is 64K. We have four memory allocation requests A, B, C  and, D




<table width="600">

 <tbody>

  <tr>

   <td width="62">1. Init</td>

   <td colspan="3" width="134"> </td>

   <td width="53"> </td>

   <td colspan="3" width="202">1024K</td>

   <td width="149"> </td>

  </tr>

  <tr>

   <td width="62">2. A=80K</td>

   <td width="62">A</td>

   <td colspan="2" width="72">128K</td>

   <td width="53"> </td>

   <td colspan="2" width="81">256K</td>

   <td width="121"> </td>

   <td width="149">512K</td>

  </tr>

  <tr>

   <td width="62">3. B=60K</td>

   <td width="62">A</td>

   <td width="36">B</td>

   <td width="36">64K</td>

   <td width="53"> </td>

   <td colspan="2" width="81">256K</td>

   <td width="121"> </td>

   <td width="149">512K</td>

  </tr>

  <tr>

   <td width="62">4. C=80K</td>

   <td width="62">A</td>

   <td width="36">B</td>

   <td width="36">64K</td>

   <td width="53">C</td>

   <td width="14"> </td>

   <td width="67">128K</td>

   <td width="121"> </td>

   <td width="149">512K</td>

  </tr>

  <tr>

   <td width="62">5. A ends</td>

   <td width="62">128K</td>

   <td width="36">B</td>

   <td width="36">64K</td>

   <td width="53">C</td>

   <td width="14"> </td>

   <td width="67">128K</td>

   <td width="121"> </td>

   <td width="149">512K</td>

  </tr>

  <tr>

   <td width="62">6. D=32K</td>

   <td width="62">128K</td>

   <td width="36">B</td>

   <td width="36">D</td>

   <td width="53">C</td>

   <td width="14"> </td>

   <td width="67">128K</td>

   <td width="121"> </td>

   <td width="149">512K</td>

  </tr>

  <tr>

   <td width="62">7. B ends</td>

   <td width="62">128K</td>

   <td width="36">64K</td>

   <td width="36">D</td>

   <td width="53">C</td>

   <td width="14"> </td>

   <td width="67">128K</td>

   <td width="121"> </td>

   <td width="149">512K</td>

  </tr>

  <tr>

   <td width="62">8. D ends</td>

   <td colspan="3" width="134">256K</td>

   <td width="53">C</td>

   <td width="14"> </td>

   <td width="67">128K</td>

   <td width="121"> </td>

   <td width="149">512K</td>

  </tr>

  <tr>

   <td width="62">9. C ends</td>

   <td colspan="3" width="134"> </td>

   <td width="53"> </td>

   <td colspan="3" width="202">1024K</td>

   <td width="149"> </td>

  </tr>

 </tbody>

</table>




The allocation could happen in the following order.




<ol>

 <li>Init – Initial state of the system, where we have a single block of maximum size.</li>

 <li>A allocates 80K of memory.

  <ol>

   <li>The smallest possible block that can accommodate the request is 128K.</li>

   <li>No block of size 128K is available. If the free list of this block is empty, the buddy allocator traverse through the free lists of larger blocks until it finds a free block.</li>

   <li>The allocator then splits the free block. Whenever a free block is split into two, one block gets either used or further split, and its buddy block gets added to the corresponding free list.</li>

   <li>The 1024K block is then split into two sub-blocks, B<sub>L</sub>​ and B<sub>​ </sub>R​ , each of size<sub>​ </sub> The block B<sub>L </sub>​ is further split and the block B<sub>​ </sub>R ​ is added to the free list of<sub>​        </sub> 512K blocks.</li>

   <li>The Block B<sub>L </sub>​ <sub>​</sub>is again split into two sub-blocks,B<sub>LL </sub>​ and B<sub>​     </sub>​LR, each of size 256K.<sub>​   </sub> The block B​LL is further split and the block B<sub>​                   </sub>LR ​           is added to the free list of 256K<sub>​                 </sub> This is repeated until there is a free 128K block</li>

   <li>Note that the left block from a split is always allocated or further split and the right block is always added to the free list.</li>

   <li>In the end of step 1, we have 128K block allocated to A and also we have free blocks of sizes 128K, 256K and 512K.</li>

  </ol></li>

 <li>B allocates 60K of memory.

  <ol>

   <li>The smallest possible block that can accommodate the request is 64K.</li>

   <li>128K block is selected and splits. The left block is assigned to B and right block — its buddy– is added to the free list of 64K blocks.</li>

  </ol></li>

 <li>C allocates 80K of memory.

  <ol>

   <li>The smallest possible block that can accommodate the request is 128K.</li>

   <li>There is no free block of size 128K.</li>

   <li>The 256K block is selected and splits. The left block is assigned to C and the its buddy is added to the free list of 128K blocks.</li>

  </ol></li>

 <li>A ends

  <ol>

   <li>The 128K block is freed and added to the free list. Whenever a block is freed, it can be coalesced with its buddy to form a single memory block. However, here its  buddy is not free, so cannot coalesce.</li>

  </ol></li>

 <li>D allocates 32K of memory.

  <ol>

   <li>The smallest possible block that can accommodate the request is 64K(smallest possible block in the system).</li>

   <li>There is a free 64k block available and is allocated to D.</li>

  </ol></li>

 <li>B ends

  <ol>

   <li>The 64K block is freed and added to the free list. This cannot be coalesced as its buddy is occupied by D.</li>

  </ol></li>

 <li>D ends

  <ol>

   <li>The 64K block is freed. Its buddy is also free. Coalesce the blocks to form a single 128K block.</li>

   <li>The buddy of 128K block is also free, coalesce the blocks again to form a</li>

  </ol></li>

</ol>

single 256K block. The buddy of 256K block is not free so further coalesce is not possible. The 256K block is added to the free list.

<ol start="9">

 <li>C ends

  <ol>

   <li>The 128K block is freed. All blocks are free now.  Coalesce all the buddy pairs to finally get a single free block of 1024K.</li>

  </ol></li>

</ol>

<h1>Specification</h1>

[Allocation]




void* buddy_alloc (int size);




On a memory request, the allocator returns the head of a free-list of the matching size (i.e., smallest block that satisfies the request). If the free-list of the matching block size is empty, then a larger block size will be selected. The selected (large) block is then splitted into two smaller blocks. Among the two blocks, left block will be used for allocation or be further splitted while the right block will be added to the appropriate free-list.







[Free]




void buddy_free(void *addr);




Whenever a block is freed, the allocator checks its buddy. If the buddy is free as well, then the two buddies are combined to form a bigger block. This process continues until one of the buddies is not free.




<em>How to find a buddy and check if the buddy is free? </em>

Suppose we have block <em>B1</em>​ of order <em>O, </em>​   we can compute the buddy using the formula below.




B2 = B1 XOR (1 &lt;&lt; O)

We provide a convenient macro BUDDY_ADDR() for you.


