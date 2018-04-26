---
title: token 合约大数溢出安全问题
comments: false
date: 2018-04-26 15:02:11
categories: 智能合约
tags:
- ZhouFyk
- 以太坊
- 智能合约
img:
---

上周连续爆出一些比较知名的 token 的安全问题，后续有消息称仍然存在同类问题的 token。简单描述一下该类问题的存在形式。

## BEC

```
  function batchTransfer(address[] _receivers, uint256 _value) public whenNotPaused returns (bool) {
    uint cnt = _receivers.length;
    uint256 amount = uint256(cnt) * _value; // 大数溢出 amount 可能从 11111.....1111 体现成 1111 从而在发送者的余额范围内
    require(cnt > 0 && cnt <= 20);
    require(_value > 0 && balances[msg.sender] >= amount);

    balances[msg.sender] = balances[msg.sender].sub(amount);
    for (uint i = 0; i < cnt; i++) {
        balances[_receivers[i]] = balances[_receivers[i]].add(_value); // 各个地址增加 _value 余额
        Transfer(msg.sender, _receivers[i], _value);
    }
    return true;
  }
```

`BEC` 的问题出现在 `batchTransfer` 方法。如注释所述，当 `amount` 超过最大值，那么该值会被截取范围内的数字，当截取后的数字处在当前用户余额范围内，那么 `_value > 0 && balances[msg.sender] >= amount` 成立。后面的代码就可以正常执行，余额减去实际的 `amount` 截取后的较小数字，然后可以给地址列表内的地址增加 `_value` 余额。

## SMT

```
    function transferProxy(address _from, address _to, uint256 _value, uint256 _feeSmt,
        uint8 _v,bytes32 _r, bytes32 _s) public transferAllowed(_from) returns (bool){

        // 将 _feeSmt + _value 的和截取后溢出值小于发起者余额 绕过检测
        // _value = 8fffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
        // _feeSmt = 7000000000000000000000000000000000000000000000000000000000000001
        if(balances[_from] < _feeSmt + _value) revert();

        uint256 nonce = nonces[_from]; // 获得发起人 nonce 
        bytes32 h = keccak256(_from,_to,_value,_feeSmt,nonce);
        if(_from != ecrecover(h,_v,_r,_s)) revert();

        // _value > 0 , _feeSmt > 0
        if(balances[_to] + _value < balances[_to]
            || balances[msg.sender] + _feeSmt < balances[msg.sender]) revert();
        balances[_to] += _value;
        Transfer(_from, _to, _value);

        balances[msg.sender] += _feeSmt;
        Transfer(_from, msg.sender, _feeSmt);

        balances[_from] -= _value + _feeSmt; // 发起人扣币为溢出之后截取的值
        nonces[_from] = nonce + 1; 
        return true;
    }
```

`SMT` 的问题在 `transferProxy` 方法。如注释所述（注释中的数值示例是从异常交易中查看得到的，这里是将两个数字之和刚好溢出而得到 0 的情况），当 `_feeSmt + _value` 的和溢出之后，截取的较小数字处在余额范围内，就可以通过 `balances[_from] < _feeSmt + _value` 的判断条件。后续代码继续执行，就会给 `_to` 用户的余额中增加资产。

## 总结

如果无法保证自身在编码时考虑到一切细节，那么应该使用安全的数学类库来进行相关的操作。而事关用户资产的代码，应该是极其重要的，所以要更加严格。
