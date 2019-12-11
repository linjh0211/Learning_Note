### 一、 相关接口

- Platform

```c++
cl_int clGetPlatformIDs(cl_uint num_entries,  
                        cl_platform_id *platforms, 
                        cl_uint *num_platforms)
/*
这个函数一般被调用两次：

    第一次调用这个函数是获得可用平台的数目
    然后为平台对象分配内存空间
    第二次调用用来获取平台对象

        num_entries是platforms中可以容纳的cl_platform_id表项的数目。如果platform不是NULL，那么num_entries必须大于零。
        platforms返回所找到的OpenCL平台清单。platforms中的每个cl_platform_id都用来标识某个特定的OpenCL平台。如果platforms是NULL，则被忽略。所返回的OpenCL平台的数目是num_entries和实际数目中较小的那一个。
        num_platforms返回实际可用的OpenCL平台数目。如果num_Platforms为NULL，则被忽略。

如果函数执行成功，clGetPlatformIDs会返回CL_SUCCESS。否则返回下面的错误

    CL_INVALID_VALUE        如果num_entries是0，而且platforms不为NULL；或则两者都是NULL
    CL_OUT_OF_HOST_MEMORY    在主机上为OpenCL平台分配内存空间失败
*/
```

- Device

```cpp
cl_int clGetDeviceIDs(cl_platform_id platform,  
	cl_device_type device_type,  
	cl_uint num_entries,
    cl_device_id *devices,  
	cl_uint *num_devices); 
/*
该函数与创建平台结构的函数参数相同，增加了cl_platform_id参数和cl_device_type。这两个参数很容易理解，cl_device_type为枚举类型，任何一本OpenCl相关书籍都可查到其取值，不做赘述。cl_platform_id便是上一函数获得的平台ID。
*/
```

- Context

```c++
cl_context clCreateContext( const cl_context_properties *properties,
							cl_uint num_devices,
							const cl_device_id *devices,
							void (CL_CALLBACK *pfn_notify)( const char *errorinfo,
															const void *private_info,
															size_t cb,
															void *user_data ),
							void *user_data,
							cl_int *errcode_ret )


cl_context clCreateContextFromType( const cl_context_properties *properties,
									cl_device_type device_type,
									void (CALLBACK *pfn_notify) (const char *errinfo,
																 const void *private_info,
																 size_t cb,
																 void *user_data),
									void *user_data,
									cl_int *errcode_ret )
/*
context 不再由别人提供，需要我们自己来创建。有两种方式来创建：已知一个 platform 和一组关联的 device，可以使用 clCreateContext 函数来创建；已知一个 platform 和 device 的类型，可以使用 clCreateContextFromType 函数来创建。
参数 pfn_notify 和 user_data 用来共同定义一个回调，用于报告 context 生命期中所出现错误的有关信息，记得把 user_data 最为最后一个参数传给回调函数。
*/
```

- program

```c++
cl_program clCreateProgramWithSource( cl_context	 context,
								  cl_uint		 count,
								  const char	 **strings,
								  const size_t	 *lengths,
								  cl_int		 *errcode_ret )

cl_program clCreateProgramWithBinary( cl_context		 context,
								  cl_uint			 num_devices,
								  const cl_device_id *device_list,
								  const size_t		 *lengths,
								  cosnt unsigned char **binaries,
								  cl_int			 *binary_status,
								  cl_int			 *errcode_ret )
    /*
    program 的意思相信大家都懂的，毕竟我们自己就是一名苦逼的 programmer。顾名思义，program 代表的是一个程序对象，里面包含着我们所写的 cl 程序。

做 OpenCL 编程，要写两种代码，一种是 host 程序，一种是 cl 程序，类比于图形编程，cl 程序就等同于 shader 程序。host 端的代码干的是为 cl 程序设置环境，打扫内存等保姆型工作， cl 程序才是真正 解决问题 的地方。

要让 cl 程序发挥作用，得先让 host 端把它送进 device 是吧。送之前得先加载进 host 吧。加载后存哪呢？存在 program 里面！！

创建一个 program 对象有两种方法，一种是利用 clCreateProgramWithSource 函数从 cl 源代码创建，另一种是利用 clCreateProgramWithBinary 函数从二进制中创建。两个函数的原型如上
    */
```

