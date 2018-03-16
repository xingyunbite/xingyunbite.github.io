---
title: EOS-合约理解
comments: false
date: 2018-03-15 18:59:16
categories: 智能合约
tags: 
- ZhouFyk
- 以太坊 
- EOS 
- 智能合约
img:
---

[合约链接](https://etherscan.io/address/0x86fa049857e0209aa7d9e616f7eb3b3b78ecfdb0#code)

## 结构组成

使用了 [dapphub 的库](https://github.com/dapphub)。

```
A:DSAuthority-----     DSAuthEvents
                  |       |
A:ERC20	 DSMath	   ---> DSAuth      DSNote
   |	   |              |           |
    -------                -----------
        |                     |
    DSTokenBase             DSStop
        |                     |
         ---------------------
                   |
                 DSToken

A:abstract contract
下面的合约继承上面的合约。DSAuthority 在 DSAuth 中被使用。
```

## 合约简介

### 1. [DSNote](https://dapp.tools/dappsys/ds-note.html)

记录合约。定义了事件 `LogNote`，声明为 `anonymous`。定义了修饰器 `note`，内部会触发事件 `LogNote`。

### 2. [DSAuth](https://dapp.tools/dappsys/ds-auth.html)

2.1 DSAuthority

抽象函数，只有一个 `canCall` 函数。

2.2 DSAuthEvents

认证事件合约。只有两个事件 `LogSerAuthority / LogSetOwner`。当设置 `Authority / Owner` 时触发。

2.3 DSAuth

继承了 `DSAuthEvents`。设置 `Authority / Owner`，以及对来源的检测。

### 3. [DSStop](https://dapp.tools/dappsys/ds-stop.html)

继承 `DSAuth`，`DSNote`。设置 `stopped`，停止标识符。

### 4. [DSMath](https://dapp.tools/dappsys/ds-math.html)

安全的常规数学运算。包含了 `uint256 / uint128 / int256 / WAD / RAY` 等相关的运算。

### 5. [ERC20](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-20.md)

以太坊上发行代币的接口。实现此接口，可以方便对代币进行操作。

### 6. [DSToken](https://dapp.tools/dappsys/ds-token.html)

6.1 DSTokenBase

继承 `ERC20 / DSMath` 合约。实现了抽象函数中的方法，完成了代币的基础功能。

6.2 DSToken

具体代币的合约，实际的交互合约。

* 以上与所链接的文档有些许差别。

## 合约代码阅读

### 1. DSNote
```
// 记录的合约
contract DSNote {
    /*
    记录 事件 声明了 anonymous，使用了 indexed
    */ 
    event LogNote(
        bytes4   indexed  sig,
        address  indexed  guy,
        bytes32  indexed  foo,
        bytes32  indexed  bar,
	uint	 	  wad,
        bytes             fax
    ) anonymous;

    // 修饰器
    modifier note {
        bytes32 foo;
        bytes32 bar;

        // 内联汇编
        assembly {
            // := 赋值
            // calldataload(p) : 从 p 位置开始调用数据 (32 bytes)
            foo := calldataload(4)
            bar := calldataload(36)
        }

        // 触发 记录事件
        LogNote(msg.sig, msg.sender, foo, bar, msg.value, msg.data);

        _;
    }
}
```

根据[`event` 的文档描述](https://solidity.readthedocs.io/en/latest/contracts.html#events)：
> Up to three parameters can receive the attribute `indexed` which will cause the respective arguments to be searched for: It is possible to filter for specific values of indexed arguments in the user interface.

最多只能有三个参数可以接受 `indexed` 属性，不知道为什么此处有 4 个。`indexed` 指定的参数可以在用户界面中被搜索。

`anonymous` 的声明，[`event` 的文档描述](https://solidity.readthedocs.io/en/latest/contracts.html#events)：
> The hash of the signature of the event is one of the topics except if you declared the event with `anonymous` specifier. This means that it is not possible to filter for specific anonymous events by name.
>
> 除非你使用 `anonymous` 声明事件，否则事件签名的哈希值将成为 topic 之一。意思是无法通过名称筛选指定的 `anonymous` 事件。

[`assembly` 的文档描述](https://solidity.readthedocs.io/en/latest/assembly.html#inline-assembly)：

> 函数式赋值, 如 x := add(y, 3)

[`opcodes` 的文档描述](https://solidity.readthedocs.io/en/latest/assembly.html#opcodes)：

> calldataload(p) : 从 p 位置开始调用数据 (32 bytes)

`foo, bar` 两个参数，一直是当前调用函数相关的值。

### 2. DSAuthority
```
contract DSAuthority {
    // 函数未实现
    function canCall(
        address src, address dst, bytes4 sig
    ) constant returns (bool);
}
```

这是一个抽象合约，`canCall` 函数未实现。`constant` 表示函数不会对合约状态进行修改。

### 3. DSAuthEvents

```
contract DSAuthEvents {
    // 记录验证 事件
    event LogSetAuthority (address indexed authority);
    // 记录主人 事件
    event LogSetOwner     (address indexed owner);
}
```

只包含了两个事件的声明。当用户设置 `authority / owner` 时会触发。

### 4. DSAuth
```
// 认证合约
// `is` 表示继承了 `DSAuthEvents`
contract DSAuth is DSAuthEvents {
    DSAuthority  public  authority;
    address      public  owner;
  
    // 修饰器
    modifier auth {
        // isAuthorized() 为 true
        assert(isAuthorized(msg.sender, msg.sig));
        _;
    }

    // 修饰器 未被使用到
    modifier authorized(bytes4 sig) {
    	// isAuthorized() 为 true
        assert(isAuthorized(msg.sender, sig));
        _;
    }
  
    // 构造函数，设置 owner
    function DSAuth() {
        owner = msg.sender;
        LogSetOwner(msg.sender);
    }

    // 验证函数
    function assert(bool x) internal {
        if (!x) throw;
    }

    // 通过 auth 验证，设置 owner，触发事件
    function setOwner(address owner_) auth
    {
        owner = owner_;
        LogSetOwner(owner);
    }

    // 通过 auth 验证，设置 authority，触发事件
    function setAuthority(DSAuthority authority_) auth
    {
        authority = authority_;
        LogSetAuthority(authority);
    }

    // 认证函数 根据 src 和 authority 的情况来返回操作结果
    function isAuthorized(address src, bytes4 sig) internal returns (bool) {
        if (src == address(this)) { 
            return true;
        } else if (src == owner) { 
            return true;
        } else if (authority == DSAuthority(0)) { 
            return false;
        } else {
            return authority.canCall(src, this, sig);
        }
    }
}
```

合约继承了认证事件。合约主要用来对认证部分进行处理。在设置 owner 和 authority 之前，触发 auth 修饰器，通过函数 `isAuthorized()` 来对消息发送者 `msg` 进行验证。消息发送者 `msg` 包含：

4.1 `msg.data(bytes)`：完整的调用数据
4.2 `msg.gas(uint)`：剩余燃气 - 在版本 0.4.21 中废弃并由 `gasleft()` 取代
4.3 `msg.sender(address)`：当前消息的发送者
4.4 `msg.sig(bytes4)`：调用数据的前四个字节（即函数的标识符）
4.5 `msg.value(uint)`：伴随消息发送的 wei 的数值

函数 `isAuthorized()` 是 `internal` 内部函数，只能在内部或者派生的合约中被调用，无需使用 `this`。首先将 `src` 与 `this` 的地址进行比较，如果为 `true` 则返回 `true`。[`this` 的定义](https://solidity.readthedocs.io/en/latest/units-and-global-variables.html#contract-related)是：

> `this` (current contract’s type): the current contract, explicitly convertible to Address
>
> 当前合约的类型：当前的合约，显式转换为地址。

然后比较 `src` 与 `owner`，如果为 `true` 返回 `true`。构造函数中 `owner` 就是部署合约的用户地址，因为构造函数只设置了 `owner`，所以默认只用操作 `owner`，到此就截止了。

然后比较 `authority` 与 `DSAuthority(0)`。`DSAuthority(0)` 表示将 0 强制转换成 `DSAuthority` 类型，此时与 `authority` 进行比较，如果相等，则说明 `authority` 未被赋值。因为任何变量如果没有被赋值，那么它的初始值就是 0。所以相等，说明未被赋值过，返回 false。而如果比较结果为 false，则说明使用了 `authority` 变量，那么可以进行下一步了，即调用 `authority.canCall(src, this, sig)` 并返回其结果。此处 `canCall` 函数并没有被实现，但是提供了一个接口给用户，如果用户想要使用，只需要继承 `DSAuthority`，然后实现该函数，然后使用该继承的合约即可。

### 5. DSStop
```
// 停止合约
contract DSStop is DSAuth, DSNote {
    // 停止标识符
    bool public stopped;

    // 修饰器 检测停止标识符
    modifier stoppable {
        assert (!stopped);
        _;
    }

    // 停止函数
    function stop() auth note {
        stopped = true;
    }

    // 启动函数
    function start() auth note {
        stopped = false;
    }
}
```

继承了 `DSAuth / DSNote` 合约。定义了 `stoppable` 检测众筹是否停止。定义了两个函数，分别用来设置停止标识符，表示启动或结束，设置的时候通过 `auth` 来进行权限验证，`note` 来触发变更。

### 6. DSMath
```
// 安全的数学运算
contract DSMath {
    
    /*
    标准 uint256 函数 256 位无符号数 即 非负整数 0 ~ 2^256 - 1

    constant 表示不会改变状态变量
    internal 表示只会被本合约和继承了的合约所使用
     */

    // 加法
    function add(uint256 x, uint256 y) constant internal returns (uint256 z) {
        assert((z = x + y) >= x);
    }

    // 减法
    function sub(uint256 x, uint256 y) constant internal returns (uint256 z) {
        assert((z = x - y) <= x);
    }

    // 乘法
    function mul(uint256 x, uint256 y) constant internal returns (uint256 z) {
        assert((z = x * y) >= x);
    }

    // 除法 当除以 0 时，solidity 会自动抛出异常
    function div(uint256 x, uint256 y) constant internal returns (uint256 z) {
        z = x / y;
    }

    // 返回较小值
    function min(uint256 x, uint256 y) constant internal returns (uint256 z) {
        return x <= y ? x : y;
    }

    // 返回较大值
    function max(uint256 x, uint256 y) constant internal returns (uint256 z) {
        return x >= y ? x : y;
    }

    /*
    uint128 functions (h is for half) 由 256 位变为 128 位 其余同上
     */

    function hadd(uint128 x, uint128 y) constant internal returns (uint128 z) {
        assert((z = x + y) >= x);
    }

    function hsub(uint128 x, uint128 y) constant internal returns (uint128 z) {
        assert((z = x - y) <= x);
    }

    function hmul(uint128 x, uint128 y) constant internal returns (uint128 z) {
        assert((z = x * y) >= x);
    }

    function hdiv(uint128 x, uint128 y) constant internal returns (uint128 z) {
        z = x / y;
    }

    function hmin(uint128 x, uint128 y) constant internal returns (uint128 z) {
        return x <= y ? x : y;
    }
    function hmax(uint128 x, uint128 y) constant internal returns (uint128 z) {
        return x >= y ? x : y;
    }


    /*
    int256 functions 有符号 256 位函数
     */

    function imin(int256 x, int256 y) constant internal returns (int256 z) {
        return x <= y ? x : y;
    }
    function imax(int256 x, int256 y) constant internal returns (int256 z) {
        return x >= y ? x : y;
    }

    /*
    WAD math 提供 18 位精度，同上 乘法和除法加入 WAD
     */

    uint128 constant WAD = 10 ** 18;

    function wadd(uint128 x, uint128 y) constant internal returns (uint128) {
        return hadd(x, y);
    }

    function wsub(uint128 x, uint128 y) constant internal returns (uint128) {
        return hsub(x, y);
    }

    function wmul(uint128 x, uint128 y) constant internal returns (uint128 z) {
        z = cast((uint256(x) * y + WAD / 2) / WAD);
    }

    function wdiv(uint128 x, uint128 y) constant internal returns (uint128 z) {
        z = cast((uint256(x) * WAD + y / 2) / y);
    }

    function wmin(uint128 x, uint128 y) constant internal returns (uint128) {
        return hmin(x, y);
    }
    function wmax(uint128 x, uint128 y) constant internal returns (uint128) {
        return hmax(x, y);
    }

    /*
    RAY math 同上，但是精度不同 为 10 ** 27
     */

    uint128 constant RAY = 10 ** 27;

    function radd(uint128 x, uint128 y) constant internal returns (uint128) {
        return hadd(x, y);
    }

    function rsub(uint128 x, uint128 y) constant internal returns (uint128) {
        return hsub(x, y);
    }

    function rmul(uint128 x, uint128 y) constant internal returns (uint128 z) {
        z = cast((uint256(x) * y + RAY / 2) / RAY);
    }

    function rdiv(uint128 x, uint128 y) constant internal returns (uint128 z) {
        z = cast((uint256(x) * RAY + y / 2) / y);
    }

    // 指数运算
    function rpow(uint128 x, uint64 n) constant internal returns (uint128 z) {
        // This famous algorithm is called "exponentiation by squaring"
        // and calculates x^n with x as fixed-point and n as regular unsigned.
        //
        // It's O(log n), instead of O(n) for naive repeated multiplication.
        //
        // These facts are why it works:
        //
        //  If n is even, then x^n = (x^2)^(n/2).
        //  If n is odd,  then x^n = x * x^(n-1),
        //   and applying the equation for even x gives
        //    x^n = x * (x^2)^((n-1) / 2).
        //
        //  Also, EVM division is flooring and
        //    floor[(n-1) / 2] = floor[n / 2].

        z = n % 2 != 0 ? x : RAY;

        for (n /= 2; n != 0; n /= 2) {
            x = rmul(x, x);

            if (n % 2 != 0) {
                z = rmul(z, x);
            }
        }
    }

    function rmin(uint128 x, uint128 y) constant internal returns (uint128) {
        return hmin(x, y);
    }
    function rmax(uint128 x, uint128 y) constant internal returns (uint128) {
        return hmax(x, y);
    }

    // x 的 uint256 类型可以无损被转换成 uint128 类型
    function cast(uint256 x) constant internal returns (uint128 z) {
        assert((z = uint128(x)) == x);
    }
}
```

### 7. ERC20
```
// ERC20 抽象合约 接口
contract ERC20 {
    // 代币总量
    function totalSupply() constant returns (uint supply);
    // 用户资产
    function balanceOf(address who) constant returns (uint value);
    // 第三方可提币额度
    function allowance(address owner, address spender) constant returns (uint _allowance);

    // 转账到指定账户
    function transfer(address to, uint value) returns (bool ok);
    // 从第三方账户转账
    function transferFrom(address from, address to, uint value) returns (bool ok);
    // 设置提币额度
    function approve(address spender, uint value) returns (bool ok);

    // 转账事件
    event Transfer(address indexed from, address indexed to, uint value);
    // 提币事件
    event Approval(address indexed owner, address indexed spender, uint value);
}
```

以太坊规定的一个代币接口。与普遍意义上的接口目的相同，对于某些功能的一些约定，满足接口来实现更方便的协同操作。如果代币都实现了接口，那么钱包就能使用同一套方式来操作这些代币等等。

### 8. DSTokenBase
```
// 代币基础合约
contract DSTokenBase is ERC20, DSMath {
    uint256                                            _supply; // 代币当前总量
    mapping (address => uint256)                       _balances; // 余额数组
    mapping (address => mapping (address => uint256))  _approvals; // 提币额度数组
    
    // 构造函数
    function DSTokenBase(uint256 supply) {
        _balances[msg.sender] = supply; // 设置发行代币者的初始余额
        _supply = supply; // 设置当前代币总额
    }
    
    // 返回当前代币总量
    function totalSupply() constant returns (uint256) {
        return _supply;
    }

    // 返回 src 用户的余额
    function balanceOf(address src) constant returns (uint256) {
        return _balances[src];
    }

    // 返回 guy 可以从 src 提取的币量
    function allowance(address src, address guy) constant returns (uint256) {
        return _approvals[src][guy];
    }
    
    // 从自己的余额中转移 wad 个代币到 dst 余额中
    function transfer(address dst, uint wad) returns (bool) {
        // 自己的余额足够转账数量 否则 throw
        assert(_balances[msg.sender] >= wad);
        
        // 通过 DSMath 来对余额进行增减
        _balances[msg.sender] = sub(_balances[msg.sender], wad);
        _balances[dst] = add(_balances[dst], wad);
        
        // 触发事件
        Transfer(msg.sender, dst, wad);
        
        return true;
    }
    
    // 从 src 转移 wad 个代币到 dst 余额
    function transferFrom(address src, address dst, uint wad) returns (bool) {
    	// 转账金额不能超额
        assert(_balances[src] >= wad);
        assert(_approvals[src][msg.sender] >= wad);
        
        // 通过 DSMath 来处理余额
        _approvals[src][msg.sender] = sub(_approvals[src][msg.sender], wad);
        _balances[src] = sub(_balances[src], wad);
        _balances[dst] = add(_balances[dst], wad);
        
        // 触发事件
        Transfer(src, dst, wad);
        
        return true;
    }
    
    // 设置 guy 可以从自己的余额中提取 wad 个代币
    function approve(address guy, uint256 wad) returns (bool) {
        _approvals[msg.sender][guy] = wad;
        
        Approval(msg.sender, guy, wad);
        
        return true;
    }
}
```

继承 `DSMath` 和抽象合约 `ERC20`。实现 `ERC20` 的代币函数，使用 `DSMath` 来处理余额。

### 9. DSToken
```
// 具体代币合约
// `DSTokenBase(0)` 运行了 `DSTokenBase` 的构造函数，传入的参数值为 0。
contract DSToken is DSTokenBase(0), DSStop {
    bytes32  public  symbol; // 代币符号 如 EOS
    uint256  public  decimals = 18; // 标准的代币精度，可以被定制

    // 构造函数 设置代币符号
    function DSToken(bytes32 symbol_) {
        symbol = symbol_;
    }

    function transfer(address dst, uint wad) stoppable note returns (bool) {
        return super.transfer(dst, wad);
    }

    function transferFrom(
        address src, address dst, uint wad
    ) stoppable note returns (bool) {
        return super.transferFrom(src, dst, wad);
    }

    function approve(address guy, uint wad) stoppable note returns (bool) {
        return super.approve(guy, wad);
    }

    // 推送 本合约 `transfer()` 的别名
    function push(address dst, uint128 wad) returns (bool) {
        return transfer(dst, wad);
    }

    // 拉取 本合约 `transferFrom()` 的特例
    function pull(address src, uint128 wad) returns (bool) {
        return transferFrom(src, msg.sender, wad);
    }

    // 充值 增加代币
    function mint(uint128 wad) auth stoppable note {
        _balances[msg.sender] = add(_balances[msg.sender], wad);
        _supply = add(_supply, wad);
    }

    // 燃烧 销毁代币
    function burn(uint128 wad) auth stoppable note {
        _balances[msg.sender] = sub(_balances[msg.sender], wad);
        _supply = sub(_supply, wad);
    }

    // Optional token name

    bytes32   public  name = "";
    
    function setName(bytes32 name_) auth {
        name = name_;
    }
}
```

`transfer / transferFrom / approve / mint / burn` 等方法使用了 `stoppable / note` 修饰器，在执行这些函数前，需要判断众筹是否已经停止，如果未停止，触发记录事件。

`mint / burn` 还在其他修饰器之前使用了 `auth` 修饰器，表示在使用这两个函数时，首要检查调用这些函数的用户是否合法。

比起 `ERC20` 约定的基本函数，本合约额外增加了 `push / pull / mint / burn`。从具体代码来看，`push` 成了 `transfer` 的一个别名，而 `pull` 调用 `transferFrom` 时将 `dst` 固定为 `msg.sender`。从逻辑上来看，`push / pull` 是针对转账的额外处理，用于当前用户对自身余额的操作。`mint / burn` 则是对应于 `EOS` 的众筹方式。

## 最后

从区块浏览器内查询到的 EOS 合约如上。合约本身并不复杂，Solidity 的基本语法，和 ERC20 代币的逻辑。但是如 `DSToken` 合约中的 `mint / burn` 则与 EOS 的众筹方式相关：[GitHub 仓库](https://github.com/EOSIO/eos-token-distribution) - [区块浏览器合约地址](https://etherscan.io/address/0xd0a6e6c54dbc68db5db3a091b171a77407ff7ccf#code)。
