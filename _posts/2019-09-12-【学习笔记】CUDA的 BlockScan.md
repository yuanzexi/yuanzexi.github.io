# 【学习笔记】Cub 的 BlockScan 函数

> 版权声明：欢迎各位转载，但是未经作者本人同意，转载文章之后必须在文章页面明显位置给出作者和原文连接，否则保留追究法律责任的权利。 {{site.url}}{{page.url}}

## 什么是 BlockScan
BlockScan 是 Cub 库提供了一种 collective 算法，用于跨 ThreadBlock 进行数据扫描访问或者简单的数值计算操作。

- Prefix Scan: 给定一组输入数据和一个二元 Reduction 操作，得到一组输出数据，其中第 i 个元素是所有在 i 之前原始输入数据元素的 Reduction 的结果。
- Prefix Sum: 包含 Prefix Scan 和 加法操作。
- `inclusive`: 第 i 个输出结果直接写入到第 i 个输入数据的位置。
- `exclusive`: 第 i 个输出结果不直接写入到第 i 个输入数据的位置。
- 对于多维 GridSize，线程按行序排序。
- BlockScan 的三种算法：
    - `cub::BLOCK_SCAN_RAKING`：一个高吞吐量的算法
    - `cub::BLOCK_SCAN_RAKING_MEMOIZE`：类似于`cub::BLOCK_SCAN_RAKING`，有更高的吞吐量，但需要牺牲额外的寄存器容量来做中转存储。
    - `cub::BLOCK_SCAN_WARP_SCANS`：一个低延迟的算法。

## 为什么用 BlockScan
当需要对 Block 内 Threads 管理的资源进行前置 Reduction 操作时，可以考虑采用。


## 怎么用 BlockScan

- 需求：计算 256 个整数项的前置累加和

```c++
#include <iostream>
#include "cuda_runtime.h"
#include <vector>
#include <cub/cub.cuh>

#define CUDA_DEBUG(x)	\
	do {				\
		if (x != cudaError::cudaSuccess) return false; \
	} while (0)

template <typename VectorType>
void PrintVector(VectorType& vector)
{
	for (auto& item : vector)
	{
		std::cout << item << ", ";
	}
	std::cout << std::endl;
}

void GetCudaProp()
{
	cudaDeviceProp cdp;
	cudaGetDeviceProperties(&cdp, 0);
	for (int i =0 ;  i < 3 ; ++i)
	{
		
		std::cout << "max Grid size: " << cdp.maxGridSize[i] << std::endl;
	}
		std::cout << "max thread size :" << cdp.maxThreadsPerBlock << std::endl;
}


template <int SCAN_DIM>
__global__ void TestBlockScan(int* d_inputs, int* aggregate)
{
	typedef cub::BlockScan<int, SCAN_DIM> BlockScan;

	__shared__ typename BlockScan::TempStorage temp_storage;

	int thread_idx = blockIdx.x * blockDim.x + threadIdx.x;

	int thread_data = d_inputs[thread_idx];

	int block_aggregate;

	__syncthreads();

	BlockScan(temp_storage).ExclusiveSum(thread_data, thread_data, block_aggregate);

	__syncthreads();

	d_inputs[thread_idx] = thread_data;

	if (threadIdx.x == 0)
	{
		atomicAdd(aggregate, block_aggregate);
	}
}

bool test_block_scan()
{
	int input_size = 256;

	std::vector<int> inputs(input_size);
	for (int i = 0 ; i < input_size; ++i)
	{
		if (i % 2 == 0)
		{
			inputs[i] = 1;
		}
	}
	int aggregate = 0;

	std::cout << "\n ----------- Before BlockScan: ------------- \n";
	PrintVector(inputs);
	std::cout << "aggregate = " << aggregate << std::endl;

	int* d_inputs;
	int* d_aggregate;

	CUDA_DEBUG(cudaMalloc(&d_inputs, sizeof(int) * input_size));
	CUDA_DEBUG(cudaMalloc(&d_aggregate, sizeof(int)));

	CUDA_DEBUG(cudaMemcpy(d_inputs, inputs.data(), sizeof(int) * inputs.size(), cudaMemcpyHostToDevice));
	CUDA_DEBUG(cudaMemcpy(d_aggregate, &aggregate, sizeof(int), cudaMemcpyHostToDevice));

	const int block_dim = 128;
	const int grid_dim = input_size / 128;
	TestBlockScan<block_dim> <<<grid_dim, block_dim>>> (d_inputs, d_aggregate);

	CUDA_DEBUG(cudaMemcpy(inputs.data(), d_inputs, sizeof(int) * inputs.size(), cudaMemcpyDeviceToHost));
	CUDA_DEBUG(cudaMemcpy(&aggregate, d_aggregate, sizeof(int), cudaMemcpyDeviceToHost));

	std::cout << "\n ----------- After BlockScan: ------------- \n";
	PrintVector(inputs);
	std::cout << "aggregate = " << aggregate << std::endl;

	return true;
}

int main( void ) {
	test_block_scan();

	system("pause");
	
	return 0;
}

```

- 输出结果：
```

 ----------- Before BlockScan: -------------
1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 
1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0,
aggregate = 0

 ----------- After BlockScan: -------------
0, 1, 1, 2, 2, 3, 3, 4, 4, 5, 5, 6, 6, 7, 7, 8, 8, 9, 9, 10, 10, 11, 11, 12, 12, 13, 13, 14, 14, 15, 15, 16, 16, 17, 17, 18, 18, 19, 19, 20, 20, 21, 21, 22, 22, 23, 23, 24, 24, 25, 25, 26, 26, 27, 27, 28, 28, 29, 29, 30, 30, 31, 31, 32, 32, 33, 33, 34, 34, 35, 35, 36, 36, 37, 37, 38, 38, 39, 39, 40, 40, 41, 41, 42, 42, 43, 43, 44, 44, 45, 45, 46, 46, 47, 47, 48, 48, 49, 49, 50, 50, 51, 51, 52, 52, 53, 53, 54, 54, 55, 55, 56, 56, 57, 57, 58, 58, 59, 59, 60, 60, 61, 61, 62, 62, 63, 63, 64, 
0, 1, 1, 2, 2, 3, 3, 4, 4, 5, 5, 6, 6, 7, 7, 8, 8, 9, 9, 10, 10, 11, 11, 12, 12, 13, 13, 14, 14, 15, 15, 16, 16, 17, 17, 18, 18, 19, 19, 20, 20, 21, 21, 22, 22, 23, 23, 24, 24, 25, 25, 26, 26, 27, 27, 28, 28, 29, 29, 30, 30, 31, 31, 32, 32, 33, 33, 34, 34, 35, 35, 36, 36, 37, 37, 38, 38, 39, 39, 40, 40, 41, 41, 42, 42, 43, 43, 44, 44, 45, 45, 46, 46, 47, 47, 48, 48, 49, 49, 50, 50, 51, 51, 52, 52, 53, 53, 54, 54, 55, 55, 56, 56, 57, 57, 58, 58, 59, 59, 60, 60, 61, 61, 62, 62, 63, 63, 64,
aggregate = 128

```


## 注意事项
- BlockScan 是针对 Block 内的指定 Threads 的局部同名变量进行 Reduce 的操作，需注意 ScanDim 和 BlockDim 的大小关系。
- 由于是前置和，即 Output[i] = Output[i - 1] + Array[i - 1]，所以需注意 BlockScan 是从 0 开始，且 Output[0] = 0。
