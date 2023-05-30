# Copy-on-write protection

The "Copy-on-wirte" (COW) portection technique has a lot to do with optimization.<br>
Each process shares the same DLL's physical memory and shares virtual memory.<br>
This method is called mapping. It is an effective operating system optimization method under the assumption that only addresses are shared and that only reading of the same DLL exists.<br>
COW works in the following situations.<br>

![image](https://github.com/Kwhitebear/Security_study/assets/99308681/8bb3747b-5426-4c9a-a4e9-e989617988dc)

If try to write to a DLL that exists in physical memory, the OS copies the physical memory DLL as it is, proceeds with the write to the copied DLL,<br>
and then updates the DLL of the user-mode process that requested the write to the newly copied DLL.<br>

![image](https://github.com/Kwhitebear/Security_study/assets/99308681/965af23f-bddc-4cb1-bdaf-d7a83e274199)

![image](https://github.com/Kwhitebear/Security_study/assets/99308681/643a3bb9-ba1d-4fc9-9b5d-d62b8a09772b)

<h2>More...</h2>

Memory belonging to a process protects its virtual address space.<br>
Windows also uses virtual memory hardware to protect memory.<br>
The implementation of this protection is processor specific.<br>
For Example, you can mark code pages in a process's address space as read-only and protect them from modification by user-mode threads.<br>

![image](https://github.com/Kwhitebear/Security_study/assets/99308681/6d347998-403b-49aa-a88a-58ff2f425892)

Assume that two processes load pages from the same DLL into virtual memory space.<br>
These virtual memory pages are mapped to the same physical memory pages for both processes.<br>
If both processes simply write to these pages, the same You can map it to a real page and share it.<br>

![image](https://github.com/Kwhitebear/Security_study/assets/99308681/b947f75a-e242-4c4c-9936-4d9c445873c8)

When process 1 writes to one of these pages, the contents of the physical page are copied to the other physical page, and the virtual memory map for process 1 is updated.<br>
Both processes now have their own instances of the apge in physical memory.<br>
other processes cannot see the changes.<br>

In summary, processes run independently using virtual memory address space, but physical memory can have physical pages shared by multiple processes. These shared pages can be used to provide data sharing and efficiency between multiple processes.<br>

When another process writes to a shared page, its access is restricted by memory protection, causing it to copy and refer to the new page.<br>
