# solidity-notes
Solidity 速查表

## 文件结构

### 版本注释

版本注释(Version Pragma)：标注 .sol 文件需要按照指定版本来进行编译，描述指定版本的规则和 npm 的一样。

```
pragma solidity ^0.4.19;
```

### 引入文件

引入文件(Importing other Source Files)：Solidity 引入文件的方式向后兼容 ES6 的 import 规则，在此基础上增加了一些语法。引入外部文件后可以使用外部文件的变量。

```
// ES6 引入外部文件。
import "filename";

// ES6 重命名外部文件的变量
import * as symbolName from "filename";

// Solidity 新增的重命名变量语法
import "filename" as symbolName;
```

### 注释

单行注释

```
// 这个是单行注释
```


多行注释

```
/*
这个是
多行注释
*/、
```


## 合同结构

在 Solidity 中的合同(Contracts)类似于面向对象语言中的类(Class)。每个合同中可以声明：状态变量(State Variables), 函数(Functions), 函数修饰符(Function Modifiers), 事件(Events), 结构类型(Struct Types) and 枚举类型(Enum Types).

### 状态变量

状态变量(State variable)：是永久存在链上的值。值有多种类型：包括值类型(Value Types)、引用类型(Reference Types)和映射(Mappings)等。

```
contract SimpleStorage {
    uint storedData; // 状态变量
}
```

### 函数

函数(Functions)：函数只能在类中进行声明，可以类比为 JavaScript 类的方法。

```
contract SimpleAuction {
    function bid() public payable { // Function
        // ...
    }
}
```

### 函数修饰符

函数修饰符(Function Modifiers)：一般用于执行该函数之前，判断函数是否符合执行条件。

```
contract Purchase {
    address public seller;

    modifier onlySeller() { // Modifier
        require(msg.sender == seller);
        _;
    }

    function abort() public onlySeller { // Modifier usage
        // ...
    }
}
```

### 事件

事件(Events)：事件可以“调用” JavaScript 中的回调函数。 

```
// Solidity
contract ClientReceipt {
    event Deposit(
        address indexed _from,
        bytes32 indexed _id,
        uint _value
    );

    function deposit(bytes32 _id) public payable {
        // 在 dapp 中 “触发” JavaScript 的回调函数
        emit Deposit(msg.sender, _id, msg.value);
    }
}
```

```
// JavaScript
var abi = /* abi as generated by the compiler */;
var ClientReceipt = web3.eth.contract(abi);
var clientReceipt = ClientReceipt.at("0x1234...ab67" /* address */);

// 在与 dapp 交互的接口中传入回调函数
var event = clientReceipt.Deposit(function(error, result) {
    if (!error)
        console.log(result);
});
```


### 结构类型

结构(Structs)：是一种自定义类型，可以将多个变量聚集起来。

```
contract Ballot {
    struct Voter { // Struct
        uint weight;
        bool voted;
        address delegate;
        uint vote;
    }
}
```

### 枚举类型

Enums(Structs)：是一种自定义类型，用于定义有限个值的集合。

```
contract Purchase {
    enum State { Created, Locked, Inactive } // Enum
}
```

## 类型

Solidity 提供几种基础数据类型(elementary types)。复合数据类型(complex types)由基础数据类型组成。

### 值类型

值类型(Value Types)：由类型的值表示数据类型。值类型的始终按值传递(pass by value)，也就是说它们在用作函数参数或赋值时总是被复制。

#### 布尔

布尔(bool)：`true` 、 `false`。

#### 整数

整数(Integers)：包括有符号(signed integers)和无符号整数(unsigned integers)。整数有从 `8` 到 `256` 的不同位数，`int` 指的的是 `int256`，`uint` 指的是 `uint256`。

```
contract SimpleStorage {
    uint storedData; // 声明 unit 类型的状态变量
}
```

#### 地址

地址(Address)：一个 20 字节的值，用于表示以太坊地址。

```
address x = 0xdCad3a6d3569DF655070DEd06cb7A1b2Ccd1D3AF
```

地址类型也有成员，如下：

- `<address>.balance`  (`uint256`): 地址余额，单位 Wei。
- `<address>.transfer(uint256 amount)`: 发送 amount 数量的以太坊(单位 Wei)给 address

### 引用类型 

引用类型(Reference Types)：由类型的引用表示的数据类型，比如数组、结构体。在 Solidity 中，引用类型在用作函数参数或赋值时，可能是传递引用，也有可能会发生复制，这点其他语言(JavaScript)会有些不一样。是传递引用，还是发生复制，取决于数据位置。

#### 数据位置

数据存储位置(Data location)：包括 `memory` 、 `storage` 和 `calldata`。复杂数据类型在 `memory` 和 `storage` 之间赋值时，如 `memory` => `storage`，赋值是副本——发生复制；在 `memory` 和 `memory` 之间赋值，或者在 `storage` 和 `storage` 之间赋值，赋值的是引用。尽可能地赋值引用，避免发生复制，可以减少 Gas 的消耗。

强制存储位置：不可改变存储位置
- 外部函数(external functions)的参数(不包含返回值)：`calldata`
- 状态变量(state variables)：`storage`

默认存储位置：可通过 `storage` 和  `memory` 声明改变存储位置
- 函数参数(包括返回值，除了 `calldata`): `memory`
- 所有其他本地变量：`storage`

#### 数组

数组(Arrays)：相同类型的元素的集合所组成的数据结构。数组可以是固定的长度或者动态的长度，也可以是 `storage` 或者 `memory` 的数据位置。

```
// 固定长度数组：一个固定长度为 k 的数组，其成员的类型为 T，记作 T[k]。写法如下：
uint[k]
uint[][k]

// 动态长度数组：记作 T[]
uint[]
byte[]

// 特殊数组
bytes
string

// 分配内存
uint[] memory a = new uint[](7);
bytes memory b = new bytes(2);

// 数组字面量
[1, 2, 3]

// 类型转换。 [1, 2, 3] 的类型是 uint8[3] memory
uint[3] memory a = [uint(1), 2, 3]

// 固定长度 memory 的数组，不能转换为动态长度 memory 的数组。(未来可能会取消该限制)
// 以下示例会报类型错误。
uint[] memory x = [uint(1), 3, 4];
```

数组成员
- length: 数组元素的数量。
- push: 动态长度的 `storage` 数组和 `bytes`(不包括 string)拥有 push 方法，可以在数组的末尾增加一个元素。



#### 结构体


### 映射 