- 编译.cl程序文件 

```c++
cl_int clBuildProgram(cl_program			program,
				   cl_uint				 num,
				   const cl_device_id	 *device_list,
				   const char			 *options,
				   void(CL_CALLBACK *pfn_notify)(cl_program program,
												 void *user_data),
				   void					 *user_data )
/*
一般创建好一个 program 对象后，接着就是对其进行编译。从上面两个创建函数可以看出，现在 cl 程序存在 program 对象中，其形式莫过于源代码和二进制。而我们的 device （无论是CPU、GPU还是DSP）能处理的仅仅是机器码和指令。所以与一般的编程相同，我们需要将 cl 程序进行编译。不同之处在于编译方法不在是点一下IDE上的按钮或者make一下，而是在 host 程序内部以 API 调用的方式来控制 编译。控制编译的 API 是 clBuildProgram，其原型如上：

这个函数会把装有 cl 程序的 program 对象传入指定的 device（第三个参数）进行编译。而且，你也可以设定多种编译 options（第四个参数）。options 包括预处理器定义和各种优化以及代码生成（比如，-DUSE_FEATURE = 1 -cl -mad -enable）。

编译完成并最终生成的可执行代码还是存在了指定 device（第三个参数指定的）的 program 中。如果在所有指定 device 上编译成功，则该函数返回 CL_SUCCESS；否则会返回一个错误码。如果存在错误，可以调用 clGetProgramBuildInfo，并指定 param_name（第三个参数） 为 CL_PROGRAM_BUILD_LOG来查看某个 device（第二个参数）上的详细构建日志。
*/
    cl_int clGetProgramBuildInfo( cl_program			 program,
						  cl_device_id			 device,
						  cl_program_buld_info	 param_name,
						  size_t				 param_value_size,
						  void					 *param_value,
						  size_t				 *param_value_size_ret )

    /*
    这类由我们自己 create 的对象，可以手动控制其引用计数，对象使用完了一定要记得 release。不然内存泄漏是很惨的。
    */
cl_int clRetainProgram( cl_program program )	// 递减引用计数
cl_int clReleaseProgram(cl_program program )	// 递增引用计数
```

- kernel

```c++
/*
如果说上述 program 对象是一个容器，装着一个 cl 程序，那么一个 kernel 对象也是一个容器，装的是 cl 程序中的一个 kernel 函数。

kernel 函数指的是 cl 程序中那些以 kernel 或 __kernel 限定的函数。一段 cl 代码可以包含多个 kernel 函数，自然就对应着多个 kernel` 对象。

kernel 对象有什么用呢？想想我们要执行某个 kernel 函数，要传递一堆的参数，在平时的C编程里面，直接调用函数即可是吧。可问题在于现在 host 端与 cl 端是分离的，不光文件分离，甚至连执行的地方都是分离的（比如，host 在CPU上，cl 在GPU上）。这种情况下怎么向 kernel 函数 传递参数呢？

我们的 kernel 对象的用处就在于此：参数传递（数据传递）。我们一直说 OpenCL 提供了一个异构计算的平台，这些各种对象其实就是对异构编程的一个抽象。
*/
cl_kernel clCreateKernel( cl_program program,
					  const char *kernel_name,
					  cl_int	 *errcode_ret )
    /*
    创建 kernel 对象的方法是将 kernel 函数名（第二个参数） 传入 clCreateKernel：
    kernel_name 指的就是 cl 代码里面 kernel 或 __kernel 关键字后面的函数名。
    */
```

一次创建多个kernel函数对象

```c++
cl_int clCreateKernelsInProgram(cl_program	 program,
							cl_uint		 num_kernels,
							cl_kernel	 *kernels,
							cl_uint		 *num_kernels_ret)
