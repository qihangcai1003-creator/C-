

## 1. 引入头文件

Eigen 是 Header-only 的库，无需编译库文件。

```cpp
#include <iostream>
// 核心模块 (矩阵、向量、代数运算)
#include <Eigen/Dense> 
// 几何模块 (四元数、旋转矩阵、欧拉角) - 只有涉及旋转时才需要
#include <Eigen/Geometry> 

using namespace Eigen;
using namespace std;
````

## 2\. 矩阵与向量定义 (Matrix & Vector)

在机器人学中，**固定大小 (Fixed Size)** 矩阵通常比动态大小更快。

| 类型 | 语法结构 | 示例代码 | 说明 |
| :--- | :--- | :--- | :--- |
| **动态矩阵** | `MatrixXd 变量名(行, 列);` | `MatrixXd A(10, 10);` | 大小在运行时确定，分配在堆上 (Heap)。 |
| **固定矩阵** | `Matrix<类型, 行, 列> 变量名;` | `Matrix3d R;` <br> `Matrix4d T;` | 3x3 或 4x4 矩阵，分配在栈上 (Stack)，**极快**。 |
| **向量** | `VectorXd` / `Vector3d` | `Vector3d v;` | `Vector3d` 本质是 `Matrix<double, 3, 1>`。 |

### 初始化与赋值

```cpp
// 1. 逗号初始化 (最常用)
Matrix2d A;
A << 1, 2,
     3, 4;

// 2. 特殊矩阵
Matrix3d I = Matrix3d::Identity(); // 单位矩阵
MatrixXd Z = MatrixXd::Zero(5, 5); // 全零矩阵
Vector3d v = Vector3d::Ones();     // 全一向量

// 3. 访问元素
double val = A(0, 1); // 第0行，第1列
```

## 3\. 基本代数运算 (Arithmetic)

Eigen 重载了 C++ 运算符，使代码接近数学公式。

```cpp
// 假设 A, B 是矩阵，x, u 是向量，s 是标量
MatrixXd A(2,2), B(2,2);
VectorXd x(2), u(2);
double s = 2.0;

// 矩阵乘法 & 加法 (State Space Equation)
VectorXd x_new = A * x + B * u; 

// 转置
MatrixXd At = A.transpose();

// 求逆 (注意：只有方阵可逆)
MatrixXd A_inv = A.inverse();
```

## 4\. 块操作 (Block Operations)

用于提取或修改矩阵的子区域，常用于构建大系统矩阵或提取旋转/平移部分。

**语法结构：**
`matrix.block(起始行, 起始列, 高度(行数), 宽度(列数))`

**代码示例：**

```cpp
Matrix4d T = Matrix4d::Identity();

// --- 读取块 ---
// 提取左上角 3x3 (旋转矩阵)
Matrix3d R = T.block(0, 0, 3, 3); 
// 提取右上角 3x1 (平移向量)
Vector3d p = T.block(0, 3, 3, 1);

// --- 写入块 ---
Matrix4d S = Matrix4d::Zero();
Matrix2d M_A = Matrix2d::Identity();
// 将 M_A 写入 S 的右下角
S.block(2, 2, 2, 2) = M_A;
```

## 5\. Map 类 (Zero-Copy / 零拷贝)

**核心功能**：将原始 C/C++ 数组 (`double*`) 伪装成 Eigen 对象进行操作，不产生内存拷贝。常用于驱动开发和 Python 接口。

**语法结构：**

  * 固定大小：`Map<Matrix类型>(原始指针)`
  * 动态大小：`Map<Matrix类型>(原始指针, 行数, 列数)`

**代码示例：**

```cpp
double raw_data[6] = {0.1, 0.2, 0.3, 0.4, 0.5, 0.6};

// 1. 映射为 Eigen 向量 (6x1)
// 修改 v_map 就会直接修改 raw_data！
Map<VectorXd> v_map(raw_data, 6); 

// 2. 像操作普通 Eigen 向量一样操作它
v_map.setZero(); // raw_data 瞬间全变 0
v_map(0) = 10.0; // raw_data[0] 变为 10.0
```

## 6\. 几何与四元数 (Geometry & Quaternion)

机器人姿态的核心。注意：Eigen 中四元数存储顺序为 `(w, x, y, z)`，但在构造函数中参数顺序通常是 `w, x, y, z` (看文档) 或者由向量构造。

**语法结构：**

```cpp
// 1. 定义旋转 (轴-角) -> 最直观
AngleAxisd rotation_vector(角度(弧度), 旋转轴(Vector3d));

// 2. 转换为四元数
Quaterniond q(rotation_vector);

// 3. 旋转一个向量
// 公式：v_new = q * v
Vector3d v_rotated = q * v_origin;
```

**代码示例：**

```cpp
// 定义绕 Z 轴旋转 90 度
AngleAxisd t_rot(M_PI / 2, Vector3d::UnitZ());

// 转换为四元数
Quaterniond q(t_rot);

// 原始向量 [1, 0, 0]
Vector3d v(1, 0, 0);

// 旋转后应为 [0, 1, 0]
Vector3d v_new = q * v; 

cout << "Rotated: " << v_new.transpose() << endl;
```