/*
clCreateKernel 一次只能为一个 kernel 函数 创建一个 kernel 对象。如果你的 cl 代码里面有很多很多 kernel 函数 怎么办？一个个来？那还不烦死！所以这种情况请使用 clCreateKernelsInProgram 为 program 中的所有 kernel 函数 创建对象。
使用 clCreateKernelsInProgram 时需要调用两次：第一次确定 kernel 数量；然后申请空间；第二次具体创建 kernel 对象。比如下面的示例代码：
*/
cl_uint numKernels;
clCreateKernelsInProgram( program, 0, NULL, &numKernels );	// 第一次调用确定数量
cl_kernel *kernels = new cl_kernel[numKernels];				// 根据数量申请空间
clCreateKernelsInProgram( program, numKernels, kernels, &numKernels ); 	// 具体创建对象
```

为kernel对象传递参数

```c++
/*
创建好之后，就可以为相应的 kernel 函数 传递参数了。使用的 API 是 clSetKernelArg;
其中，arg_index 是从 0 开始的，即第一个参数索引为 0，第二个为 1，依次类推。
最后当然得记得增减计数器：
*/
cl_int clSetKernelArg( cl_kernel	 kernel,
				   cl_uint		 arg_index,
				   size_t		 *arg_size,
				   const void	 *arg_value )
cl_int RetainKernel( cl_kernel kernel )
cl_int ReleaseKernel(cl_kernel kernel )
```

**注意：有时我们会考虑对一个 `program` 对象重新编译。记住，重新编译前一定要把与该 `program` 对象相关的 `kernel` 对象全部释放掉，否则重新编译会出错！！！**

- binary的opencl程序
  - binary的好处

  ```c++
  /*
  提到创建 program 对象有两种方式： clCreateProgramWithSource 和 clCreateProgramWithBinary。区别仅在于 opencl 程序在用户面前的展现形式，前者是源代码形式，后者是二进制形式。二进制形式的数据格式是不透明的，不同的实现可以有不同的标准。使用二进制形式的好处有二：一是由于二进制码已经经过编译（部分编译为中间件或全部编译为可执行文件），所以加载速度更快，需要的内存更少；二是可以保护 opencl 代码，保护知识产权。
  */
  ```

  - 如何存储成binary

  ```c++
  /*
  二进制版的 opencl 程序从哪里来？前文说过，所有的 cl 代码都要经过加载并创建 program 对象，然后由 program 对象在 device 上面编译并执行。难道还有其他方式编译 opencl 代码？答案是：NO!
  
  意味着我们还是需要将代码送到 device 里面编译。你会说，这不是多此一举吗？看起来确实有点，不过一般的做法是在软件安装的时候就进行编译保存二进制形式，然后真正运行时才加载二进制。这样分成两个步骤的话，倒也说的过去。
  
  省去前面那些与 platform、device 和 context的代码，我们直接进入创建 program 的地方。首先还是利用 clCreateProgramWithSource 函数读取源代码文件并用 clBuildProgram 函数编译。示例代码如下：
  */
  cl_int status;
  cl_program program;
  
  ifstream kernelFile("binary_kernel.ocl", ios::in);
  if(!kernelFile.is_open())
  	return;
  
  ostringstream oss;
  oss << kernelFile.rdbuf();
  
  string srcStdStr = oss.str();
  const char *srcStr = srcStdStr.c_str();
  
  program = clCreateProgramWithSource(context, 
  									1, 
  									(const char **)&srcStr,
  									NULL,
  									NULL);
  
  if(program ==NULL)
  	return;
  
  status = clBuildProgram(program, 0, NULL, NULL, NULL, NULL);
  
  if(status != CL_SUCCESS)
  	return;
  
  ```

  - 如何保存将binary保存到磁盘上

  ```c++
  /*
  现在我们已经将 opencl 代码在 device 编译完成了。接下来要做的就是将编译好的二进制取出来存在磁盘上。使用的 API 就是 clGetProgramInfo:
  */
  cl_int clGetProgramInfo(cl_program program,
  						cl_program_info param_name,
  						size_t param_value_size,
  						void *param_value,
  						size_t *param_value_size_ret)
      /*使用方法见如下代码片段（为使逻辑清晰，省略了错误检测，实际开发可不要省啊）*/
      cl_uint numDevices = 0;
  
  // 获取 program 绑定过的 device 数量
  clGetProgramInfo(program,
  				CL_PROGRAM_NUM_DEVICES,
  				sizeof(cl_uint),
  				&numDevices,
  				NULL);
  
  // 获取所有的 device ID
  cl_device_id *devices = new cl_device_id[numDevices];
  clGetProgramInfo(program,
  				CL_PROGRAM_DEVICES,
  				sizeof(cl_device_id) * numDevices,
  				devices,
  				NULL);
  
  // 决定每个 program 二进制的大小
  size_t *programBinarySizes = new size_t[numDevices];
  clGetProgramInfo(program,
  				CL_PROGRAM_BINARY_SIZES,
  				sizeof(size_t) * numDevices,
  				programBinarySizes,
  				NULL);
  
  unsigned char **programBinaries = new unsigned char *[numDevices];
  for(cl_uint i = 0; i < numDevices; ++i)
  	programBinaries[i] = new unsigned char[programBinarySizes[i]];
  
  // 获取所有的 program 二进制
  clGetProgramInfo(program,
  				CL_PROGRAM_BINARIES,
  				sizeof(unsigned char *) * numDevices,
  				programBinaries,
  				NULL);
  
  // 存储 device 所需要的二进制
  for(cl_uint i = 0; i < numDevices; ++i){
  	// 只存储 device 需要的二进制，多个 device 需要存储多个二进制
  	if(devices[i] == device){
  		FILE *fp = fopen("kernel_binary_ocl.bin", "wb"); 
  		fwrite(programBinaries[i], 1， programBinarySizes[i], fp);
  		fclose(fp);
  		break;
  	}
  }
  
  // 清理
  delete[] devices;
  delete [] programBinarySizes;
  for(cl_uint i = 0; i < numDevices; ++i)
  	delete [] programBinaries[i];
  delete[] programBinaries;
  
  ```

  - 读取binary

  ```c++
  /*
  经过上面一系列的操作，我们的磁盘上应该存在一个二进制版的 opencl 程序了。里面的内容可能是可读的，也可能是不可读的。这个视不同厂商实现而不同。
  
  相对于存储，读取看起来就清爽的多，无非是打开二进制文件，然后调用 clCreateProgramWithBinary函数。示例如下：
  */
  FILE *fp= fopen("kernel_binary_ocl.bin", "rb");
  
  // 获取二进制的大小
  size_t binarySize;
  fseek(fp, 0, SEEK_END);
  binarySize = ftell(fp);
  rewind(fp);
  
  // 加载二进制文件
  unsigned char *programBinary = new unsigned char[binarySize];
  fread(programBinary, 1, binarySize, fp);
  fclose(fp);
  
  cl_program program;
  program = clCreateProgramWithBinary(context,
  									1,
  									&device,
  									&binarySize,
  									(const unsigned char**)&programBinary,
  									NULL，
  									NULL）；
  
  delete [] programBinary;
  
  clBuildProgram(program 0, NULL, NULL, NULL, NULL);
  /*
  这里要注意，即使加载是二进制版，我们在之后还是要对其进行 clBuildProgram。原因在于，我们无法保证所谓的二进制一定是可执行码。因为每个厂商的实现不一，有的可能就是最终执行码，而有的却是中间码。所以无论是从源代码还是二进制创建 program，之后都需要 clBuildProgram。
  
  这样兜了一圈，发现要使用二进制版还是用了一遍源代码方式，感觉代码复杂好多，有点多余。其实换个角度来看，我们完全可以写成两个程序，一个专门用来读取源代码并编译生成二进制，另一个才是读取二进制运行软件。前者开发人员使用，后者才是给用户使用的。只有这样才能体现二进制版的优势。
  */
  ```

  

参考：

- https://www.photoneray.com/opencl_04/